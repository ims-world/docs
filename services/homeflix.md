---
title: "HomeFlix"
description: "Stack médias — Jellyfin, Sonarr, Radarr, Prowlarr, qBittorrent, Gluetun"
icon: "clapperboard"
iconType: "duotone"
---

![HomeFlix Stack Médias](/assets/homeflix-banner.png)

## Accès Rapides aux Services

<Tabs>
  <Tab title="🌐 Publics (Internet)">
    <CardGroup cols={2}>
      <Card title="Jellyfin (Streaming)" icon="play" href="https://homeflix.ims-world.fr">
        Interface principale de streaming vidéo 4K/HDR.
      </Card>
      <Card title="Jellyseerr (Demandes)" icon="film" href="https://videoclub.ims-world.fr">
        Portail de demande de films et séries.
      </Card>
    </CardGroup>
  </Tab>
  <Tab title="🔒 Tailnet Only (VPN)">
    <CardGroup cols={2}>
      <Card title="Radarr (Films)" icon="video" href="https://radarr.ims-world.fr">
        Gestion et automatisation des films.
      </Card>
      <Card title="Sonarr (Séries)" icon="tv" href="https://sonarr.ims-world.fr">
        Gestion et automatisation des séries.
      </Card>
      <Card title="Prowlarr (Indexers)" icon="magnifying-glass" href="https://prowlarr.ims-world.fr">
        Gestionnaire centralisé des indexeurs torrent.
      </Card>
      <Card title="qBittorrent (Client)" icon="download" href="https://qbit.ims-world.fr">
        Client torrent routé via VPN Gluetun.
      </Card>
    </CardGroup>
  </Tab>
  <Tab title="💻 Administration CLI">
    ```bash
    # Se connecter à la VM Coolify
    ssh root@100.64.0.5
    
    # Accéder au dossier du service HomeFlix
    cd /data/coolify/services/w39uebmcnse7yctsft8hzed8/
    
    # Inspecter les logs des 9 containers
    docker compose logs -f --tail=100
    ```
  </Tab>
</Tabs>

## Topologie de la Stack HomeFlix (9 Containers)

```mermaid
graph TB
    subgraph INGRESS ["🌐 Accès External & VPN Middleware"]
        PUB_CLIENT["Client Public"]
        VPN_CLIENT["Client Tailnet VPN"]
        TRAEFIK["Traefik v3.7 Proxy"]
    end

    subgraph HOMEFLIX_STACK ["🎬 Stack HomeFlix (VM 104 Docker)"]
        subgraph PUBLIC_SERVICES ["Services Exposés"]
            JELLYFIN["Jellyfin (homeflix.ims-world.fr)"]
            JELLYSEERR["Jellyseerr (videoclub.ims-world.fr)"]
        end

        subgraph VPN_RESTRICTED ["Services Filtrés (vpn-only)"]
            RADARR["Radarr (radarr.ims-world.fr)"]
            SONARR["Sonarr (sonarr.ims-world.fr)"]
            PROWLARR["Prowlarr (prowlarr.ims-world.fr)"]
            QBIT["qBittorrent (qbit.ims-world.fr)\n[HostHeaderValidation=false]"]
        end

        subgraph INTERNAL_STACK ["Infrastructure Interne"]
            GLUETUN["Gluetun VPN (ProtonVPN WireGuard)"]
            RECYCLARR["Recyclarr (TRaSH-Guides Sync Cron)"]
            AUTOHEAL["Autoheal (qBit Monitoring)"]
        end
    end

    subgraph NAS_STORAGE ["📁 Stockage Shared NAS (NFS /mnt/nas-storage)"]
        CONFIG_SSD["SSD Local VM (configs & bases SQLite)"]
        MEDIA_NAS["NAS HDD (/mnt/storage/homeflix)"]
        MOOV["movies/"]
        SERIES["series/"]
        DL["downloads/ (SAME FS -> Hardlinks OK!)"]
    end

    PUB_CLIENT -->|HTTPS| TRAEFIK
    VPN_CLIENT -->|HTTPS (100.64.0.x)| TRAEFIK

    TRAEFIK --> JELLYFIN
    TRAEFIK --> JELLYSEERR
    TRAEFIK -->|vpn-only| RADARR
    TRAEFIK -->|vpn-only| SONARR
    TRAEFIK -->|vpn-only| PROWLARR
    TRAEFIK -->|vpn-only| QBIT

    QBIT <-->|Réseau via Container| GLUETUN
    PROWLARR --> QBIT
    RADARR --> PROWLARR
    SONARR --> PROWLARR
    RADARR -->|Hardlinks| MEDIA_NAS
    SONARR -->|Hardlinks| MEDIA_NAS

    MEDIA_NAS --- MOOV
    MEDIA_NAS --- SERIES
    MEDIA_NAS --- DL

    classDef pub fill:#F97316,stroke:#FB923C,color:#fff;
    classDef vpn fill:#1a2b3c,stroke:#F97316,color:#fff;
    classDef nas fill:#2c3e50,stroke:#34495e,color:#fff;
    class JELLYFIN,JELLYSEERR pub;
    class RADARR,SONARR,PROWLARR,QBIT,GLUETUN vpn;
    class CONFIG_SSD,MEDIA_NAS,MOOV,SERIES,DL nas;
```

## Vue d'ensemble

<Info>
Stack Médias de Production — 1.6 To de données, 9 conteneurs interdépendants, hardlinks optimisés sur le NAS sans duplication physique.
- **UUID Coolify** : `w39uebmcnse7yctsft8hzed8`
- **Chemin d'accès sur la VM** : `/data/coolify/services/w39uebmcnse7yctsft8hzed8/`
</Info>

| Service | Domaine | Rôle |
|---|---|---|
| Jellyfin | `homeflix.ims-world.fr` | Diffusion médias, exposé publiquement |
| Jellyseerr | `videoclub.ims-world.fr` | Demandes utilisateurs, exposé publiquement |
| qBittorrent | `qbit.ims-world.fr` | Client torrent, **Tailscale-only** |
| Prowlarr | `prowlarr.ims-world.fr` | Agrégateur d'indexers, **Tailscale-only** |
| Radarr | `radarr.ims-world.fr` | Gestion films, **Tailscale-only** |
| Sonarr | `sonarr.ims-world.fr` | Gestion séries, **Tailscale-only** |
| Gluetun | interne | VPN (ProtonVPN WireGuard), kill-switch pour qBittorrent |
| Recyclarr | interne | Sync profils qualité TRaSH-Guides, cron 04h00 |
| Autoheal | interne | Redémarre qBittorrent si Gluetun tombe |

## Architecture Stockage — Répartition SSD/NAS

<Warning>
`config/`+`cache/` sur SSD local (petits fichiers, accès fréquent), `movies/`+`series/`+`downloads/` groupés sur le NAS.

**Contrainte technique impérative** : les hardlinks ne traversent pas deux filesystems différents — `downloads/` DOIT rester sur le même volume que `movies/`/`series/`, sinon Radarr/Sonarr ne peuvent plus faire de hardlink à l'import (copie complète à la place, double l'espace utilisé).
</Warning>

```
./config           SSD local (VM) — configs, bases SQLite *arr, qBittorrent
./cache            SSD local (VM) — thumbnails Jellyfin
/mnt/nas-storage/homeflix/movies      NAS — hardlinks garantis avec downloads/
/mnt/nas-storage/homeflix/series      NAS
/mnt/nas-storage/homeflix/downloads   NAS
```

## Détails Techniques & Troubleshooting

<AccordionGroup>
  <Accordion title="Audit & Gestion des Hardlinks (rdfind)">
    <Steps>
      <Step title="Audit des doublons réels (rdfind)">
        ```bash
        rdfind -dryrun true -makehardlinks false -makeresultsfile true movies series downloads
        ```
      </Step>
      <Step title="Comptage des fichiers en hardlink">
        ```bash
        find /mnt/nas-storage/homeflix/ -type f -links +1 | wc -l   # nombre de fichiers hardlinkés
        ```
      </Step>
    </Steps>
    <Warning>
    **Ménage à faire via Radarr/Sonarr, jamais en supprimant les fichiers à la main** — ces outils gèrent proprement le nettoyage DB + fichiers + torrents associés. Une suppression manuelle désynchronise la base de l'appli.
    </Warning>
  </Accordion>

  <Accordion title="Métrologie Disque — Piège du vs df">
    <Warning>
    `du -sh` sur le dossier homeflix peut afficher artificiellement un volume supérieur (ex: 2.7 To) en surcomptant les hardlinks. Toujours croiser avec **`df -h`** pour connaître l'occupation réelle au niveau bloc (1.6 To réels).
    </Warning>

    **`du` additionne la taille de chaque fichier à chaque fois qu'il le rencontre — y compris pour des hardlinks — donc il sur-compte. `df` seul reflète l'espace disque réel.**

    Toujours valider une migration avec hardlinks via `df`, jamais `du` seul.
  </Accordion>

  <Accordion title="Le grand troubleshooting qBittorrent (401 / 403)">
    Trois fausses pistes explorées avant la vraie cause :
    <Steps>
      <Step title="vpn-only middleware — éliminé">
        Code HTTP différent de celui attendu (403 vs 401 observé), et Radarr avec le même middleware fonctionnait.
      </Step>
      <Step title="Double router Traefik auto-généré — confirmé normal">
        Comparaison avec le Mac Mini a montré que ce doublon existe aussi en prod historique, pas une anomalie.
      </Step>
      <Step title="Instabilité Gluetun — détournement d'~1h de diagnostic">
        `DOT: 'on'` (DNS-over-TLS) timeoutait par intermittence à travers le tunnel ProtonVPN, causant des redémarrages VPN en boucle. Stabilisé seul après quelques minutes.
      </Step>
    </Steps>

    <Warning>
    **Cause réelle** : `WebUI\HostHeaderValidation` de qBittorrent (activé par défaut) rejette les requêtes dont le header `Host` ne correspond pas à un domaine autorisé — malgré un `ServerDomains=*` (wildcard) déjà en place.

    ```ini
    WebUI\HostHeaderValidation=false
    ```

    Compromis sécurité assumé (désactive une protection anti-DNS-rebinding) — risque jugé limité, l'accès étant déjà filtré par `vpn-only` en amont.
    </Warning>
  </Accordion>

  <Accordion title="Passthrough GPU (Iris Xe)">
    Volontairement différé — section `devices:` retirée du compose Jellyfin. Nécessite IOMMU + reboot host. Voir [Host Proxmox](/infrastructure/proxmox-host#gpu-iris-xe-non-exploite).
  </Accordion>
</AccordionGroup>
