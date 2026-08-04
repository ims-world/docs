---
title: "IMS-PBS (LXC 103)"
description: "Proxmox Backup Server — sauvegarde des VM/LXC applicatifs"
---

## Rôle

Datastore de sauvegarde déduplicée pour les VM/LXC applicatifs (VM Coolify en premier lieu). Stocke les backups sur le HDD du NAS via NFS.

<Warning>
**Ne sauvegarde PAS la donnée brute du NAS.** Principe anti-circularité : le NAS et PBS eux-mêmes sont sauvegardés séparément (`vzdump` vers le NVMe local du host), jamais dans le datastore qu'ils hébergent/gèrent.
</Warning>

## Fiche technique

| Propriété | Valeur |
|---|---|
| **VMID** | 103 |
| **Type** | LXC Debian 12, **privilégié** |
| **CPU / RAM** | 2 cores / 1024 MB |
| **Réseau** | `vmbr0` (192.168.1.51/24) + `vmbr1` (10.10.10.3/24) + client Tailscale dédié |
| **Accès Tailscale** | `100.64.0.2`, hostname `ims-pve-103-pbs` |

<Warning>
**Écart vs plan initial** : PBS devait être unprivileged. Un LXC unprivileged ne peut pas monter de partage NFS — restriction noyau sur les user namespaces, non contournable. Passage en privilégié nécessaire après une tentative infructueuse.
</Warning>

## Accès distant — client Tailscale dédié (Option B)

Contrairement au NAS (aucun accès distant), PBS a son propre client Tailscale plutôt qu'un subnet router généraliste — accès chirurgical, pas d'exposition du LAN entier.

<Steps>
  <Step title="Device TUN requis dans la config LXC">
    ```
    lxc.cgroup2.devices.allow: c 10:200 rwm
    lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
    ```
    <Warning>À ne pas oublier en cas de recréation du CT — sans ces lignes, Tailscale ne démarre pas.</Warning>
  </Step>
  <Step title="Installation">
    ```bash
    curl -fsSL https://tailscale.com/install.sh | sh
    tailscale up --login-server=https://vpn.ims-world.fr --hostname=ims-pbs --accept-routes
    ```
  </Step>
</Steps>

## Datastore

```mermaid
graph LR
    subgraph PVE_HOST ["🖥️ Proxmox Host (MS-01)"]
        VM104["VM 104 (IMS-Coolify)"]
        VZDUMP["vzdump (Local NVMe)"]
    end

    subgraph PBS_LXC ["💾 IMS-PBS (LXC 103)"]
        PBS_SRV["Proxmox Backup Server Engine"]
        MNT["/mnt/pbs-datastore (NFSv3 Mount)"]
    end

    subgraph NAS_LXC ["📁 IMS-NAS (LXC 100)"]
        NAS_NFS["NFS Export: /mnt/storage/backups\n(10.10.10.1 - vers=3)"]
        HDD["HDD 3To Physique"]
    end

    VM104 -->|1. Backup quotidien 02:00 (vma.zst chunks)| PBS_SRV
    PBS_SRV -->|2. Écriture chunks déduplicatifs| MNT
    MNT <-->|3. Transit NFSv3 (Réseau Isolé vmbr1)| NAS_NFS
    NAS_NFS --> HDD

    PVE_HOST -.->|4. Anti-circularité: vzdump direct du NAS & PBS| VZDUMP

    classDef host fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef pbs fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef nas fill:#1a2b3c,stroke:#0F6E56,color:#fff;
    class VM104,VZDUMP host;
    class PBS_SRV,MNT pbs;
    class NAS_NFS,HDD nas;
```

| Propriété | Valeur |
|---|---|
| **Nom** | `pve-backups` |
| **Backing path** | `/mnt/pbs-datastore` |
| **Source NFS** | `10.10.10.1:/mnt/storage/backups` (le HDD du NAS) |

<Note>
Nommé `pve-backups`, pas `nas-backups` — le premier nom prêtait à confusion (laissait croire à un backup de la donnée brute du NAS, alors que c'est l'inverse : le NAS héberge le stockage physique des backups des *autres* machines).
</Note>

## Jobs configurés

| Job | Schedule | Détail |
|---|---|---|
| Garbage Collection | `sat 03:00` | Libère les chunks non référencés |
| Prune | quotidien | Rétention 7 daily / 4 weekly / 3 monthly |
| Verify | `sun 04:00` | Détecte la corruption silencieuse |
| **Backup VM Coolify** | `02:00` quotidien | Configuré côté Proxmox host (Datacenter → Backup) |

<Check>
Premier backup manuel de la VM Coolify (104) lancé et validé avec succès.
</Check>

<Warning>
**Montage NFS forcé en `vers=3`** (pas la valeur par défaut v4.2) — un bug d'interaction NFSv4.2/MergerFS causait un `Stale file handle` systématique en toute fin de backup (échec du commit du manifest, après un transfert 100% réussi). Voir [Dépannage courant](/procedures/depannage-courant#stale-file-handle-en-fin-de-backup-nfsv4-2-mergerfs) pour le détail complet du diagnostic.
```
# fstab du CT PBS
10.10.10.1:/mnt/storage/backups  /mnt/pbs-datastore  nfs  defaults,nofail,_netdev,vers=3  0 0
```
</Warning>

## Grand troubleshooting NFS — résumé

Le montage du datastore a nécessité de résoudre une cascade de blocages :

1. LXC unprivileged → montage NFS impossible → passage en privilégié
2. Feature `mount=nfs` omise lors de la recréation → `access denied by server` trompeur (en réalité une erreur locale `Permission denied`)
3. Port SSH du Mac Mini bloqué par **Endlessh** (tarpit anti-bot sur le port 22) — le vrai port est **4242**
4. Permissions `acme.json` / options MergerFS (`inodecalc=path-hash`) ajustées côté NAS pour la stabilité NFS long terme

<Card title="Détail complet du troubleshooting" icon="magnifying-glass" href="/procedures/depannage-courant">
  Toutes les commandes de diagnostic et fausses pistes explorées.
</Card>
