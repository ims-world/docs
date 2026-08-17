---
title: "ADR-007 — Calcul d'Inodes MergerFS (path-hash) & Diagnostic de Hardlinks"
description: "Décision concernant l'impact de la politique inodecalc=path-hash sur la vérification des inodes et des hardlinks"
icon: "hard-drive"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-16)*

---

## Contexte

Le stockage partagé du NAS (`LXC 100`) repose sur une superposition de couches de système de fichiers :

```
Client (ex: VM IMS-Coolify)
  └─ Point de montage NFS /mnt/nas-storage (10.10.10.1:/mnt/storage)
       └─ NAS LXC 100 : Pool MergerFS /mnt/storage
            └─ Disque physique sous-jacent : /mnt/disk1
```

La configuration de production de MergerFS utilise l'option `inodecalc=path-hash`. Cette politique calcule un **inode virtuel déterministe basé sur le hash du chemin du fichier**, plutôt que de remonter l'inode réel du système de fichiers sous-jacent (ext4/ZFS).

En conséquence :
- Deux fichiers strictement hardlinkés au niveau du disque physique (possédant le même inode réel sur `/mnt/disk1`) affichent des **inodes virtuels différents** lorsqu'ils sont inspectés via `/mnt/storage` (sur le NAS) ou via `/mnt/nas-storage` (sur la VM Coolify par NFS).
- Les utilitaires comme `stat`, `ls -i` ou `du` héritent de ce biais et comptent les hardlinks comme des fichiers physiques distincts, faussant les mesures d'espace disque.

---

## Décision

1. **Conservation de `inodecalc=path-hash`** : Nous conservons cette politique sur le pool MergerFS pour garantir la stabilité du système de fichiers virtuel FUSE et éviter les collisions d'inodes lors de l'agrégation de plusieurs disques.
2. **Règle d'Or de Diagnostic** : **Tout diagnostic d'inode, de vérification de hardlink ou de mesure d'espace disque réel DOIT impérativement être exécuté en SSH direct sur le NAS LXC 100, en ciblant le point de montage du disque physique (`/mnt/disk1/...`)**.

```bash
# Entrer dans le LXC NAS 100 depuis l'hôte Proxmox
sudo pct enter 100

# ❌ NE JAMAIS DIAGNOSTIQUER ICI (MergerFS path-hash fausse l'inode) :
stat -c '%d:%i' /mnt/storage/homeflix/movies/Film.mkv

# ✅ TOUJOURS DIAGNOSTIQUER ICI (Disque physique réel) :
stat -c '%d:%i %s %n' /mnt/disk1/homeflix/movies/Film.mkv
```

Un hardlink est confirmé valide uniquement si la paire `device:inode` retournée sur `/mnt/disk1` est strictement identique entre les deux chemins comparés.

---

## Conséquences

### Positives
- **Fiabilité absolue des diagnostics** : Évite d'interpréter à tort des hardlinks valides comme des doublons ou des fichiers cassés.
- **Transparence sur l'espace disque réel** : Mesures d'occupation mémoire exactes via `/mnt/disk1`.

### Négatives / Contraintes
- Impossibilité de valider l'intégrité d'un hardlink depuis la VM Coolify ou via les points de montage NFS (obligation de passer par le NAS LXC 100).
