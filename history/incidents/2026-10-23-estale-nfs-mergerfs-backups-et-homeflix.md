---
title: "Incident — Échecs ESTALE NFS sur Backups PBS et HomeFlix (Retrait de MergerFS)"
description: "Déconnexions NFS en cascade et échecs de sauvegarde causés par l'invalidation des descripteurs FUSE MergerFS lors des redémarrages du NAS, résolus par le passage en bind-mount ext4 direct"
icon: "hard-drive"
iconType: "duotone"
last_reviewed: "2026-10-23"
---

<Badge color="green">🟢 Résolu & Correctif Structurel Déployé</Badge> *(2026-10-23)*

*Incidents survenus entre le 15/10 et le 23/10/2026, analysés et corrigés définitivement le 23/10/2026.*

---

## Chronologie des Faits

### 1. Incident n°1 — Échec récurrent des sauvegardes de la VM 104 (15/10 → 19/10)

Du 15 au 19 octobre 2026, le job de sauvegarde automatique `vzdump` de la VM 104 (`ims-coolify`, planifié à 02:00 vers le storage PVE `pbs-coolify`) a échoué chaque nuit de façon silencieuse :

```text
ERROR: VM 104 qmp command 'backup' failed - backup register image failed: 
command error: unable to get shared lock - ESTALE: Stale file handle
```

- Le freeze/thaw du guest agent QEMU et l'instanciation du snapshot QMP réussissaient sur le nœud `pve`. L'échec survenait lors de la prise de verrou côté Proxmox Backup Server (LXC 103).
- **Angle mort de notification** : Les alertes étaient envoyées uniquement vers `mail-to-root` (boîte locale non surveillée). L'incident n'a été découvert manuellement que le 19/10 en fin de matinée.
- **Rétablissement à chaud** : Bien que le point de montage `/mnt/pbs-datastore` répondait aux commandes shell (`ls`), le démon `proxmox-backup-proxy` conservait des descripteurs de fichiers périmés. Un redémarrage des services PBS sur le LXC 103 a immédiatement permis la reprise des sauvegardes :
  ```bash
  systemctl restart proxmox-backup-proxy proxmox-backup
  ```

### 2. Incident n°2 — Coupure HomeFlix et échec PBS post-reboot du NAS (23/10)

Le 22 octobre au soir, le conteneur LXC 100 (`ims-nas`) a été redémarré dans le cadre des travaux d'enrôlement réseau. Dès la nuit suivante :

1. **Échec du backup VM 104 à 02:00** :
   ```text
   ERROR: VM 104 qmp command 'backup' failed - backup connect failed: command error: namespace not found
   ```
   Bien que le stockage `pbs-coolify` n'ait aucun namespace configuré, PBS a renvoyé cette erreur trompeuse car son datastore `/mnt/pbs-datastore` était en état `ESTALE` : PBS était dans l'incapacité de lire la racine du datastore.
2. **Coupure du streaming HomeFlix (Jellyfin)** :
   - À 03:00, la tâche nocturne Trickplay de Jellyfin signalait `Media not found` sur l'ensemble de la médiathèque (`/movies` et `/series`).
   - À 11:35, la tâche d'analyse de la médiathèque levait une exception `System.IO.IOException: Stale file handle : '/movies'`.
   - À 13:15, toute tentative de lecture vidéo déclenchait une erreur `FFmpeg exited with code 140`.
3. **Rétablissement d'urgence du 23/10** :
   - Sur le LXC 103 : démontage forcé, remontage et redémarrage des démons PBS :
     ```bash
     umount -l /mnt/pbs-datastore && mount /mnt/pbs-datastore
     systemctl restart proxmox-backup-proxy proxmox-backup
     ```
   - Sur la VM 104 : redémarrage des conteneurs applicatifs branchés sur le NAS (`docker restart jellyfin sonarr radarr prowlarr photoprism immich zipline forgejo`), rétablissant immédiatement le service sans nécessiter de remontage sur l'hôte.

---

## Cause Racine (Root Cause)

Le point de montage `/mnt/storage` sur le NAS (LXC 100) était agrégé via **MergerFS**, un système de fichiers en espace utilisateur (**FUSE**), puis réexporté via le serveur de noyau NFS (`nfsd`).

1. **Rupture des handles FUSE** : Le protocole NFS attribue des descripteurs de fichiers (`filehandles`) qui dépendent de la structure interne du système de fichiers sous-jacent. Lorsqu'un processus FUSE (MergerFS) redémarre ou est réinitialisé, il régénère ses identifiants internes en mémoire.
2. **Perte de correspondance côté client** : Pour les clients NFS (LXC 103 et VM 104), les handles ouverts deviennent instantanément obsolètes (`ESTALE: Stale file handle`).
3. **Inefficacité des options de contournement** : Les directives telles que `inodecalc=path-hash`, `use_ino` ou `fsid=` ne maintiennent pas la persistance des descripteurs au travers d'une réinitialisation de l'espace utilisateur FUSE.
4. **Constat d'architecture** : **Exporter un système FUSE (MergerFS) en NFS constitue un défaut de conception structurel** pour un environnement de production.

---

## Correctif Appliqué — Retrait de MergerFS sur `/mnt/storage`

Plutôt que d'empiler des scripts de surveillance et de reconnexion automatique, la décision a été prise de **supprimer purement et simplement MergerFS** du chemin critique de stockage :

1. **Bind-mount direct ext4** : Le montage FUSE `/mnt/storage` est remplacé dans le `/etc/fstab` du LXC 100 par un bind-mount direct du disque physique HDD (`/mnt/disk1` → `/mnt/storage`), calqué sur la configuration déjà opérationnelle du SSD `/mnt/storage-hot` :
   ```text
   # /etc/fstab sur le LXC 100 (NAS)
   /mnt/disk1        /mnt/storage      none  bind  0 0
   /mnt/ssd-hot-raw  /mnt/storage-hot  none  bind  0 0
   ```
2. **Exports NFS préservés** : La configuration `/etc/exports` et les chemins de montage des clients restent strictement identiques. L'export s'effectue désormais sur du **`ext4` natif**, dont les descripteurs NFS persistent parfaitement lors d'un arrêt ou redémarrage du NAS.
3. **Fin de la virtualisation d'inodes** : Les inodes redeviennent les vrais inodes `ext4`. La politique `inodecalc=path-hash` ([ADR-007](/history/adr/adr-007-calcul-inodes-mergerfs-path-hash)) devient sans objet.

---

## Enseignements & Actions Préventives

1. **Règle d'or de stockage** : Ne jamais exporter en NFS un système de fichiers FUSE.
2. **Surveillance des sauvegardes (Ntfy)** : L'incident n°1 a mis en évidence le danger d'une alerte cantonnée à `mail-to-root`. Le branchement d'une notification push Ntfy sur le topic `ims-alerts` via l'interface Proxmox VE (**Datacenter → Notifications**) est inscrit en priorité haute dans la roadmap.
3. **Interprétation de `namespace not found`** : Cette erreur sur un job vzdump PBS ne signifie pas uniquement un namespace absent dans la configuration, mais peut être le symptôme direct d'un datastore PBS inaccessible suite à une erreur `ESTALE`.
