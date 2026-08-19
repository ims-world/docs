---
title: "Incident — Stale NFS Filehandles sur Jellyfin (vzdump stop LXC 100)"
description: "Invalidation des descripteurs de fichiers NFS causée par le redémarrage quotidien de MergerFS FUSE et suppression de la sauvegarde automatique LXC 100"
icon: "hard-drive"
iconType: "duotone"
---

<Badge color="green">🟢 Résolu & Choix d'Architecture Acté</Badge> *(2026-08-19)*

---

## Symptôme

Après quelques jours d'uptime sans incident, la lecture de films et séries sur **HomeFlix (Jellyfin)** échouait systématiquement (aussi bien en lecture directe qu'en transcodage matériel QSV, tous codecs confondus). Le problème n'affectait pas la navigation dans la bibliothèque mais uniquement l'ouverture des flux vidéo, et se résolvait temporairement lors du redémarrage du conteneur Jellyfin.

---

## Cause Racine (Root Cause)

Le conteneur **LXC 100 (`ims-nas`)** agrège le pool de stockage froid (`storage`) via **MergerFS**, qui s'exécute en tant que système de fichiers en espace utilisateur (**FUSE**).

1. Un job de sauvegarde Proxmox automatique (`backup-6b92bcb1-ad11`, VMID 100, planifié chaque matin à 05h00 en `--mode stop`) arrêtait et relançait automatiquement le conteneur LXC 100.
2. À chaque redémarrage de `ims-nas`, le démon MergerFS réinitialise son instance FUSE et ses identifiants internes.
3. Pour le serveur NFS (`nfsd`) et le client NFS de la **VM IMS-Coolify (VM 104)**, les descripteurs de fichiers (`filehandles`) ouverts sur le point de montage `/mnt/nas-storage` devenaient **définitivement invalides (`Stale filehandle`)**.
4. Le client NFS du noyau Linux ne peut pas récupérer automatiquement une session FUSE réinitialisée : la connexion est rompue jusqu'à l'exécution d'un remontage forcé et le redémarrage des processus conteneurisés (Jellyfin, HomeFlix, PhotoPrism) qui conservaient des descripteurs de fichiers ouverts.

---

## Pourquoi le Mode Snapshot n'était pas une Option

L'alternative consistant à passer le job de sauvegarde de LXC 100 en `--mode snapshot` (pour éviter le `stop`) a été expérimentée mais a **échoué** :
- Lors de la phase d'instantané, Proxmox tente d'exécuter un gel (*freeze*) du système de fichiers du conteneur.
- Comme le point de montage `mp0` (MergerFS) demeure monté dans l'espace de nommage (*mount namespace*) du conteneur, l'opération de gel **bloque indéfiniment**.
- L'exclusion du contenu (`backup=0`) sur les points de montage `mp0` et `mp1` **n'exclut pas** l'opération de gel du mount namespace. Le mode `stop` reste donc la seule option technique valide pour sauvegarder cette LXC tant que MergerFS y réside.

---

## Découverte Annexe & Correctif d'Urgence

Au cours du diagnostic, un risque d'escalade critique a été identifié : le point de montage `mp1` (SSD Samsung 870 EVO 4 To, hébergeant `storage-hot` pour Immich et Forgejo) ne possédait pas la directive `backup=0`.

<Warning>
**Risque de Saturation du Disque Proxmox** : Sans la directive `backup=0`, tout job `vzdump` sur la LXC 100 tentait de copier l'intégralité du SSD 4 To dans le stockage local de Proxmox (`local-lvm`), risquant de sature l'hyperviseur.
</Warning>

**Fix appliqué immédiatement sur l'hôte MS-01** :
```bash
sudo pct set 100 -mp1 /dev/disk/by-id/ata-Samsung_SSD_870_EVO_4TB_S758NX0W713130Z,mp=/mnt/ssd-hot-raw,backup=0
```
Les deux points de montage `mp0` (MergerFS HDD) et `mp1` (SSD Hot) sont désormais tous les deux strictement exclus du périmètre de sauvegarde (`backup=0`).

---

## Décisions d'Architecture & Correctifs

1. **Suppression de la Sauvegarde Automatique Quotidienne de LXC 100** :
   - La sauvegarde automatique de LXC 100 ne protégeait qu'un système de fichiers racine (*rootfs*) de 8 Go quasiment statique.
   - Le coût (interruption NFS quotidienne et descripteurs corrompus) était totalement disproportionné par rapport au bénéfice. Le job `backup-6b92bcb1-ad11` a été **désactivé et supprimé**.
2. **Reboot complet du Host MS-01 non impacté** :
   - Lors d'un redémarrage complet de l'hyperviseur, l'ordre d'initialisation des guests Proxmox garantit que `ims-nas` (LXC 100, `order=1`) démarre et initialise MergerFS **avant** le démarrage de la VM Coolify (VM 104, `order=3`). Le montage NFS s'établit donc proprement au boot.
3. **Rejet du Watchdog par Polling** :
   - La mise en place d'un script de surveillance automatique (*watchdog*) effectuant des lectures régulières sur le disque a été écartée pour éviter de sortir inutilement les disques durs du mode veille (*spin-down*).

---

## Point d'Attention à Surveiller

Une entrée `NFSD: starting 90-second grace period` observée dans le `dmesg` à 03h00 le 19/08 reste à confirmer (potentiellement liée à l'exécution du job de sauvegarde de la LXC 103 PBS). À surveiller si le symptôme réapparaît hors intervention.
