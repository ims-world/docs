---
title: "HomeFlix"
description: "Médiathèque vidéo : Jellyfin et Jellyseerr en public, Radarr, Sonarr, Prowlarr et qBittorrent (derrière ProtonVPN) restreints au tailnet."
icon: "film"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v10.9.11"
---

## Rôle

Pile de neuf conteneurs : Jellyfin diffuse la médiathèque, Jellyseerr reçoit les demandes, Radarr et Sonarr cherchent films et séries par Prowlarr, qBittorrent télécharge à travers le tunnel ProtonVPN de Gluetun. Les fichiers terminés sont reliés à la bibliothèque par liens physiques, sans copie.

## Accès

<CardGroup cols={3}>
  <Card title="Jellyfin" icon="play" href="https://homeflix.ims-world.fr">Lecture</Card>
  <Card title="Jellyseerr" icon="film" href="https://videoclub.ims-world.fr">Demandes</Card>
  <Card title="Radarr" icon="video" href="https://radarr.ims-world.fr">Films (tailnet)</Card>
  <Card title="Sonarr" icon="tv" href="https://sonarr.ims-world.fr">Séries (tailnet)</Card>
  <Card title="Prowlarr" icon="magnifying-glass" href="https://prowlarr.ims-world.fr">Indexeurs (tailnet)</Card>
  <Card title="qBittorrent" icon="download" href="https://qbit.ims-world.fr">Téléchargements (tailnet)</Card>
</CardGroup>

```bash
# Depuis la VM 104
cd /data/coolify/services/w39uebmcnse7yctsft8hzed8/ && docker compose logs -f --tail=100
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `homeflix`, `videoclub` (publics) ; `radarr`, `sonarr`, `prowlarr`, `qbit` (tailnet) `.ims-world.fr` |
| Accès | Jellyfin : comptes natifs ; Jellyseerr : comptes Jellyfin ; outils : tailnet et formulaires natifs |
| Hôte | VM 104 |
| Coolify | UUID `w39uebmcnse7yctsft8hzed8`, chemin `/data/coolify/services/w39uebmcnse7yctsft8hzed8/` |
| Image | Jellyfin `v10.9.11`, Jellyseerr `v3.4.1`, Gluetun `qmcgaw/gluetun:v3.41.3` ; autres images dans le compose |
| Données | Configurations et bases SQLite sur la VM 104 ; médias sur le NAS (`/mnt/nas-storage/homeflix`) |
| Sauvegarde | Configurations avec la VM 104 ; médias non sauvegardés |
| Supervision | Sondes Uptime Kuma (Jellyfin, Jellyseerr en public ; outils par le réseau Docker) |
| Mise en service | Août 2026 |

## Fonctionnement

| Conteneur | Rôle |
| --- | --- |
| Jellyfin | Diffusion, transcodage matériel par l'iGPU (Intel Quick Sync) |
| Jellyseerr | Demandes de films et séries |
| Radarr, Sonarr | Recherche, téléchargement et classement des films et séries |
| Prowlarr | Indexeurs, partagés par Radarr et Sonarr |
| qBittorrent | Client torrent, dans le réseau de Gluetun (`network_mode: service:gluetun`) |
| Gluetun | Tunnel WireGuard ProtonVPN, coupe-circuit, redirection de port |
| Recyclarr | Profils de qualité TRaSH-Guides, synchronisés chaque jour à 04:00 |
| Autoheal | Redémarre qBittorrent si le tunnel tombe |

Les routeurs de Radarr, Sonarr, Prowlarr et qBittorrent sont dans `vpn-only.yaml` ; celui de qBittorrent pointe vers le conteneur Gluetun (port 8080), qui porte son réseau.

### Transcodage matériel

L'iGPU Iris Xe du MS-01 est attribuée à la VM 104 ([ADR-008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe)) et transmise au conteneur (`devices: /dev/dri:/dev/dri`). Dans Jellyfin, **Tableau de bord → Lecture → Transcodage** : **Intel QuickSync (QSV)**, périphérique `/dev/dri/renderD128`, H.264 et HEVC en décodage et encodage. Mesure à la mise en service : 29,7 fois le temps réel.

## Données et sauvegarde

```text
/data/coolify/services/w39uebmcnse7yctsft8hzed8/data/config   configurations et bases (VM 104)
/mnt/nas-storage/homeflix/movies       bibliothèque des films
/mnt/nas-storage/homeflix/series       bibliothèque des séries
/mnt/nas-storage/homeflix/downloads    téléchargements de qBittorrent
```

<Warning>
`downloads/` doit rester sur le même système de fichiers que `movies/` et `series/` : un lien physique ne traverse pas deux systèmes de fichiers. Sinon, Radarr et Sonarr copient les fichiers et l'espace occupé double.
</Warning>

Les configurations sont sauvegardées avec la VM 104. La médiathèque ne l'est pas : elle se retélécharge.

## Exploitation

<AccordionGroup>
  <Accordion title="Supprimer un film ou un épisode">
    Un fichier occupe de l'espace tant que l'un de ses liens existe.

    1. Radarr ou Sonarr : supprimer le fichier (lien dans `movies/` ou `series/`).
    2. qBittorrent : supprimer le torrent en cochant **Supprimer aussi les fichiers** (lien dans `downloads/`).

    Ne jamais supprimer à la main un fichier encore suivi par qBittorrent : le torrent passe en erreur.
  </Accordion>
  <Accordion title="Repérer les doublons et les téléchargements orphelins">
    Depuis ims-ops : `just duplicates`. Le rapport est calculé sur le NAS, sur le disque lui-même, et ne supprime rien. Voir [ims-nas](/infrastructure/ims-nas).
  </Accordion>
  <Accordion title="Débloquer un import manuel (« Unable to parse file »)">
    Le nom du fichier ne permet pas de déduire la qualité. Dans Radarr ou Sonarr : **Activity → Queue → Manual Import**, choisir la qualité réelle, vérifier que le mode d'import est **Hardlink** (jamais **Copy**), valider.
  </Accordion>
  <Accordion title="Vérifier le port redirigé par ProtonVPN">
    ```bash
    GLUETUN=$(docker ps --format '{{.Names}}' | grep "^gluetun" | head -1)
    docker exec "$GLUETUN" cat /tmp/gluetun/forwarded_port
    ```
    Le port doit correspondre au port d'écoute de qBittorrent (**Options → Connexion**).
  </Accordion>
  <Accordion title="Mesurer l'espace réellement occupé">
    `du` sur plusieurs dossiers séparés compte deux fois un fichier présent dans `downloads/` et dans la bibliothèque. Mesurer en une seule commande (`du -sh movies series downloads`) ou lire `df -h`.
  </Accordion>
</AccordionGroup>

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| Téléchargements mélangés à la racine de `downloads/` | Torrent ajouté sans catégorie | Une catégorie par usage dans qBittorrent (`movies`, `series`), chacune avec son dossier ; Radarr et Sonarr utilisent la leur |
| Épisode téléchargé mais non importé | Titre de l'épisode encore « TBA » chez le fournisseur de métadonnées | Attendre le titre définitif, ou régler **Episode Title Required** dans **Media Management** |
| Jellyfin ne démarre pas : `/dev/dri: no such file or directory` | Module `i915` absent après une mise à jour du noyau | Voir l'[incident du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri) |
| Lecture impossible, `Stale file handle` | Montage NFS périmé (avant le retrait de MergerFS) | Redémarrer les conteneurs concernés ; voir le [dépannage](/procedures/depannage-courant) |

## Voir aussi

- [ims-nas](/infrastructure/ims-nas)
- [Restreindre un service au tailnet](/procedures/securiser-service-vpn-only)
