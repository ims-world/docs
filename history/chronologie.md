---
title: "Changelog"
description: "Chronologie du projet de migration Mac Mini → MS-01"
---

## 17/08/2026 — Migration PhotoPrism sur MS-01 & SSO OIDC

### 🆕 Nouveautés

- **PhotoPrism en production sur MS-01** — Migration réussie de la bibliothèque photo & studio d'archivage RAW sur `studio.ims-world.fr` (UUID Coolify `yfotvbtkqj8cqw5alox6gfpr`). Le domaine d'origine a été conservé à l'identique pour réutiliser le client OIDC Authentik `photo-prism` sans casser les redirections. Restauration confirmée de **24 565 photos** via dump/restore SQL MariaDB 11.8.8. Voir [PhotoPrism](/services/photoprism).
- **Architecture de stockage hybride** — Les originaux RAW (`originals/`, ~574 Go) et les métadonnées manuelles YAML (`storage/sidecar/`, 42 Go) résident sur le pool capacitif NFS HDD (`storage`), tandis que la base de données MariaDB tourne sur un **volume nommé Docker local** (`photoprism-mariadb-data`) sur le SSD de la VM Coolify pour garantir des I/O optimales. Voir [Architecture de Stockage](/services/photoprism#architecture-de-stockage-nfs-vs-volume-nomme-docker).
- **Ingestion de photos RAW via WebDAV** — Documentation de la procédure de dépôt de fichiers RAW via le serveur WebDAV natif de PhotoPrism (`/import/` et `/originals/`), avec pas-à-pas macOS/Windows, gestion des identifiants et commande CLI `photoprism import`. Voir [Ingestion de Photos & WebDAV](/services/photoprism#ingestion-de-photos--webdav).

### 🔧 Améliorations & Maintenance

- **Mise à niveau de Coolify en v4.3.6 & Workaround IHM** — Montée en version de l'orchestrateur Coolify vers la v4.3.6. Documentation de la procédure de secours `docker restart coolify-proxy` requise pour rétablir l'accès à `coolify.ims-world.fr` en cas de perte de liaison IHM post-update. Voir [VM IMS-Coolify](/infrastructure/vm-coolify#procédure-post-mise-à-jour-coolify-perte-ihm).
- **Enrichissement de l'ADR-001 (Abandon de Beszel & Dualité LGTM/Dozzle)** — Révision de l'ADR-001 pour officialiser l'abandon définitif de Beszel au profit de la stack unifiée LGTM (Grafana/Loki/Alloy) pour la métrologie et l'alerting, tout en maintenant Dozzle (`logs.ims-world.fr`) pour le live-tailing léger des logs conteneurs en 1 clic. Voir [ADR-001 — Stack Monitoring LGTM & Maintien de Dozzle](/history/adr/adr-001-stack-monitoring-lgtm).

## 16/08/2026 — Audit Hardlinks HomeFlix, Nettoyage NAS (~330 Go) & ADR-007

### 🔧 Améliorations & Maintenance

- **Audit & Nettoyage des Orphelins `downloads/`** — Résolution d'un problème d'accumulation de fichiers orphelins dans `downloads/` causé par la suppression de contenus dans Radarr/Sonarr sans suppression du torrent qBittorrent associé. Un script de scan par comparaison d'inodes réels a permis de libérer **~330 Go d'espace disque** (disponibilité portée de 791 Go à 1.1 To). Voir [Détection & Nettoyage des Orphelins](/services/homeflix#detection--nettoyage-des-orphelins-downloads-audit--script-inodes).
- **Formalisation de l'ADR-007 (MergerFS `inodecalc=path-hash`)** — Documentation de la règle d'or pour le diagnostic d'inodes et de hardlinks : les calculs d'inodes virtuels par FUSE/MergerFS faussant les résultats via `/mnt/storage` ou NFS, tout diagnostic de hardlink doit s'effectuer en SSH direct sur le LXC NAS 100 sur le point de montage du disque physique (`/mnt/disk1/`). Voir [ADR-007 — Calcul d'Inodes MergerFS](/history/adr/adr-007-calcul-inodes-mergerfs-path-hash).
- **Procédure de résolution des imports manuels bloqués** — Documentation de la procédure d'import manuel pour les fichiers au nom générique bloqués avec l'erreur `Unable to parse file` dans Radarr/Sonarr. Voir [Résolution des Imports Manuels Bloqués](/services/homeflix#resolution-des-imports-manuels-bloques-unable-to-parse-file).

## Semaine du 14/08/2026 — Forgejo, Patrimo, Zipline & Coolify v4.3.2

### 🆕 Nouveautés

- **Forgejo en production** — Une forge Git self-hosted est en ligne sur `forge.ims-world.fr` pour héberger dépôts, issues et Pull Requests, avec miroir de sauvegarde automatique de dépôts GitHub (`sentryx`, `FailyBanDiscordBot`, `my_printf`, `MonitoringServer`, `Intra-IMS`, `default-ansible`). GitHub reste la plateforme principale de développement, Forgejo joue le rôle de miroir de sécurité secondaire. Voir [Forgejo](/services/forgejo).
- **Git SSH accessible depuis n'importe où** — Le trafic Git SSH sort par un port dédié `2222` exposé en NAT direct sur la Bbox (`git clone ssh://git@forge.ims-world.fr:2222/...`), sans VPN obligatoire. Le port SSH système du homelab reste isolé sur `4242`. Voir [ADR-006 — Exposition du port SSH Forgejo](/history/adr/adr-006-exposition-port-ssh-forgejo-bbox).
- **SSO Authentik OIDC natif sur Forgejo** — La connexion à `forge.ims-world.fr` passe directement par le SSO central `auth.ims-world.fr` en OIDC natif. Les inscriptions publiques directes sont fermées (`DISABLE_REGISTRATION=true`) : l'accès passe par Authentik ou par un compte local provisionné. Voir [Authentification & SSO OIDC](/services/forgejo#1-authentification--sso-oidc).
- **Patrimo en production (Application Coolify Git Build)** — Projet personnel Node.js déployé sur `patrimo.ims-world.fr`. Contrairement aux *Services Coolify* statiques, Patrimo est configuré en **Application Coolify** avec déploiement continu automatisé à chaque push GitHub via la GitHub App Coolify. La base de données PostgreSQL utilise un volume nommé Docker interne (`/var/lib/docker/volumes/`), exclu du périmètre de sauvegarde PBS en raison de son statut non-critique actuel. Voir [Patrimo](/services/patrimo).
- **Zipline en production** — Une plateforme de partage de fichiers, d'hébergement de captures d'écran (compatible ShareX) et de raccourcissement de liens est en ligne sur `share.ims-world.fr`. Les téléversements automatisés depuis un poste de travail passent par l'API Zipline avec jeton d'accès. Voir [Zipline](/services/zipline).
- **SSO Authentik OIDC natif sur Zipline** — La connexion à `share.ims-world.fr` passe directement par le SSO central `auth.ims-world.fr` via OIDC natif, sans middleware Forward-Auth. Voir [Sécurité & Authentification](/services/zipline#sécurité--authentification).

### 🔧 Améliorations

- **Mise à niveau de Coolify en v4.3.2** — L'orchestrateur principal hébergé sur la VM 104 a été mis à niveau en version **v4.3.2**, apportant des améliorations de stabilité sur la gestion des Webhooks Git et des builds Compose. Voir [VM IMS-Coolify](/infrastructure/vm-coolify).
- **VM Coolify en CPU mode `host` (x86-64-v2)** — Le CPU physique Intel i5-12600H de l'hôte MS-01 est désormais exposé intégralement à la VM Coolify (VMID 104) à la place du profil générique `kvm64`. Les binaires natifs Node.js qui exigent les instructions `x86-64-v2` (comme `sharp` pour l'optimisation d'images) démarrent sans contournement, et les performances vectorielles sont au maximum. Voir [ADR-005 — CPU VM Coolify en mode host](/history/adr/adr-005-cpu-vm-coolify-mode-host).
- **Matrice des rôles & droits d'accès (RBAC Authentik)** — Une matrice croisée documente désormais le périmètre d'accès applicatif des 5 groupes Authentik (`authentik Admins`, `admins`, `membres`, `invites`, `authentik Read-only`) pour chaque service exposé du homelab. Concrètement, on sait au premier coup d'œil qui peut atteindre quoi. Voir [Matrice des Rôles & Droits d'Accès (RBAC Authentik)](/reseau/matrice-securite-exposition#matrice-des-rles--droits-daccs-rbac-authentik).
- **Relais mail Forgejo via Resend** — Les notifications transactionnelles de Forgejo (invitations, alertes de dépôt) partent via le relais SMTP Resend (`forgejo@ims-world.fr`), aligné sur le reste des services du homelab. Voir [Forgejo — Fiche Service](/services/forgejo#fiche-service).

## Semaine du 13/08/2026 — Stirling PDF, boîte à outils PDF self-hosted

### 🆕 Nouveautés

- **Stirling PDF en production** — Une boîte à outils PDF complète est en ligne sur `pdf.ims-world.fr` pour la fusion, la découpe, la conversion et l'édition de documents PDF, avec OCR intégré via Tesseract v5. Le service est *stateless* : aucun document utilisateur n'est conservé après traitement. Voir [Stirling PDF](/services/stirling-pdf).
- **SSO Authentik sur Stirling PDF** — L'accès à `pdf.ims-world.fr` passe par le SSO central `auth.ims-world.fr`. L'authentification native de Stirling PDF est désactivée (`DOCKER_ENABLE_SECURITY=false`) pour éviter une double mire de connexion. Voir [Sécurité & Authentification](/services/stirling-pdf#sécurité--authentification).

### 🔧 Améliorations

- **Forward-Auth Authentik étendu à Stirling PDF** — L'Outpost Proxy Authentik (`ak-outpost-ims-outpost:9000`) s'intercale via le middleware Traefik en amont de Stirling PDF, qui n'a pas d'authentification native activée. Toute requête vers `pdf.ims-world.fr` exige une session Authentik valide avant d'atteindre l'interface. Voir [Outpost Proxy & Forward-Auth Traefik](/services/authentik#outpost-proxy--forward-auth-traefik).

## Semaine du 11/08/2026 — Statuspage Uptime Kuma & boîte à outils IT-Tools

### 🆕 Nouveautés

- **Uptime Kuma en production** — Une statuspage et un moteur de monitoring actif sont en ligne sur `status.ims-world.fr`. Uptime Kuma surveille en HTTP et en TCP la disponibilité des services publics et internes du homelab, avec accès protégé par SSO Authentik. Voir [Uptime Kuma](/services/uptime-kuma).
- **Alerting Uptime Kuma → Ntfy** — Les incidents détectés par Uptime Kuma déclenchent des notifications push sur mobile via Ntfy, avec des templates distincts pour les états *down* et *recovered*. Concrètement, une panne de service remonte sur le téléphone en quelques secondes. Voir [Alerting & Templates Ntfy](/services/uptime-kuma#alerting--templates-ntfy-liquidjs).
- **IT-Tools en production** — Une boîte à outils développeur et IT (générateurs, convertisseurs, utilitaires réseau) est disponible sur `tools.ims-world.fr`, protégée par le SSO Authentik. Voir [IT-Tools](/services/it-tools).

### 🔧 Améliorations

- **Forward-Auth Authentik étendu à Uptime Kuma & IT-Tools** — L'Outpost Proxy d'Authentik (`ak-outpost-ims-outpost:9000`) s'intercale via le middleware Traefik en amont des deux nouvelles interfaces, qui n'ont pas d'authentification native. Toute requête vers `status.ims-world.fr` ou `tools.ims-world.fr` exige une session Authentik valide avant d'atteindre l'application. Voir [Outpost Proxy & Forward-Auth Traefik](/services/authentik#outpost-proxy--forward-auth-traefik).

## Semaine du 11/08/2026 — Immich, médiathèque photo & vidéo self-hosted

### 🆕 Nouveautés

- **Immich en production** — Une médiathèque photo et vidéo self-hosted est en ligne sur `photos.ims-world.fr`, avec sauvegarde automatique depuis les applications mobiles iOS et Android (v3.x). La bibliothèque démarre avec 61 880 assets déjà indexés. Voir [Immich](/services/immich).
- **Recherche par IA (CLIP) & reconnaissance faciale** — Le moteur Machine Learning d'Immich permet la recherche sémantique en langage naturel (« plage au coucher du soleil ») et le regroupement automatique des visages, sans dépendre d'un service cloud externe. Voir [Composants & Stockage](/services/immich#composants--stockage).
- **SSO Authentik sur Immich** — L'authentification passe par le SSO central `auth.ims-world.fr` (OIDC), en plus de l'authentification native Immich pour les comptes techniques. Voir [Fiche Service](/services/immich#fiche-service).

### 🔧 Améliorations

- **Matrice de sécurité mise à jour** — Immich rejoint la Zone 1 (Public WAN) de la matrice d'exposition, aux côtés d'Authentik, Vaultwarden et HomeFlix, avec accès HTTPS protégé par Traefik et Let's Encrypt. Voir [Matrice de Sécurité & d'Exposition](/reseau/matrice-securite-exposition).
- **Cartographie Coolify mise à jour** — La liste des services orchestrés sur la VM Coolify inclut désormais Immich et son UUID de déploiement. Voir [VM IMS-Coolify](/infrastructure/vm-coolify).

## Semaine du 10/08/2026 — Notifications push, logs live & Forward-Auth Authentik

### 🆕 Nouveautés

- **Ntfy en production** — Un serveur de notifications push est en ligne sur `ntfy.ims-world.fr`, accessible depuis le web et les applications mobiles Android/iOS via le topic `ims-alerts`. Il sert de point de sortie unique pour les alertes du homelab. Voir [Ntfy — Notifications Push](/services/ntfy).
- **Dozzle en production** — Une interface de logs Docker en direct est disponible sur `logs.ims-world.fr` pour visualiser en temps réel les logs de tous les conteneurs de la VM Coolify. L'accès est protégé par Authentik (Forward-Auth Outpost). Voir [Dozzle — Logs Docker en Direct](/services/dozzle).
- **Alerting Grafana → Ntfy** — Grafana pousse désormais ses alertes vers Ntfy via un contact point Webhook, avec des payloads distincts pour les états *firing* et *resolved*. Les notifications arrivent sur mobile en priorité 4 (alerte) ou 3 (résolution). Voir [Alerting & Contact Point Ntfy](/services/monitoring#alerting--contact-point-ntfy).

### 🔧 Améliorations

- **Forward-Auth Authentik pour les apps sans SSO** — L'Outpost Proxy embarqué d'Authentik (`ak-outpost-ims-outpost:9000`) s'intercale via le middleware Traefik `forwardAuth` en amont des applications qui n'ont pas d'authentification native. Dozzle est le premier consommateur : toute requête vers `logs.ims-world.fr` exige une session Authentik valide avant d'atteindre l'interface. Le filtrage des accès se pilote depuis la console Authentik (*Applications → Outposts / Policies*). Voir [Outpost Proxy & Forward-Auth Traefik](/services/authentik#outpost-proxy--forward-auth-traefik).

## Semaine du 10/08/2026 — Stack Monitoring LGTM en production

### 🆕 Nouveautés

- **Stack Monitoring LGTM disponible** — Grafana, Loki et Prometheus sont en ligne sur `monitoring.ims-world.fr` pour la métrologie et la centralisation des logs de tout le homelab. L'interface est accessible depuis le tailnet, via connexion SSO Authentik. Voir [Stack Monitoring (LGTM)](/services/monitoring).
- **Collecte unifiée via Grafana Alloy** — Un agent Grafana Alloy tourne sur chaque hôte surveillé et pousse métriques et logs vers la stack en `remote-write`, sans configuration à maintenir côté serveur. Voir [Composants & Fonctionnement](/services/monitoring#composants--fonctionnement).
- **SSO Authentik OIDC sur Grafana** — La connexion à Grafana passe par le bouton *Sign in with authentik*. Trois rôles RBAC sont provisionnés côté Authentik : `Grafana Admins` (Admin), `Grafana Editors` (Éditeur) et `Grafana Viewers` (Lecteur). L'inscription libre est désactivée (`GF_USERS_ALLOW_SIGN_UP=false`) et un compte admin local reste disponible en secours. Voir [Intégration SSO Authentik OIDC](/services/monitoring#2-intégration-sso-authentik-oidc).

### 🔧 Améliorations

- **Isolation réseau du monitoring** — `monitoring.ims-world.fr` est masqué en DNS OVH (résolution `127.0.0.1` côté public) et filtré par Traefik au tailnet `100.64.0.0/10`. Concrètement, l'interface n'est joignable que via le VPN. Voir [Masquage DNS & Isolation Réseau](/services/monitoring#1-masquage-dns--isolation-réseau).
- **Matrice de sécurité étendue au monitoring** — Grafana rejoint la Zone 2 (Tailnet Overlay) de la matrice d'exposition, aux côtés de qBittorrent, Radarr, Sonarr, Prowlarr et Headplane. La cartographie des 3 zones de confiance est à jour. Voir [Matrice de Sécurité & d'Exposition](/reseau/matrice-securite-exposition).
- **Règle d'or de routage Traefik/Coolify formalisée** — Une nouvelle section documente quand utiliser le champ *Domains* de l'UI Coolify et quand ré-expliciter les labels Traefik manuellement dans le `docker-compose.yml`. Dès qu'un service utilise un middleware sur-mesure (`vpn-only`, forward-auth), il faut laisser le champ Domains vide côté Coolify pour éviter qu'un routeur automatique parallèle ne ré-expose le service publiquement. Voir [Règle d'Or de Routage](/reseau/traefik-proxy#️-règle-dor-de-routage--ui-coolify-vs-labels-compose-manuels).

## Semaine du 06/08/2026 — Audit sécurité `vpn-only` validé & RBAC Authentik

### 🆕 Nouveautés

- **Groupes & rôles RBAC sur Authentik** — Trois groupes officiels encadrent désormais l'accès aux applications : `admins` (superuser système), `membres` (famille & amis, accès complet à Vaultwarden, HomeFlix, etc.) et `invites` (accès restreint aux outils de divertissement). Chaque nouveau compte est automatiquement rattaché au groupe `membres` à la création. Voir [Authentik — Groupes & Rôles RBAC](/services/authentik#-groupes--rôles-rbac).
- **Procédure d'invitation utilisateur pas-à-pas** — Un parcours documenté permet de générer un lien d'invitation unique depuis Authentik, de l'envoyer par email via Resend (`no-reply@ims-world.fr`) et de laisser l'invité créer son compte lui-même, avec affectation automatique des droits. Voir [Authentik — Inviter un nouvel utilisateur](/services/authentik#-procédure--inviter-un-nouvel-utilisateur).

### 🔧 Améliorations

- **Durcissement Vaultwarden** — Les inscriptions de nouveaux comptes sont désormais fermées sur `vault.ims-world.fr` (`SIGNUPS_ALLOWED=false`) une fois le coffre-fort initialisé, ce qui bloque toute création de compte non autorisée. Le panneau d'administration `/admin` est en plus restreint au tailnet (`100.64.0.0/10`) via le middleware Traefik `vpn-only` : concrètement, il n'est plus joignable depuis l'Internet public, uniquement via le VPN. Voir [Vaultwarden — Politique de Sécurité & Durcissement](/services/vaultwarden#🛡️-politique-de-sécurité--durcissement-hardening).
- **Étanchéité `vpn-only` confirmée sur la stack HomeFlix** — L'audit du middleware `vpn-only` est terminé : qBittorrent, Radarr, Sonarr et Prowlarr sont accessibles uniquement depuis le tailnet (`100.64.0.0/10`) et renvoient `403 Forbidden` depuis le WAN public. Concrètement, ces interfaces d'administration restent joignables via le VPN sans risque d'exposition publique. Voir [Traefik Proxy](/reseau/traefik-proxy) et [Matrice de Sécurité](/reseau/matrice-securite-exposition).
- **Double verrouillage confirmé sur les services privés** — La protection combine un masquage DNS OVH (les sous-domaines privés pointent vers `127.0.0.1`, donc l'IP publique Bbox n'est pas révélée par une résolution DNS) et un filtrage applicatif Traefik (`ipAllowList` restreint à `100.64.0.0/10`). Une requête WAN qui contournerait le DNS est tout de même rejetée en `403 Forbidden` par le proxy. Voir [Traefik Proxy](/reseau/traefik-proxy#double-verrouillage--limites-de-sécurité-réelle).
- **Roadmap mise à jour** — L'étape « Audit de Sécurité Middleware `vpn-only` » est marquée comme effectuée. Une piste de durcissement Layer 4 (bind Tailscale ou règles pare-feu Proxmox) est identifiée pour rendre les services privés totalement injoignables depuis le WAN, même au niveau TCP. Voir [Roadmap](/procedures/roadmap).

Le reste de l'infrastructure et des services publics (Authentik, Vaultwarden, HomeFlix, Headscale/Headplane) tourne en état stable depuis les livraisons de la semaine précédente.

## Semaine du 05/08/2026 — Transcodage matériel HomeFlix

### 🆕 Nouveautés

- **Transcodage matériel Jellyfin (QuickSync)** — L'iGPU Intel Iris Xe du MS-01 est désormais attribuée en passthrough PCIe à la VM Coolify. HomeFlix (Jellyfin) accélère le transcodage H.264, HEVC et AV1 en matériel, ce qui réduit fortement la charge CPU pendant la lecture et permet plus de flux simultanés. Voir [Host Proxmox](/infrastructure/proxmox-host#gpu-igpu-iris-xe-passthrough-vm-coolify) et [HomeFlix](/services/homeflix).

### 🔧 Améliorations

- **Roadmap mise à jour** — L'étape « Passthrough GPU » est marquée comme effectuée sur la roadmap du projet. Voir [Roadmap](/procedures/roadmap).

## Semaine du 04/08/2026 — Résilience matérielle & monitoring hors-bande

### 🆕 Nouveautés

- **Afficheur Kiosk Raspberry Pi 3B+** — Un Raspberry Pi 3B+ dédié pilote l'affichage physique du rack Labrax via un écran principal Wisecoco 7.84" (1280x400) et un second écran OLED 0.91" (128x32) encastrés dans un module 2U 3D. Un navigateur headless fait tourner en boucle la bannière IMS et des pages web en mode lisibilité, contrôlé par un bouton poussoir GPIO (appui court = change source, long 3s = éteint). Voir [Raspberry Pi 3B+ & Écrans](/infrastructure/rpi-monitor).
- **Désinstallation définitive de Cap** — Abandon de l'expérimentation du service d'enregistrement d'écran Cap, retiré du homelab.

### 🔧 Améliorations

- **Module 3D imprimable pour le Raspberry Pi** — La fiche du Raspberry Pi 3B+ référence le modèle MakerWorld « Screen module for 10-inch rack — Raspberry Pi 2U », avec ses rendus CAD 3D (vue de façade et vue isométrique intérieure). Voir [Raspberry Pi 3B+ & Écrans](/infrastructure/rpi-monitor).
- **Tarpit SSH sur le Mac Mini** — Le port 22 du Mac Mini pointe désormais vers Endlessh (tarpit anti-bot) pour piéger les scans automatisés. L'accès SSH légitime passe par le port `4242`.
- **Résolution DNS de l'ancien Coolify** — L'ancienne instance Coolify du Mac Mini reste joignable via `coolify-old.ims-world.fr` grâce à un enregistrement DNS dans Headscale, le temps de la phase de validation post-cutover.
- **Matrice de sécurité interactive** — La cartographie des zones de confiance (WAN public, LAN, tailnet, admin) est désormais présentée sous forme d'onglets navigables avec cartes cliquables vers chaque service et sa politique d'exposition. Voir [Matrice de Sécurité & d'Exposition](/reseau/matrice-securite-exposition).
- **Arbre de décision de dépannage** — La page de dépannage inclut désormais un diagramme interactif qui guide symptôme par symptôme vers la cause probable et sa solution. Voir [Dépannage courant](/procedures/depannage-courant).

## Semaine du 03/08/2026 — Stabilisation post-cutover

### 🆕 Nouveautés

- **VPN self-hosted disponible** — L'accès distant au tailnet passe désormais par notre propre control plane, avec une interface d'administration web pour gérer utilisateurs, clés et appareils. Voir [Headscale + Headplane](/services/headscale-headplane).
- **Portail de demandes HomeFlix** — Jellyseerr est en ligne sur `videoclub.ims-world.fr` pour demander films et séries directement, en complément du streaming sur `homeflix.ims-world.fr`. Voir [HomeFlix](/services/homeflix).

### 🔧 Améliorations

- **Traefik mis à jour en v3.7** — Le reverse proxy public a été passé de la v3.6.23 à la v3.7 pendant le cutover, sans interruption perceptible côté utilisateur.
- **Certificats renouvelés automatiquement** — Le renouvellement des certificats HTTPS est de nouveau fonctionnel sur tous les domaines publics après la mise en place du challenge DNS-01. Plus d'avertissements de certificats expirés.
- **SSO Authentik consolidé** — L'identité centrale (`auth.ims-world.fr`) est désormais servie depuis la nouvelle infrastructure, avec le branding par domaine préservé pour chaque service.

### 🐛 Corrections

- **HomeFlix : téléchargements qBittorrent** — Les erreurs d'accès à l'interface qBittorrent derrière le proxy sont corrigées ; le WebUI est de nouveau accessible normalement.
- **Headscale : reconnexion des appareils** — La boucle de reconnexion OIDC constatée en début de bascule est résolue ; tous les appareils du tailnet se reconnectent sans intervention.
## 🎉 02/08/2026 — Cutover complet

Les 4 services essentiels (Authentik, Vaultwarden, HomeFlix, Headscale/Headplane) sont basculés en production sur le MS-01. Port-forward Bbox corrigé et basculé. Découverte structurelle majeure : le port-forward route tout le trafic public d'un coup, pas de bascule partielle possible (voir [Traefik Proxy](/reseau/traefik-proxy)).

Cascade de 8 blocages résolus dans la même session : accès console (Chrome), SSH manquant, label réseau Traefik manquant, port-forward mal ciblé, cache DNS transitoire, crash-loop OIDC Headscale, certificats DNS-01 en cours de négociation, warning cosmétique Coolify.

Mise à jour Traefik v3.6.23 → v3.7 effectuée en avance sur le planning, sans incident réel.

## 01/08/2026 — Préparation Headscale (Phases A-C)

Migration de données complète (config, `noise_private.key`, base SQLite) — tous les utilisateurs et appareils confirmés présents. Décision actée de ne pas tester via sous-domaine `-ng` (le `server_url` de Headscale est une identité, pas un routage). Piège du dossier fantôme rencontré une 3ème fois, avec complication inédite (nécessité de recréer le container).

Expérimentation du service Cap testée puis abandonnée définitivement (service retiré du homelab).

## 31/07/2026 — DNS-01 et mise à jour Traefik

Configuration du challenge DNS-01 (OVH) sur le proxy MS-01 — 3 blocages résolus (email ACME manquant, permissions `acme.json`, panne du résolveur DNS OVH contournée via `8.8.8.8`). Découverte que la vraie prod (Mac Mini) souffrait du même problème de renouvellement de certificat depuis 5 jours, corrigé par la même occasion.

## 30/07/2026 — HomeFlix migré et validé

Migration la plus critique du projet : 1.6 To, 426 hardlinks préservés, 9 services. Vérification pré-migration avec `rdfind`. Restructuration stockage (config/cache sur SSD, médias sur NAS). Grand troubleshooting qBittorrent (3 fausses pistes avant la cause réelle : `HostHeaderValidation`). Piège `du` vs `df` découvert et documenté.

## 28/07/2026 — Vaultwarden migré et validé

Découverte du piège `config.json` à domaine figé. Technique d'accès facilité via ACL POSIX mise en place pour la suite du projet.

## 23/07/2026 — Authentik migré et validé

Premier service stateful migré. Protocole de migration affiné à cette occasion (dump/restore Postgres, découverte du branding par domaine et de l'emplacement réel des médias `/data/media`).

## Mi-juillet 2026 — Infrastructure de base

Déploiement complet du socle Proxmox : LXC NAS (MergerFS + NFS + SMB), LXC PBS (Proxmox Backup Server, datastore via NFS), VM Coolify (Docker + Coolify 4.1.2). Autostart et ordre de boot validés par reboot réel. Décision stratégique majeure : migrer seulement 4 services essentiels avant cutover anticipé, plutôt que la stack complète (~15 services) — réduit la fenêtre de risque et la contrainte disque mono-HDD.

<Info>
Journaux de session complets et détaillés disponibles séparément pour l'historique exhaustif (diagnostics pas-à-pas, commandes exactes, fausses pistes explorées). Cette page résume les décisions et jalons, pas le détail opérationnel.
</Info>
