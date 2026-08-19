---
title: "Politique de Sauvegarde & Tâches Planifiées"
description: "Architecture de protection des données, anti-circularité, chronologie nocturne et tolérance aux coupures"
icon: "shield-check"
iconType: "duotone"
last_reviewed: "2026-08-17"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active</Badge>

<Info>
Cette page constitue la **vue consolidée de l'architecture de sauvegarde et de la chronologie nocturne** du homelab IMS-WORLD. Elle recense l'ensemble des tâches planifiées pour éviter les chevauchements de ressources et faciliter les diagnostics.
</Info>

---

## 📌 Principes d'Architecture & Règles Fondamentales

### 1. Découplage Coolify / Proxmox
**Coolify ne gère aucune sauvegarde nativement** (aucun job de backup applicatif automatisé n'est configuré dans l'UI Coolify). La sécurité et l'historisation des données reposent intégralement sur **Proxmox Backup Server (PBS)**, qui effectue un snapshot dédupliqué quotidien à chaud de la **VM 104 (IMS-Coolify)** complète au niveau hyperviseur. Ce snapshot englobe à la fois le système d'exploitation, l'arborescence `/data/coolify/` et l'intégralité des **volumes Docker nommés** (`/var/lib/docker/volumes/`).

### 2. Règle d'Anti-Circularité
Le NAS (LXC 100) et PBS (LXC 103) ne sont **jamais** sauvegardés dans le datastore PBS qu'ils hébergent ou gèrent eux-mêmes. Leurs sauvegardes s'effectuent via l'utilitaire `vzdump` directement de l'hôte Proxmox vers le stockage NVMe local (`local` / `/var/lib/vz/dump/`).

### 3. Exclusions de Sauvegarde & Stabilité NFS
Le conteneur NAS (**LXC 100**) est **exclu des sauvegardes automatiques quotidiennes**. Le redémarrage de LXC 100 réinitialise l'instance FUSE MergerFS, ce qui invalide définitivement les descripteurs de fichiers NFS (`Stale filehandle`) côté VM Coolify. La sauvegarde de la LXC 100 (rootfs 8 Go quasiment statique) s'effectue uniquement de manière manuelle avant toute opération de maintenance en `--mode stop`. Voir le [Post-Mortem du 19/08/2026](/history/incidents/2026-08-19-stale-nfs-filehandle-jellyfin-mergerfs).

---

## 🕒 Chronologie Nocturne Consolidée

| Heure | Job | Emplacement | Détail & Comportement |
|---|---|---|---|
| **02:00** | Backup PBS — VM 104 (Coolify) | Host Proxmox → PBS (NFS) | Mode Snapshot, incrémental dédupliqué |
| **03:00** | Backup local — LXC 103 (PBS) | Host Proxmox → Storage `local` (NVMe) | Mode Snapshot, rootfs uniquement (8 Go alloués) |
| **04:00** | Verify PBS | LXC PBS (103) | Vérification d'intégrité anti-corruption du datastore PBS |
| **~04:00** | Sync mirrors GitHub (Forgejo) | VM Coolify | Synchronisation automatique des dépôts miroirs secondaires |
| **Manual** | Backup local — LXC 100 (NAS) | Host Proxmox → Storage `local` (NVMe) | **Manuel uniquement** (`--mode stop` lors des maintenances) |

<Info>
**Exclusion de LXC 100 de la Chronologie Nocturne** : Le job de sauvegarde automatique `backup-6b92bcb1-ad11` sur LXC 100 a été désactivé pour éliminer les invalidations NFS intempestives sur Jellyfin, HomeFlix et PhotoPrism.
</Info>

---

## ⚙️ Détail Technique des Jobs & Pièges Système

### 1. Sauvegarde PBS — VM 104 (IMS-Coolify)

```bash
# Configuré via Datacenter → Backup dans l'UI Proxmox
vzdump 104 --storage pbs-coolify --mode snapshot
```
- **Storage** : `pbs-coolify` (datastore PBS via NFS, forcé en NFSv3 — voir l'[ADR-003](/history/adr/adr-003-partitionnement-stockage-nfs)).

### 2. Sauvegarde Locale NVMe — LXC 103 (PBS) & LXC 100 (NAS Manuel)

```bash
# LXC 100 (NAS) — Mode STOP obligatoire
vzdump 100 --storage local --mode stop

# LXC 103 (PBS) — Mode SNAPSHOT
vzdump 103 --storage local --mode snapshot
```

<Warning>
**Storage `local` (type `dir`, `/var/lib/vz`)** : Ne pas utiliser `local-lvm` (LVM-Thin refuse le stockage de type archive `backup` avec l'erreur `wrong content type`).
</Warning>

<Warning>
**Workaround Freeze MergerFS (LXC 100)** : En mode `snapshot`, la sauvegarde du LXC 100 reste bloquée indéfiniment sur la commande `freeze guest filesystem` en raison de l'interaction entre FUSE/MergerFS et le noyau. Le passage en `--mode stop` résout définitivement le blocage (arrêt propre de ~10 secondes, sauvegarde, redémarrage automatique).
</Warning>

<Info>
**Warning non-bloquant (LXC 103)** : Un avertissement `failed to open /mnt/pbs-datastore: Stale file handle` apparaît lors du snapshot du LXC 103 mais n'empêche pas la réussite du backup du rootfs.
</Info>

*Taille moyenne des archives : ~2 à 2.5 Go par conteneur, temps d'exécution observés : 7 à 10 secondes.*

---

## 📊 Matrice de Rétention & Restauration

| Cible | Type de Backup | Stockage Cible | Rétention | Procédure de Restauration |
|---|---|---|---|---|
| **VM 104 (Coolify)** | Snapshot PBS | PBS Datastore (NFS) | 7 quotidiens, 4 hebdomadaires | [Restaurer la VM Coolify (PRA)](/procedures/plan-reprise-activite-pra#étape-3--restauration-de-la-vm-coolify-vm-104) |
| **LXC 103 (PBS)** | vzdump Snapshot | Host NVMe (`local`) | 3 roulements | `pctrestore <vmid> /var/lib/vz/dump/vzdump-lxc-103-*.tar.zst` |
| **LXC 100 (NAS)** | vzdump Stop | Host NVMe (`local`) | 3 roulements | `pctrestore <vmid> /var/lib/vz/dump/vzdump-lxc-100-*.tar.zst` |

---
*Dernière révision de cette fiche : 17 août 2026*
