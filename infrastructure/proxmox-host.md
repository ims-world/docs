---
title: "Host Proxmox (MS-01)"
description: "Hyperviseur principal — Minisforum MS-01, Proxmox VE 9.2.3"
---

## Fiche matériel

| Propriété | Valeur |
|---|---|
| **Modèle** | Minisforum MS-01 |
| **CPU** | Intel i5-12600H (iGPU Iris Xe intégrée, non exploitée actuellement) |
| **RAM** | 31 Gi disponibles |
| **Stockage NVMe** | ~930 Go, LVM-Thin (`local-lvm`, ~793 Go utilisables) |
| **OS** | Proxmox VE 9.2.3 |
| **Accès admin** | `cmolotkoff@pam` (compte nominatif), `root@pam` en break-glass local uniquement |

## Topologie Matérielle & Allocation

```mermaid
graph TD
    subgraph HW ["💻 Hardware MS-01 (Intel i5-12600H, 31 GiB RAM)"]
        CPU["12 Cores / 16 Threads"]
        RAM["31 GiB RAM"]
        NVME["930 Go NVMe (LVM-Thin: local-lvm)"]
        HDD["Disque 3To Apple/Seagate (Passthrough mp0)"]
    end

    subgraph PVE ["🖥️ Proxmox VE 9.2.3"]
        subgraph LXC100 ["IMS-NAS (LXC 100)"]
            NAS_RES["2 Cores | 1 Go RAM | mp0 Passthrough HDD"]
        end
        subgraph LXC103 ["IMS-PBS (LXC 103)"]
            PBS_RES["2 Cores | 1 Go RAM | Datastore NFS"]
        end
        subgraph VM104 ["IMS-Coolify (VM 104)"]
            COOL_RES["6 Cores | 12 Go RAM | 128 Go NVMe"]
        end
    end

    CPU --> PVE
    RAM --> PVE
    NVME --> VM104
    HDD -->|Passthrough mp0| LXC100

    classDef hw fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef vm fill:#0F6E56,stroke:#16A085,color:#fff;
    class CPU,RAM,NVME,HDD hw;
    class LXC100,LXC103,VM104 vm;
```

<Warning>
Le firewall Proxmox 3 niveaux (node → datacenter → VM) n'est **pas encore configuré**. À faire avant toute exposition publique supplémentaire. Ordre impératif : règles niveau nœud d'abord, vérifier l'accès GUI+SSH immédiatement après activation, garder la console web ouverte pendant l'opération.
</Warning>

## Repos APT

Le format `deb822` (`.sources`) est utilisé sur PVE9, pas l'ancien `pve-enterprise.list` :

```bash
# /etc/apt/sources.list.d/pve-enterprise.sources → Enabled: false
# /etc/apt/sources.list.d/proxmox.sources → pve-no-subscription, Enabled: true
```

## Guests hébergés

| VMID | Nom | Type | Rôle |
|---|---|---|---|
| 100 | ims-nas | LXC privilégié | Stockage NFS/SMB |
| 103 | ims-pbs | LXC privilégié | Sauvegardes |
| 104 | ims-coolify | VM | Orchestration Docker |
| 101 | vm-test | VM | Test, non utilisé en prod |
| 102 | ims-windows | VM | Environnement Windows |
| 9000 | ubuntu-2404-template | Template | Base pour clonage de VM Ubuntu |

## Autostart et ordre de boot

```mermaid
sequenceDiagram
    autonumber
    participant Host as 🖥️ Proxmox Host (MS-01)
    participant NAS as 📁 LXC 100 (IMS-NAS)
    participant PBS as 💾 LXC 103 (IMS-PBS)
    participant Coolify as 🚀 VM 104 (IMS-Coolify)

    Note over Host: Démarrage de l'hyperviseur (PVE 9.2.3)
    Host->>NAS: Startup Order 1 (up=15s)
    Note over NAS: Initialisation MergerFS + NFS Exports
    Host->>PBS: Startup Order 2 (up=10s)
    Note over PBS: Montage NFS (/mnt/pbs-datastore via 10.10.10.1)
    Host->>Coolify: Startup Order 3 (up=20s)
    Note over Coolify: Montages NFS + Démarrage Stack Docker (Traefik, Authentik...)
```

<Check>
Validé par un reboot complet réel du host — les trois guests de production redémarrent automatiquement dans le bon ordre.
</Check>

```bash
# NAS en premier (les autres en dépendent via NFS)
pct set 100 -onboot 1 -startup order=1,up=15

# PBS ensuite
pct set 103 -onboot 1 -startup order=2,up=10

# Coolify en dernier
qm set 104 --onboot 1 --startup order=3,up=20
```

## Monitoring bas niveau — contrainte importante

<Warning>
**Tout outil nécessitant un accès device bloc direct (ioctl ATA/SCSI) doit tourner sur le host, jamais dans un LXC avec passthrough mountpoint.** Le passthrough (`mp0`) donne accès au filesystem monté, pas au device brut. Concerne `smartd` et `hd-idle` — voir [Dépannage courant](/procedures/depannage-courant) pour le détail complet de cette découverte.
</Warning>

```bash
# smartd et hd-idle tournent tous les deux sur le HOST, ciblant le disque NAS
systemctl status smartd hd-idle --no-pager
smartctl -H /dev/disk/by-id/ata-APPLE_HDD_ST3000DM001_Z1F3N0NZ
hdparm -C /dev/disk/by-id/ata-APPLE_HDD_ST3000DM001_Z1F3N0NZ   # doit passer "standby" après 30min d'inactivité
```

## GPU (iGPU Iris Xe) — non exploité

Passthrough prévu pour le transcodage matériel Jellyfin (HomeFlix), volontairement différé. Nécessite :
1. Activation IOMMU (BIOS + kernel cmdline `intel_iommu=on`) — **reboot host requis**
2. Ajout `hostpci0` dans la config de la VM cible
3. Installation des drivers i965/intel media dans le guest

<Card title="Suivi de cette tâche" icon="microchip" href="/services/homeflix#gpu-passthrough">
  Voir HomeFlix pour le contexte d'usage.
</Card>
