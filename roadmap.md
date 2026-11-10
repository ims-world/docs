---
title: "Feuille de route"
description: "Chantiers ouverts par priorité, idées à évaluer, et chantiers réalisés récemment."
icon: "list-check"
iconType: "duotone"
last_reviewed: "2026-11-08"
---

Une ligne par chantier. Un chantier terminé passe dans **Réalisés**, avec un lien vers la page qui le décrit ; le détail va dans la [chronologie](/history/chronologie).

## Priorité haute

| Chantier | Pourquoi | Où |
| --- | --- | --- |
| Protéger les données du NAS : photos Immich, dépôts Forgejo, documents Paperless-ngx, originaux PhotoPrism | Aucune copie aujourd'hui ; perte définitive si un disque lâche | NAS, [politique de sauvegarde](/procedures/politique-sauvegardes#ce-qui-nest-pas-sauvegardé) |
| Copie hors site des sauvegardes PBS | Datastore et médiathèque sur le même HDD. Piste : Raspberry Pi avec disque, en synchronisation PBS | PBS |
| Export nocturne de Paperless-ngx vers le HDD du NAS | Seule copie des documents en attendant le NAS de sauvegarde ; préalable à la destruction de papiers et à l'import Google Drive | [Paperless-ngx](/services/paperless#données-et-sauvegarde) |
| QDevice Corosync | Deux votes seulement : la perte d'un nœud fait perdre le quorum | Raspberry Pi |

## Priorité moyenne

| Chantier | Pourquoi | Où |
| --- | --- | --- |
| Passer n8n en 3.x | Après la sortie stable (décembre 2026) : rapport de migration, protection SSRF qui bloque le tailnet, sauvegarde préalable ; passage sans retour | [n8n](/services/n8n) |
| Passer les applications forward-auth sur l'outpost intégré d'Authentik | Retirer le socket Docker monté en `root` dans le worker. Recréer dans le provider File le routeur `/outpost.goauthentik.io/` de chaque application, en priorité élevée et avec `vpn-only` là où il s'applique | [Authentik](/services/authentik) |
| Utilisateur PBS dédié pour `pbs-coolify` | Le stockage se connecte avec `root@pam` | PBS |
| Harmoniser les DNS : Quad9 (`9.9.9.9`) pour les invités | Trois DNS coexistent : Bbox (nœuds, worker), Quad9 (NAS, PBS), Cloudflare (VM de `just vm`). À faire : `pct set 105 -nameserver 9.9.9.9` puis redémarrage du worker ; `vm_nameserver: 9.9.9.9` dans ims-ops ; domaine de recherche de `pve` : `ims-world.fr` au lieu de `ims.local` | Nœuds, ims-ops |
| Pare-feu Proxmox | Non activé ; règles par nœud d'abord, console ouverte pendant l'activation | Nœuds |
| Raccorder Windows (VM 102) en RDP dans Termix | Démarrage à la demande à étudier (plugin Proxmox Power) | [Termix](/services/termix) |
| Renouveler la clé Headscale de `infra` | Expire le 18/10/2027 | [Headscale](/reseau/headscale-headplane) |
| Domotique : Hue, HomeKit, téléviseurs, application mobile, clé Zigbee | Intégrations à configurer | [Home Assistant](/infrastructure/vm-home-assistant) |
| Paperless-ngx : Tika et Gotenberg, puis import en masse | Documents Office des exports Google Drive ; environ 0,5 Go de RAM au repos | [Paperless-ngx](/services/paperless) |
| Paperless-ngx : alerte Ntfy sur échec d'ingestion | Paperless ne notifie pas ses échecs ; alerte Grafana sur Loki | [Paperless-ngx](/services/paperless) |
| Montage de l'extension du rack | Impression terminée | [Labrax](/infrastructure/labrax) |

## À évaluer

| Idée | Note |
| --- | --- |
| Accès Intel vPro / AMT sur le MS-01 | Console matérielle sans sortie vidéo |
| DNS secondaire (AdGuard Home, Pi-hole) | Sur le Mac Mini ou le Raspberry Pi |
| Exporteur Prometheus pour Jellyfin | Sessions, transcodages |
| Forward-auth Authentik devant Radarr, Sonarr, Prowlarr, qBittorrent | Leur connexion native est la seule protection face aux appareils du LAN |
| Caddy à la place de Traefik | Essai : `xcaddy` avec OVH et CrowdSec |
| Portail self-service de VM temporaires (IMS-CLOUD) | Au-dessus de `just vm` et de l'API Proxmox |
| Certificats SSH courts (OPKSSH) | À la place de la clé statique de Termix |
| Second HDD | Disque ext4 indépendant ([procédure](/procedures/ajout-nouveau-disque#option-a-ajouter-un-disque-de-capacité)) |
| Sonde de détection d'intrusion réseau, Sentryx | — |
| Ingestion automatique dans Paperless-ngx | Dossier de dépôt partagé, boîte e-mail dédiée |
| Applications : Homepage, Imgcompress, Strix, Semgrep, Dawarich, Keep It Shot, Cap, Cyber Strike AI, Page Agent, Meetily | Une à la fois, selon le besoin |

## Réalisés

| Date | Chantier | Page |
| --- | --- | --- |
| 10/11/2026 | Paperless-ngx : gestion documentaire, tailnet, OIDC Authentik avec groupes | [Paperless-ngx](/services/paperless) |
| 08/11/2026 | Mises à jour de seize services, dont trois de sécurité ; commande `just app-update` | [Administration avec ims-ops](/procedures/administration-ims-ops#mettre-à-jour-un-service-coolify) |
| 06/11/2026 | Expressions `wud.tag.include` vérifiées sur tous les conteneurs surveillés : 18 corrigées, dont Forgejo épinglé en 11.0.16 | [WUD](/services/whatsupdocker) |
| 04/11/2026 | HomeFlix en deux ressources Coolify, Bazarr, notifications Ntfy, redirection de port automatique, images à jour | [HomeFlix](/services/homeflix) |
| 03/11/2026 | Contrôles de sécurité : routeurs de `vpn-only.yaml`, TXT `_acme-challenge`, observation CrowdSec close | [Traefik](/reseau/traefik-proxy) |
| 03/11/2026 | Identifiants OVH du proxy sortis de Coolify (fichiers `_FILE`), nouvelle clé limitée à la zone DNS | [Traefik](/reseau/traefik-proxy#certificats) |
| 03/11/2026 | Images épinglées : Vaultwarden, Zipline, IT-Tools, Stirling PDF ; plus aucun service en `latest` | [WUD](/services/whatsupdocker) |
| 03/11/2026 | VM 102 et `lab-elo` ajoutées au pool de sauvegarde | [Politique de sauvegarde](/procedures/politique-sauvegardes) |
| 03/11/2026 | Alerte hors du homelab : battement de cœur des nœuds et de la VM 104 vers Healthchecks.io | [ADR-015](/history/adr/adr-015-alerte-externe-battement-de-coeur) |
| 01/11/2026 | Premiers tests de restauration (worker 105, VM 104) et commandes d'exploitation ims-ops | [Plan de reprise](/procedures/plan-reprise-activite-pra) |
| 25/10/2026 | Refonte de la documentation | [Contribuer](/reference/contribuer) |
| 25/10/2026 | Worker 105 repassé en IP fixe (`192.168.1.198`) | [Worker 105](/infrastructure/lxc-coolify-worker) |
| 24/10/2026 | Sauvegarde par pool et alertes Ntfy, y compris le worker et Home Assistant | [ADR-014](/history/adr/adr-014-sauvegarde-par-pool-unique) |
| 24/10/2026 | Création et destruction de VM par ims-ops, template 9001 | [Administration avec ims-ops](/procedures/administration-ims-ops) |
| 24/10/2026 | Termix raccordé au parc | [Termix](/services/termix) |
| 24/10/2026 | Sonde Uptime Kuma et bail réservé pour Home Assistant | [Home Assistant](/infrastructure/vm-home-assistant) |
| 23/10/2026 | Socle commun appliqué par ims-ops | [Socle commun](/infrastructure/socle-commun) |
| 23/10/2026 | Retrait de MergerFS | [ADR-012](/history/adr/adr-012-retrait-de-mergerfs) |
| 22/10/2026 | NAS et worker dans le tailnet | [Architecture réseau](/reseau/architecture-reseau) |
| 22/09/2026 | n8n | [n8n](/services/n8n) |
| 14/09/2026 | Home Assistant en VM, WUD | [Home Assistant](/infrastructure/vm-home-assistant), [WUD](/services/whatsupdocker) |
| 13/09/2026 | Enrôlement au tailnet par Authentik | [Headscale](/reseau/headscale-headplane) |
| 23/08/2026 | Cluster à deux nœuds | [Mac Mini](/infrastructure/mac-mini) |
| 22/08/2026 | CrowdSec et pare-feu applicatif | [CrowdSec](/reseau/crowdsec) |

Les chantiers plus anciens sont dans la [chronologie](/history/chronologie).
