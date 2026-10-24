---
title: "ims-nas (LXC 100)"
description: "Stockage central du homelab : partages NFS pour les invités, SMB pour le LAN, sur un HDD de 3 To et un SSD de 4 To."
icon: "hard-drive"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

import { ips, hardware } from "/snippets/variables.mdx";

## Fiche

| Propriété | Valeur |
| --- | --- |
| Type | Conteneur LXC privilégié (requis pour le serveur NFS et les disques transmis) |
| Nœud et VMID | `pve`, 100 |
| Noms | Proxmox `ims-nas` · inventaire `ims-nas` · tailnet `ims-pve-100-nas` |
| OS | Debian |
| CPU et RAM | 2 cœurs · 1 Go |
| Disques | HDD 3 To (`mp0`) et SSD 4 To (`mp1`), sur {hardware.sataCard} |
| IP LAN | <code>{ips.nasLan}</code> ; réseau interne `vmbr1` : `10.10.10.1` |
| IP tailnet | <code>{ips.nas}</code> |
| Sauvegarde | Hors pool `backup` (voir plus bas) |
| Statut | En service |

## Ce qui tourne dessus

| Partage | Protocole | Clients |
| --- | --- | --- |
| `/mnt/storage` | NFS lecture-écriture sur `10.10.10.0/24` | VM 104, PBS |
| `/mnt/storage` | NFS lecture seule sur `192.168.1.0/24` | LAN |
| `/mnt/storage-hot` | NFS lecture-écriture sur `10.10.10.0/24` | VM 104 |
| `storage` | SMB, compte `cmolotkoff` | LAN et tailnet : <code>smb://{ips.nasLan}/storage</code> |

## Disques et montages

| Disque | Référence | Monté sur | Exposé comme | Contenu |
| --- | --- | --- | --- | --- |
| HDD 3 To, ext4 | Seagate ST3000DM001 (`Z1F3N0NZ`) | `/mnt/disk1` (`mp0`) | `/mnt/storage` (bind-mount) | Médiathèque, PhotoPrism, documents, datastore PBS |
| SSD 4 To, ext4 | Samsung 870 EVO (`S758NX0W713130Z`) | `/mnt/ssd-hot-raw` (`mp1`) | `/mnt/storage-hot` (bind-mount) | Immich, Forgejo |

```text
/mnt/storage       → /mnt/disk1        documents/  backups/  homeflix/  photoprism-data/
/mnt/storage-hot   → /mnt/ssd-hot-raw  immich-data/  forgejo-data/
```

Le découpage en deux niveaux est expliqué dans l'[ADR-003](/history/adr/adr-003-partitionnement-stockage-nfs). Côté VM 104, les partages sont montés sur `/mnt/nas-storage` et `/mnt/nas-hot`.

<Warning>
Ne jamais exporter en NFS un système de fichiers FUSE (MergerFS ou équivalent). FUSE régénère ses identifiants à chaque redémarrage, ce qui invalide les descripteurs NFS des clients (`ESTALE`). C'est la cause de l'[incident du 23/10/2026](/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix), résolu par le passage aux bind-mounts ext4.
</Warning>

## Exports NFS

```text
/mnt/storage      10.10.10.0/24(rw,no_root_squash)
/mnt/storage      192.168.1.0/24(ro,root_squash)
/mnt/storage-hot  10.10.10.0/24(rw,no_root_squash)
```

## Détection des doublons

`jdupes` est installé pour le rapport de doublons de la médiathèque, lancé depuis ims-ops avec `just duplicates`. Le rapport est produit en lecture seule, directement sur le NAS. Il liste les fichiers identiques (liens physiques exclus), les titres présents en plusieurs fichiers, et les téléchargements reliés à aucun fichier de la bibliothèque.

La suppression se fait depuis qBittorrent (torrent et fichiers), jamais à la main sur le disque.

<Warning>
Ne pas utiliser `jdupes -L` sur la médiathèque : la commande remplace des fichiers suivis par qBittorrent, Radarr et Sonarr sans qu'ils le sachent.
</Warning>

## Particularités

- Le compte `cmolotkoff` (UID 1000) est aussi le propriétaire des fichiers partagés et l'utilisateur Samba. Son mot de passe système est verrouillé. **Ne jamais modifier son UID ni supprimer ce compte** : toutes les permissions des partages en dépendent.
- `smartd` et `hd-idle` ne peuvent pas tourner dans ce conteneur ; ils tournent sur le [nœud](/infrastructure/proxmox-host).
- Les deux disques sont montés avec `backup=0` : une sauvegarde du conteneur n'embarque jamais leur contenu.
- Écarts au [socle commun](/infrastructure/socle-commun) : envoi Alloy par `10.10.10.2`, paquet `jdupes`.

## Sauvegarde et reprise

Le NAS est exclu du pool `backup` : il héberge le datastore du PBS, qui ne peut pas se sauvegarder lui-même. Son système (environ 8 Go, quasi statique) se sauvegarde à la main avant une intervention :

```bash
vzdump 100 --mode stop --compress zstd --storage local
```

La médiathèque n'est pas sauvegardée (elle se retélécharge). Le datastore PBS partage le HDD avec elle : la copie hors site des sauvegardes est suivie dans la [feuille de route](/roadmap).

## Supervision

Agent Alloy, collecteur `hwmon` désactivé. Santé des disques relevée par le nœud.

## Voir aussi

- [Ajouter un disque](/procedures/ajout-nouveau-disque)
- [Politique de sauvegarde](/procedures/politique-sauvegardes)
