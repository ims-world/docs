---
title: "ADR-012 — Retrait de MergerFS"
description: "Les partages NFS du NAS reposent sur des bind-mounts ext4 directs ; aucun système de fichiers FUSE n'est exporté en NFS."
icon: "scale-balanced"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

## Statut

Accepté — 23/10/2026. Remplace l'[ADR-007](/history/adr/adr-007-calcul-inodes-mergerfs-path-hash).

## Contexte

`/mnt/storage` était un pool MergerFS (FUSE), exporté en NFS vers la VM 104 et le PBS. Le pool ne comptait qu'un disque. À chaque redémarrage du processus FUSE (redémarrage ou sauvegarde du NAS), les identifiants internes changeaient et les descripteurs NFS des clients devenaient invalides (`ESTALE`) :

- août 2026 : lectures Jellyfin impossibles après chaque sauvegarde du NAS ([incident du 19/08](/history/incidents/2026-08-19-stale-nfs-filehandle-jellyfin-mergerfs)) ;
- octobre 2026 : cinq nuits de sauvegardes PBS en échec, puis HomeFlix coupé ([incident du 23/10](/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix)).

Les options `inodecalc=path-hash`, `use_ino` ou `fsid=` ne rendent pas les descripteurs persistants à travers un redémarrage de FUSE.

## Décision

- `/mnt/storage` devient un bind-mount direct du disque ext4 (`/mnt/disk1`), sur le modèle de `/mnt/storage-hot`.
- Les exports NFS et les points de montage des clients restent identiques.
- Aucun système de fichiers FUSE n'est exporté en NFS. Un disque supplémentaire reste un système de fichiers indépendant, exporté séparément ([Ajouter un disque](/procedures/ajout-nouveau-disque)).

## Alternatives écartées

| Option | Raison du rejet |
| --- | --- |
| Garder MergerFS et ajouter un script de reconnexion | Traite le symptôme ; un échec du script recrée la panne |
| Options MergerFS (`inodecalc`, `fsid`, NFS v3) | Testées : ne survivent pas à un redémarrage de FUSE |
| Exporter chaque disque et agréger côté client | Même problème déplacé, et complexité pour un seul disque |

## Conséquences

- Les descripteurs NFS survivent à un redémarrage du NAS.
- Les inodes vus sur `/mnt/storage` sont les vrais inodes ext4 : l'ADR-007 n'a plus d'objet.
- Plus d'agrégation de capacité : un nouveau disque apparaît comme un second emplacement, à déclarer dans les applications, avec un dossier de téléchargement propre pour garder les liens physiques.
