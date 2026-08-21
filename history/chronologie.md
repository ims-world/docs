---
title: "Changelog & Historique"
description: "Chronologie du projet et journal exhaustif des livraisons de l'infrastructure Homelab"
---

<Update label="Semaine du 13/08/2026 au 19/08/2026" description="Récapitulatif hebdomadaire : PhotoPrism, Forgejo, tier SSD & fiabilité GPU">
  ### ✨ Nouveaux services
  - **[PhotoPrism](/services/photoprism)** est disponible sur `studio.ims-world.fr`. Votre bibliothèque photo et studio d'archivage RAW sont hébergés en interne, avec restauration confirmée de 24 565 photos et dépôt WebDAV pour l'ingestion.
  - **[Forgejo](/services/forgejo)** est en production sur `forge.ims-world.fr`. Vous pouvez héberger vos dépôts Git, issues et Pull Requests avec SSO Authentik, plus un miroir de sauvegarde automatique depuis GitHub. Le clonage SSH est accessible depuis n'importe quel réseau via `ssh://git@forge.ims-world.fr:2222/…`.
  - **[Patrimo](/services/patrimo)** est en ligne sur `patrimo.ims-world.fr` avec déploiement continu à chaque push.
  - **[Zipline](/services/zipline)** est disponible sur `share.ims-world.fr` pour le partage de fichiers, les captures ShareX et le raccourcissement de liens, avec connexion SSO.
  - **[Stirling PDF](/services/stirling-pdf)** est publié sur `pdf.ims-world.fr`. Vous disposez d'une boîte à outils PDF (fusion, découpe, conversion, OCR) en mode *stateless* : aucun document n'est conservé après traitement. L'accès est protégé par SSO Authentik.

  ### 🔧 Mises à jour
  - **Coolify** est passé en v4.3.2 puis v4.3.6 : gestion améliorée des webhooks Git et des builds Compose. Une [procédure de secours](/infrastructure/vm-coolify#procédure-post-mise-à-jour-coolify-perte-ihm) est documentée si l'accès à `coolify.ims-world.fr` est perdu après une mise à jour.
  - **Migration terminée vers le MS-01** : l'ensemble des services applicatifs tourne désormais sur le nouveau serveur. Le [Mac Mini 2014](/infrastructure/mac-mini) bascule officiellement en Standby Chaud de secours.
  - **Tier de stockage `storage-hot` sur SSD dédié** : les données chaudes d'Immich, Forgejo et consorts sont déplacées sur un SSD 4 To dédié. Vous gagnez 3 To d'espace libre et de meilleures performances. Voir [Ajout d'un nouveau disque](/procedures/ajout-nouveau-disque).
  - **Nettoyage HomeFlix** : environ 330 Go d'espace libéré sur le NAS après audit des fichiers orphelins. La disponibilité passe de 791 Go à 1,1 To. Voir [HomeFlix](/services/homeflix).
  - **Accélération matérielle Jellyfin** : le passthrough complet de l'iGPU vers la VM Coolify est officialisé. Les transcodes atteignent 29,7× le temps réel. Voir [ADR-008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe).

  ### 🐛 Corrections
  - **Jellyfin & Sonarr rétablis après incident GPU** : la perte du périphérique `/dev/dri` au redémarrage a été corrigée et un correctif préventif est en place pour éviter toute récidive lors des mises à jour du noyau. Voir le [Post-Mortem du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri).
  - **Grafana accessible en SSO** : suite à un blocage HTTP 403 identifié sur le middleware `vpn-only`, Grafana bascule sur la connexion SSO Authentik OIDC. Voir [ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only).
  - **Imports Radarr / Sonarr débloqués** : la procédure de résolution des fichiers en erreur `Unable to parse file` est documentée sur [HomeFlix](/services/homeflix#resolution-des-imports-manuels-bloques-unable-to-parse-file).
</Update>

<Update label="21/08/2026" description="Déploiement Home Assistant (Smart Home), Monitoring Avancé & Dashboards">
  ### 🏠 Domotique & Smart Home (Home Assistant)
  - **Déploiement de Home Assistant 2025.10.2** — Déploiement en conteneur Docker pur sur la VM 104 (`ims-coolify`) avec publication publique sur `home.ims-world.fr`. Authentification native HA + 2FA TOTP pour préserver la compatibilité OAuth2 avec l'application mobile *Companion App*. Voir [Home Assistant](/services/home-assistant).

  ### 📊 Métrologie & Supervision Avancée (Stack LGTM)
  - **Monitoring SMART Bare-Metal (`ms01-pve`)** — Déploiement du *textfile collector* Node Exporter alimenté par `smartmon.sh` (cron 5 min) pour remonter la santé SMART, températures et heures de vol des disques physiques (`nvme0n1`, `sda` SSD, `sdb` HDD) avec respect strict du spin-down (`smartctl -n standby`).
  - **Scrape Uptime Kuma (Pull) & Métriques SSL** — Ingestion des métriques d'Uptime Kuma (`/metrics` Basic Auth) avec calcul des SLO 24h/7j/30j et suivi d'expiration des certificats SSL/TLS.
  - **Rétention 1 An Prometheus** — Passage de la rétention TSDB de 30 jours à **1 an** (`--storage.tsdb.retention.time=1y`) pour un coût de stockage estimé à ~5-6 Go/an.
  - **Nouveaux Dashboards Grafana Exécutifs** — Publication de 4 nouveaux dashboards de production : *Vue d'ensemble — IMS-WORLD*, *Uptime Kuma - Overview*, *Gestion des disques*, et *Traefik — Reverse Proxy*. Correction des labels sur le template *Node Exporter Full* (ID `1860`). Voir [Stack Monitoring](/services/monitoring).
</Update>

<Update label="20/08/2026" description="Mises à Jour Applicatives & Avancement Châssis Labrax">
  ### 📦 Mises à Jour Applicatives
  - **Coolify v4.3.9** — Mise à niveau du moteur d'orchestration applicatif sur la VM 104.
  - **Jellyseerr v3.4.1** — Mise à jour du portail de requêtes média de la stack HomeFlix (`videoclub.ims-world.fr`).

  ### 🖨️ Hardware & Châssis 3D Labrax
  - **Impression 3D Châssis Terminée** — L'impression 3D de l'intégralité des pièces du châssis rackable 10" Labrax s'est achevée avec succès cette nuit. L'étape suivante concerne le montage physique et l'intégration des composants.
</Update>

<Update label="19/08/2026 - 20/08/2026" description="Isolation Réseau vpn-only, CIS Benchmark & Bypass SNAT Tailscale">
  ### 🛡️ Refonte Réseau & Résolution Double Incident vpn-only (ADR-009)
  - **Durcissement Démon Docker (`"userland-proxy": false`)** — Application de la recommandation officielle du **CIS Docker Benchmark** sur la VM Coolify. Basculement natif du noyau Linux (iptables `DNAT` + `MASQUERADE` + `net.ipv4.route_localnet`), éliminant la substitution d'IP par le binaire `docker-proxy`.
  - **Bypass du SNAT Tailscale (`--snat-subnet-routes=false`)** — Résolution du 2nd bug apparu post-reboot MS-01 en désactivant le masquage automatique `ts-postrouting` de Tailscale sur le trafic forwardé vers le bridge Docker (`coolify`).
  - **Provider File Centralisé Traefik (`vpn-only.yaml`)** — Isolation réseau étanche et centralisée des 7 services d'administration (`coolify`, `headplane`, `qbit`, `sonarr`, `radarr`, `prowlarr`, `monitoring`) dans `/data/coolify/proxy/dynamic/vpn-only.yaml`. Retrait des domaines dans l'UI Coolify pour éliminer les concurrences de routeurs.
  - **Découplage Headscale / Headplane** — Séparation de `vpn.ims-world.fr` (public WAN, coordination Tailscale) et `https://admin.vpn.ims-world.fr/admin` (privé Tailnet avec suffixe `/admin` obligatoire et résolution split-horizon `100.64.0.4`). Voir l'[ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only) et le [Post-Mortem du 18-20/08/2026](/history/incidents/2026-08-18-blocage-traefik-vpn-only-docker-proxy).
</Update>

<Update label="19/08/2026" description="Incident Stale NFS Filehandle Jellyfin & Optimization Backup LXC 100">
  ### 🚨 Incident Rétabli & Arbitrage Sauvegardes
  - **Échecs de lecture Jellyfin post-vzdump** — Résolution des échecs de lecture vidéo causés par l'invalidation irréversible des descripteurs NFS (`Stale filehandle`) lors du redémarrage quotidien de MergerFS FUSE par le job de sauvegarde `vzdump --mode stop` de la LXC 100 (`ims-nas`).
  - **Désactivation du Backup Automatique LXC 100** — Suppression du job automatique quotidien. La sauvegarde du rootfs (8 Go statique) est désormais uniquement manuelle avant maintenance.
  - **Correction d'urgence `backup=0` sur `mp1`** — Ajout de la directive `backup=0` sur le point de montage `mp1` (SSD 4 To) pour éliminer tout risque de saturation du stockage local de Proxmox (`local-lvm`). Voir le [Post-Mortem du 19/08/2026](/history/incidents/2026-08-19-stale-nfs-filehandle-jellyfin-mergerfs).
</Update>

<Update label="19/08/2026" description="Incident GPU Passthrough (/dev/dri) & Métapaquet Kernels">
  ### 🚨 Incident Rétabli
  - **Disparition de `/dev/dri` au redémarrage complet** — Résolution d'un dysfonctionnement au boot de la VM Coolify provoquant l'échec de Jellyfin et Sonarr avec l'erreur `error gathering device information while adding custom device "/dev/dri": no such file or directory`.
  - **Cause racine identifiée** : La mise à jour automatique en arrière-plan (`unattended-upgrades`) vers le noyau Ubuntu `6.8.0-138-generic` n'avait pas tiré le paquet `linux-modules-extra` correspondant (qui contient le pilote `i915`).
  - **Rétablissement & Fix Préventif** : Rétablissement à chaud sans reboot via `apt install linux-modules-extra-$(uname -r)` + `modprobe i915`, puis installation préventive du métapaquet générique `linux-modules-extra-generic` pour automatiser les futurs redémarrages noyau. Voir le [Post-Mortem du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri).
</Update>

<Update label="18/08/2026" description="Tier Storage-Hot SSD 4To & ADR-009">
  ### ⚡ Stockage & Sécurité
  - **Bascule réussie du tier `storage-hot` sur SSD 4To dédié** — Migration de `storage-hot` (données applicatives chaudes d'Immich et Forgejo) depuis un bind-mount HDD vers un SSD Samsung 870 EVO 4To raccordé en SATA natif sur le contrôleur ASM1166 du MS-01.
  - **Libération de 934 Go sur le HDD principal** — La suppression de l'ancienne copie après validation a libéré 934 Go sur `/mnt/disk1`. Le tier `storage-hot` dispose désormais de 3.6 To utiles (443 Go utilisés, 3.0 To libres). Voir [Ajout d'un nouveau disque](/procedures/ajout-nouveau-disque).
  - **Procédure de bascule d'un tier NFS formalisée** — Documentation pas-à-pas de la séquence exacte (désexport NFS `exportfs -u`, remontage bind `mount --bind`, réexport `exportfs -r`, `systemctl daemon-reload` et `umount -f` côté client VM Coolify).
  - **Enseignements système & contrôleur ASM1166** — Confirmation que le contrôleur ASM1166 exige l'extinction du host avant toute connexion/déconnexion SATA (hotplug instable sous Linux).
  - **Publication de l'ADR-009 (Bug `docker-proxy` / `vpn-only`)** — Découverte de la substitution systématique de l'IP source d'origine par l'IP passerelle bridge Docker (`10.0.1.1`) provoquant un blocage HTTP 403 du middleware Traefik `vpn-only`. Grafana basculé sur le SSO Authentik OIDC et inscription d'une fenêtre de maintenance pour tester `"userland-proxy": false`. Voir [ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only).
</Update>

<Update label="17/08/2026" description="PhotoPrism, Décommissionnement Mac Mini & ADR-008">
  ### 📸 PhotoPrism & Décommissionnement
  - **PhotoPrism en production sur MS-01** — Migration réussie de la bibliothèque photo & studio d'archivage RAW sur `studio.ims-world.fr` (UUID Coolify `yfotvbtkqj8cqw5alox6gfpr`). Restauration confirmée de **24 565 photos** via dump/restore SQL MariaDB 11.8.8. Voir [PhotoPrism](/services/photoprism).
  - **Architecture de stockage hybride PhotoPrism** — Les originaux RAW (`originals/`, ~574 Go) et métadonnées YAML (`storage/sidecar/`, 42 Go) résident sur le pool capacitif NFS HDD (`storage`), tandis que MariaDB tourne sur un volume nommé Docker local (`photoprism-mariadb-data`) sur le SSD de la VM Coolify.
  - **Ingestion RAW via WebDAV** — Documentation de la procédure de dépôt via WebDAV (`/import/` et `/originals/`) et commande CLI `photoprism import`.
  - **Décommissionnement officiel du Mac Mini 2014** — L'ancien serveur principal Mac Mini 2014 a été officiellement déconnecté de la production suite à la migration réussie de l'ensemble des services applicatifs sur le MS-01. Il bascule en mode Standby Chaud de secours. Voir [Mac Mini 2014](/infrastructure/mac-mini).
  - **Mise à niveau de Coolify en v4.3.6 & Workaround IHM** — Montée en version vers v4.3.6. Documentation de la procédure de secours `docker restart coolify-proxy` requise pour rétablir l'accès à `coolify.ims-world.fr` en cas de perte de liaison IHM post-update. Voir [VM IMS-Coolify](/infrastructure/vm-coolify#procédure-post-mise-à-jour-coolify-perte-ihm).
  - **Enrichissement de l'ADR-001 (Abandon de Beszel & Dualité LGTM/Dozzle)** — Officialisation de l'abandon de Beszel au profit de la stack LGTM (Grafana/Loki/Alloy) et maintien de Dozzle (`logs.ims-world.fr`). Voir [ADR-001](/history/adr/adr-001-stack-monitoring-lgtm).
  - **Adoption de l'ADR-008 (GPU Passthrough)** — Formalisation du passthrough PCI complet de l'iGPU Iris Xe via VFIO/IOMMU vers la VM Coolify, passage au chipset `q35` et validation Jellyfin QSV à 29.7x le temps réel. Voir [ADR-008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe).
  - **Formalisation de la Politique de Sauvegarde** — Publication de la page consolidant la chronologie nocturne (02h-05h), les sauvegardes `vzdump` validées des LXC 100 & 103, les contournements FUSE/MergerFS (`--mode stop`) et les règles d'anti-circularité. Voir [Politique de Sauvegarde](/infrastructure/politique-sauvegardes).
</Update>

<Update label="16/08/2026" description="Audit Hardlinks HomeFlix, Nettoyage NAS (~330 Go) & ADR-007">
  ### 🧹 Stockage & Diagnostic
  - **Audit & Nettoyage des Orphelins `downloads/`** — Résolution d'un problème d'accumulation de fichiers orphelins dans `downloads/` causé par la suppression de contenus dans Radarr/Sonarr sans suppression du torrent qBittorrent associé. Un script de scan par comparaison d'inodes réels a permis de libérer **~330 Go d'espace disque** (disponibilité portée de 791 Go à 1.1 To). Voir [HomeFlix](/services/homeflix#detection--nettoyage-des-orphelins-downloads-audit--script-inodes).
  - **Formalisation de l'ADR-007 (MergerFS `inodecalc=path-hash`)** — Documentation de la règle d'or pour le diagnostic d'inodes et de hardlinks : les calculs d'inodes virtuels par FUSE/MergerFS faussant les résultats via `/mnt/storage` ou NFS, tout diagnostic de hardlink doit s'effectuer en SSH direct sur le LXC NAS 100 sur le point de montage physique (`/mnt/disk1/`). Voir [ADR-007](/history/adr/adr-007-calcul-inodes-mergerfs-path-hash).
  - **Procédure d'import manuel Radarr/Sonarr** — Documentation de la résolution des fichiers bloqués avec `Unable to parse file`. Voir [HomeFlix](/services/homeflix#resolution-des-imports-manuels-bloques-unable-to-parse-file).
</Update>

<Update label="14/08/2026" description="Forgejo, Patrimo, Zipline & Coolify v4.3.2">
  ### 🚀 Nouveaux Services & CPU Host Mode
  - **Forgejo en production** — Forge Git self-hosted sur `forge.ims-world.fr` pour héberger dépôts, issues et Pull Requests, avec miroir de sauvegarde automatique de dépôts GitHub (`sentryx`, `FailyBanDiscordBot`, `my_printf`, `MonitoringServer`, `Intra-IMS`, `default-ansible`). SSO Authentik OIDC natif. Voir [Forgejo](/services/forgejo).
  - **Git SSH accessible partout (ADR-006)** — Trafic Git SSH sortant par le port dédié `2222` en NAT Bbox direct (`git clone ssh://git@forge.ims-world.fr:2222/...`). Port SSH système préservé sur `4242`. Voir [ADR-006](/history/adr/adr-006-exposition-port-ssh-forgejo-bbox).
  - **Patrimo en production** — Application Node.js sur `patrimo.ims-world.fr` en mode *Application Coolify Git Build* avec déploiement continu à chaque push GitHub. Voir [Patrimo](/services/patrimo).
  - **Zipline en production** — Plateforme de partage de fichiers, captures d'écran (compatible ShareX) et raccourcissement de liens sur `share.ims-world.fr` avec SSO Authentik OIDC natif. Voir [Zipline](/services/zipline).
  - **VM Coolify en CPU mode `host` (ADR-005)** — Le CPU i5-12600H est exposé intégralement à la VM Coolify (`x86-64-v2`). Les binaires natifs exigeant `sharp` démarrant sans contournement. Voir [ADR-005](/history/adr/adr-005-cpu-vm-coolify-mode-host).
  - **Mise à niveau de Coolify en v4.3.2** — Amélioration de la gestion des Webhooks Git et des builds Compose.
  - **Relais mail Forgejo via Resend** — Notifications transactionnelles via `forgejo@ims-world.fr`.
</Update>

<Update label="13/08/2026" description="Stirling PDF en production">
  ### 📄 Outillage PDF Stateless
  - **Stirling PDF en production** — Boîte à outils PDF (fusion, découpe, conversion, OCR Tesseract v5) sur `pdf.ims-world.fr`, en mode *stateless* (aucun document conservé après traitement). Authentification native désactivée (`DOCKER_ENABLE_SECURITY=false`). Voir [Stirling PDF](/services/stirling-pdf).
  - **Forward-Auth Authentik étendu** — L'Outpost Proxy Authentik (`ak-outpost-ims-outpost:9000`) s'intercale via Traefik en amont de Stirling PDF. Voir [Authentik](/services/authentik#outpost-proxy--forward-auth-traefik).
</Update>

<Update label="11/08/2026" description="Statuspage Uptime Kuma, IT-Tools & Immich">
  ### 📊 Monitoring, Tools & Médiathèque Photo
  - **Uptime Kuma & Alerting Ntfy** — Statuspage et monitoring actif sur `status.ims-world.fr` avec notifications push mobiles instantanées via Ntfy (templates LiquidJS pour états *down* et *recovered*). Voir [Uptime Kuma](/services/uptime-kuma).
  - **IT-Tools en production** — Boîte à outils développeur (générateurs, convertisseurs, utilitaires réseau) sur `tools.ims-world.fr`. Voir [IT-Tools](/services/it-tools).
  - **Forward-Auth Authentik étendu** — Outpost Proxy Authentik configuré en amont de Uptime Kuma et IT-Tools.
  - **Immich en production** — Médiathèque photo & vidéo self-hosted sur `photos.ims-world.fr` avec sauvegarde automatique iOS/Android, recherche IA (CLIP) et reconnaissance faciale (61 880 assets indexés). Voir [Immich](/services/immich).
</Update>

<Update label="10/08/2026" description="Ntfy, Dozzle & Stack Monitoring LGTM">
  ### 🔔 Push Alerts, Logs Live & Stack LGTM
  - **Ntfy en production** — Serveur de notifications push sur `ntfy.ims-world.fr` (`ims-alerts`) servant de point de sortie unique pour toutes les alertes du homelab. Voir [Ntfy](/services/ntfy).
  - **Dozzle en production** — Viewer de logs Docker en direct sur `logs.ims-world.fr` protégé par Forward-Auth Authentik. Voir [Dozzle](/services/dozzle).
  - **Stack Monitoring LGTM** — Grafana, Loki et Prometheus sur `monitoring.ims-world.fr` avec agent Grafana Alloy en `remote-write` et SSO Authentik OIDC (`Grafana Admins`, `Editors`, `Viewers`). Voir [Monitoring](/services/monitoring).
  - **Alerting Grafana → Ntfy** — Webhook contact point poussant les alertes *firing* (priorité 4) et *resolved* (priorité 3) sur mobile.
  - **Règle d'or de routage Traefik/Coolify formalisée** — Documentation de l'obligation de laisser le champ *Domains* vide dans l'UI Coolify dès qu'un service utilise un middleware sur-mesure (`vpn-only`, forward-auth) pour éviter l'exposition publique par un routeur parallèle automatique. Voir [Traefik Proxy](/reseau/traefik-proxy#️-règle-dor-de-routage--ui-coolify-vs-labels-compose-manuels).
</Update>

<Update label="06/08/2026" description="Audit sécurité vpn-only & RBAC Authentik">
  ### 🔐 Sécurité & Droits d'Accès
  - **Groupes & Rôles RBAC Authentik** — Structuration des 3 groupes (`admins`, `membres`, `invites`) et procédure d'invitation par email Resend (`no-reply@ims-world.fr`). Voir [Authentik](/services/authentik#-groupes--rôles-rbac).
  - **Durcissement Vaultwarden** — Fermeture des inscriptions (`SIGNUPS_ALLOWED=false`) et restriction de l'URL `/admin` au Tailnet (`100.64.0.0/10`) via le middleware Traefik `vpn-only`. Voir [Vaultwarden](/services/vaultwarden#🛡️-politique-de-sécurité--durcissement-hardening).
  - **Étanchéité `vpn-only` sur Stack HomeFlix** — Confirmation du renvoi HTTP `403 Forbidden` depuis le WAN sur qBittorrent, Radarr, Sonarr et Prowlarr.
</Update>

<Update label="04/08/2026 - 05/08/2026" description="Passthrough GPU Jellyfin, RPi 3B+ & Tarpit SSH">
  ### 🎬 GPU Passthrough & Hardware Rack
  - **Transcodage matériel Jellyfin (QuickSync)** — Attribution PCIe de l'iGPU Intel Iris Xe à la VM Coolify. HomeFlix accélère le H.264/HEVC/AV1 avec charge CPU minimale. Voir [HomeFlix](/services/homeflix).
  - **Afficheur Kiosk Raspberry Pi 3B+** — Écran principal Wisecoco 7.84" (1280x400) et OLED 0.91" (128x32) dans un module 2U 3D pour le rack Labrax. Bouton poussoir GPIO (court = switch source, long 3s = extinction). Voir [Raspberry Pi 3B+](/infrastructure/rpi-monitor).
  - **Tarpit SSH Endlessh sur Mac Mini** — Port 22 routé vers Endlessh pour piéger les scans anti-bots (SSH légitime déplacé sur le port 4242).
  - **DNS de secours Mac Mini** — Enregistrement Headscale `coolify-old.ims-world.fr` conservé pendant la validation post-cutover.
  - **Matrice de sécurité interactive & Arbre de dépannage** — Publication des onglets interactifs de sécurité et du logigramme de dépannage.
</Update>

<Update label="02/08/2026" description="Cutover complet sur MS-01">
  ### 🎉 Cutover de Production
  - **Bascule en production des 4 services majeurs** (Authentik, Vaultwarden, HomeFlix, Headscale/Headplane) du Mac Mini vers l'hyperviseur MS-01.
  - **Port-forward Bbox** — Bascule du bloc de redirections et découverte de l'impossibilité de bascule partielle (tout le trafic public bascule d'un coup).
  - **Cascade de 8 blocages résolus** : Accès console Chrome, SSH manquant, label réseau Traefik manquant, port-forward mal ciblé, cache DNS transitoire, crash-loop OIDC Headscale, certificats DNS-01 Let's Encrypt, warning cosmétique Coolify.
  - **Mise à niveau Traefik** : Passage de Traefik v3.6.23 à v3.7 sans interruption.
</Update>

<Update label="15/07/2026 - 31/07/2026" description="Socle Proxmox, Challenge DNS-01 & Migrations Fichiers">
  ### 🏗️ Socle d'Infrastructure & Migrations de Masse
  - **Challenge DNS-01 OVH (31/07)** — Configuration du challenge DNS-01 (OVH) sur Traefik MS-01 pour les certificats HTTPS jokers. Résolution du blocage de renouvellement sur le Mac Mini.
  - **Migration HomeFlix 1.6 To (30/07)** — Transfert de 1.6 To avec préservation des 426 hardlinks (`rdfind`). Restructuration du stockage (config/cache sur SSD, médias sur NAS). Résolution du problème WebUI qBittorrent (`HostHeaderValidation`). Découverte et documentation du piège `du` vs `df`.
  - **Migration Vaultwarden (28/07)** — Découverte du piège `config.json` à domaine figé et mise en place de la gestion des droits par ACL POSIX.
  - **Migration Authentik (23/07)** — Premier service stateful migré (dump/restore Postgres, branding par domaine et médias `/data/media`).
  - **Déploiement du Socle Proxmox (Mi-juillet)** — Déploiement complet du socle Proxmox VE 9.2.3 : LXC 100 NAS (MergerFS + NFS + SMB), LXC 103 PBS (Proxmox Backup Server, datastore NFS), VM 104 Coolify (Docker + Coolify 4.1.2). Autostart et ordre de boot (NAS order=1 → PBS order=2 → Coolify order=3) validés par reboot physique.
</Update>
