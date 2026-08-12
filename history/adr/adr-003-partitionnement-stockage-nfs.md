---
title: "ADR-003 — Partitionnement du Stockage NFS (storage vs storage-hot)"
description: "Décision d'architecture concernant le découpage du stockage partagé du NAS IMS-NAS"
icon: "hard-drive"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-12)*

---

## Contexte

Les conteneurs applicatifs hébergés sur la VM IMS-Coolify consomment des données aux profils d'E/S et d'accès très hétérogènes :
- Média volumineux à lecture séquentielle et stockage froid (films/séries HomeFlix, archives RAW Photoprism).
- Fichiers fréquemment accédés et miniatures générées en continu (Immich photo/vidéo upload, thumbnails, transcodages).

Un point de montage NFS unique risquait de provoquer de la contention E/S sur le pool de disques du NAS.

---

## Décision

Nous avons formalisé un **découpage du stockage partagé NFS en 2 tiers distincts** entre le NAS (LXC 100) et la VM Coolify (VM 104) :

1. **`storage` (`/mnt/nas-storage`)** :
   - Usage : Stockage de masse, médias et archives froides (HomeFlix Jellyfin, Radarr, Sonarr, qBittorrent, téléchargements).
   - Accès : Principalement séquentiel.

2. **`storage-hot` (`/mnt/nas-hot`)** :
   - Usage : Données applicatives chaudes à accès fréquent et petites E/S répétitives (Bibliothèque photo/vidéo Immich `immich-data`).
   - Accès : Aléatoire et réactif.

---

## Conséquences

### Positives
- **Isolation des performances E/S** : Les lectures/écritures intensives d'Immich n'impactent pas le débit de streaming de HomeFlix.
- **Organisation claire des volumes Docker** : Séparation stricte des bind-mounts applicatifs.
- **Règles de sauvegarde différenciées** : Fréquence et rétention de snapshot PBS ajustables selon le tier de stockage.

### Négatives / Contraintes
- Nécessite de maintenir 2 points de montage NFS distincts dans `/etc/fstab` de la VM Coolify avec l'option `nofail,_netdev`.
