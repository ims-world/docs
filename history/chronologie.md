---
title: "Changelog & Historique"
description: "Chronologie du projet et journal exhaustif des livraisons de l'infrastructure Homelab"
---

<Update label="22/09/2026" description="Déploiement Plateforme d'Automatisation n8n v2.10.4 (Queue Mode, Redis & Task Runners) sur Coolify">
  ### ⚡ Orchestration & Automatisation de Flux (n8n)

  - **Mise en Service de la Stack n8n sur Coolify** — Déploiement du template officiel Coolify haute performance pour **n8n v2.10.4** sur la VM 104 (`ims-coolify`, UUID `uifode0ypia57wbkyoertbxh`). Architecture en **Queue Mode** articulée autour de 5 conteneurs : `n8n` (serveur HTTP/webhooks), `n8n-worker` (travailleur asynchrone), `redis:6-alpine` (broker Bull Queue), `task-runners:2.10.4` (exécuteur Python natif sandboxed) et `postgres:16-alpine`.
  - **Exposition Publique Sécurisée (`automation.ims-world.fr`)** — Routage Traefik v3 avec terminaison TLS 1.3 Let's Encrypt (DNS-01 OVH) et protection bouncer CrowdSec. Résolution immédiate via le wildcard DNS `*.ims-world.fr`.
  - **Architecture d'Authentification & Webhooks Ouverts** — Choix de l'authentification native de n8n avec **2FA TOTP**, assurant la transparence totale et l'accès sans proxy pour les webhooks entrants (`/webhook/*`). Voir [n8n (Automation)](/services/n8n).
</Update>

<Update label="14/09/2026" description="Déploiement WhatsUpDocker (WUD), Vague de Mises à Jour Applicatives (8 services) & Template VM 8000 Tailscale">
  ### 🔔 Supervision des Mises à Jour (WhatsUpDocker)

  - **Déploiement de WhatsUpDocker (WUD)** — Mise en service du conteneur `getwud/wud:9.0.2` sur la VM 104 (UUID `qwe5jrqlqtwqneevgkf6mwr9`). Il analyse en continu les registres Docker pour détecter les nouvelles versions stables selon la norme SemVer.
  - **Alerting Push Immédiat via Ntfy** — Intégration directe du trigger Ntfy avec authentification par jeton (`WUD_TRIGGER_NTFY_DEFAULT_AUTH_TOKEN`) pour notifier les nouvelles versions sur le topic `ims-alerts`.
  - **Sécurisation Réseau & Faux Positif qBittorrent** — Isolation stricte du tableau de bord WUD en Zone 2 (`vpn-only.yaml`) sans exposition WAN. Diagnostic du faux positif SemVer sur qBittorrent (`20.04.1` provenant d'un ancien tag de build Ubuntu 2021) et recommandation de maintien en version `5.0.4`. Voir [WhatsUpDocker](/services/whatsupdocker).

  ### 📦 Vague de Mises à Jour Applicatives (8 Services)

  - **Traefik (Coolify Proxy) `v3.7.13`** — Mise à niveau du moteur de reverse proxy central sur la VM 104 avec maintien du bouncer CrowdSec. Voir [Traefik (Coolify Proxy)](/reseau/traefik-proxy).
  - **Gluetun `v3.41.3` (Stack HomeFlix)** — Montée de version de `qmcgaw/gluetun:v3.40.0` vers `v3.41.3` pour corriger le deadlock critique sur la réattribution dynamique de port ProtonVPN. Modernisation des variables DNS (`DNS_SERVER: 'on'`, `DNS_UPSTREAM_RESOLVERS: cloudflare`). Voir [HomeFlix](/services/homeflix).
  - **Headscale `v0.29.3` & Headplane `0.7.1`** — Montée de version et résolution des incompatibilités de configuration de Headscale v0.29 (suppression de la directive obsolète `randomize_client_port` et imbrication de `node.ephemeral.inactivity_timeout: 30m`). Voir [Headscale & Headplane](/services/headscale-headplane).
  - **Immich `v3.2.0`** — Migration majeure du serveur et du conteneur de machine learning avec réindexation des modèles d'inférence. Voir [Immich](/services/immich).
  - **Uptime Kuma `v2.5.4`** — Mise à jour du moteur de supervision actif et de la page de statut. Voir [Uptime Kuma](/services/uptime-kuma).
  - **Ntfy `v2.28.0`** — Mise à jour du serveur de notifications push. Voir [Ntfy](/services/ntfy).
  - **Dozzle `v11.0.1`** — Passage en version majeure v11 de l'afficheur live des logs Docker. Voir [Dozzle](/services/dozzle).
  - **CrowdSec Agent `v1.8.1` & Shield Web UI `2026.8.3`** — Mise à jour du moteur d'analyse comportementale et de l'interface de gestion du pare-feu. Voir [CrowdSec](/services/crowdsec).

  ### 🖥️ Automatisation Proxmox & Template Cloud-Init (VM 8000)

  - **Enrôlement Automatique Tailscale sur Template Ubuntu 24.04** — Ajout d'une configuration Cloud-Init `vendor-data.yaml` sur le template Proxmox (`VMID 8000`). Toute nouvelle VM clonée installe Tailscale et rejoint automatiquement le Tailnet privé sur `vpn.ims-world.fr` dès son premier boot via une clé réutilisable. Voir [Hôte Proxmox MS-01](/infrastructure/proxmox-host) et [Ajout d'une machine Headscale](/procedures/ajout-machine-headscale).
  - **Chantiers Ansible & Audit des Procédures** — Inscription à la feuille de route des playbooks d'automatisation de création de VM, du portail self-service de VM éphémères et de l'audit complet des procédures opérationnelles (Chantier 1.6). Voir [Feuille de Route](/procedures/roadmap).

  ### 🏠 Domotique & Déploiement VM HAOS (Mac Mini)

  - **Bascule d'Architecture & Déploiement HAOS (VMID 106)** — Abandon de l'ancien conteneur Docker sur Coolify et déploiement d'une machine virtuelle dédiée **Home Assistant OS 18.2 / Core 2026.9.2** sur le Mac Mini (`pve-macmini`). Raccordement natif au bridge `vmbr0` pour restaurer la découverte mDNS/Bonjour directe (Philips Hue, Apple HomeKit Bridge).
  - **Routage HTTPS Traefik (Zone 2 `vpn-only`) & Découverte UI HTTP HAOS 2026+** — Configuration du reverse proxy centralisé Traefik sur la VM 104 (`vpn-only.yaml`) acheminant `https://home.ims-world.fr` vers le backend LAN `http://192.168.1.92:80` avec certificat Let's Encrypt. Diagnostic et résolution du blocage HTTP 400 consécutif à l'obsolescence du bloc `http:` dans `configuration.yaml` au profit de la nouvelle interface graphique **Paramètres > Système > Réseau > Serveur HTTP** (activation de `X-Forwarded-For` et déclaration des `trusted_proxies`).
  - **Accès Distant Sécurisé via Add-on Tailscale** — Déploiement du module complémentaire Tailscale connecté au serveur Headscale (`vpn.ims-world.fr`), attribuant l'adresse IP Tailnet `100.64.0.5`. Voir [Home Assistant](/services/home-assistant).
</Update>

<Update label="13/09/2026" description="Intégration OIDC Authentik - Headscale (Groupe Membres) & Validation Premier Enrôlement">
  ### 🔐 Fédération d'Identité & Contrôle d'Accès (OIDC)

  - **Interconnexion Authentik ➔ Headscale OIDC** — Déploiement d'un fournisseur d'identité OAuth2/OIDC dédié (`headscale-oidc`) sur Authentik avec chiffrement PKCE et restriction stricte aux groupes `admins` et `membres`.
  - **Validation du Premier Enrôlement Utilisateur** — Enregistrement réussi du premier compte utilisateur (`cmolotkoff`) sur le Tailnet avec redirection web interactive via `https://vpn.ims-world.fr/oidc/callback`. Création automatique de l'espace de nommage utilisateur dans Headscale. Voir [Headscale & Headplane](/services/headscale-headplane) et [Authentik](/services/authentik).
  - **Mise à Jour de la Matrice de Sécurité** — Formalisation des niveaux d'habilitation OIDC et des accès réseau pour les membres du Tailnet. Voir [Matrice de Sécurité](/reseau/matrice-securite-exposition).

  ### 📋 Planification & Résilience

  - **Cadrage des Accès Elo & VM Dédiée** — Inscription à la feuille de route de la procédure d'accès Tailnet restreint et du dimensionnement d'une VM de travail dédiée.
  - **Supervision & Sauvegardes VM** — Inscription du chantier d'audit global des snapshots PBS et de la validation des alertes Ntfy sur l'ensemble des machines virtuelles. Voir [Feuille de Route](/procedures/roadmap).
</Update>

<Update label="29/08/2026 - 31/08/2026" description="Cadrage Roadmap : Services IA (Page Agent Alibaba, Meetilty) & Audit Nœud Worker">
  ### 💡 Innovation & Backlog Applicatif

  - **Cadrage Nouveaux Services IA** — Intégration dans la feuille de route de **Page Agent** (automatisation de navigation par agent IA browser Alibaba) et de **Meetilty** (prise de notes et transcription locale de réunions).
  - **Audit de Résilience Multi-Nœuds** — Consolidation de la documentation sur le Nœud Worker LXC 105 du Mac Mini et intégration des règles de redondance Traefik. Voir [Feuille de Route](/procedures/roadmap).
</Update>

<Update label="28/08/2026" description="Topologie Multi-Nœuds Coolify v4.3.14 (Master VM 104 + Worker LXC 105 Mac Mini)">
  ### 🚀 Orchestration & Multi-Nœuds

  - **Mise à Jour Coolify `v4.3.14` & Enrôlement Nœud Worker Distant** — Passage de l'instance principale en **Coolify v4.3.14** (Master / Control Plane sur VM 104 `ims-coolify`) et création du Nœud Worker LXC 105 (`pve-macmini-worker_LXC-105` sur Debian 13, IP `192.168.1.198`) sur le Mac Mini.
  - **Securisation SSH & Nettoyage RAM** — Authentification SSH dédiée via la clé `coolify-secondary-node` (`prohibit-password`), routage réseau unifié sur le Traefik principal VM 104 et suppression des conteneurs d'IHM autonomes pour préserver la RAM du Mac Mini. Voir [Coolify Worker LXC 105](/infrastructure/lxc-coolify-worker) et [VM Coolify](/infrastructure/vm-coolify).
</Update>

<Update label="26/08/2026" description="Pages d'Erreur Custom 404/5xx Traefik & Tuning CrowdSec AppSec WAF">
  ### 🖼️ Reverse Proxy & UX

  - **Mise en Production des Pages d'Erreur Custom Traefik** — Déploiement du conteneur helper Nginx (`vciwi7dolcl0hw1mffvjcfha`) et des middlewares `error-404@file` & `error-5xx@file`. Masquage de sécurité des erreurs 403 (`vpn-only`) en 404 générique pour la discrétion réseau (stealth) et affichage dynamique des codes 5xx via paramètre d'URL. Voir [Traefik (Coolify Proxy)](/reseau/traefik-proxy#-gestion-globale-des-pages-derreur-custom-404-403-masqué--5xx).

  ### 🛡️ Sécurité & WAF

  - **Correction des Faux Positifs AppSec WAF** — Neutralisation des bannissements automatiques intempestifs générés par le scénario `crowdsecurity/crowdsec-appsec-outofband` sur les requêtes AJAX légitimes (Grafana, Jellyfin, Patrimo). Ajustement du profil `/etc/crowdsec/profiles.yaml` pour conserver l'enregistrement des alertes sans bloquer l'accès. Voir le [Post-Mortem du 26/08/2026](/history/incidents/2026-08-26-faux-positifs-crowdsec-appsec-outofband) et la fiche [CrowdSec](/services/crowdsec#pare-feu-applicatif-waf-appsec--tuning-anti-faux-positifs).
</Update>

<Update label="24/08/2026" description="Alloy Mac Mini, Migration Authentik v2026.8.0, Incident Bypass SSO Dozzle & Incident DNS Wildcard">
  ### 📊 Supervision & Infrastructure

  - **Réintégration de l'Agent Alloy sur le Mac Mini (`pve-macmini`)** — Redéploiement complet du démon systemd Grafana Alloy sur le Nœud 2 du cluster (`100.64.0.6`) avec `node_exporter` (CPU/RAM/disque), tâche cron 5m `smartmon.sh` (monitoring SMART du SSD 256 Go) et transmission des logs journald vers Loki (`10.10.10.2:3100`). Voir [Stack Monitoring](/services/monitoring) et [Mac Mini](/infrastructure/mac-mini).

  ### 🔑 Mises à jour Applicatives

  - **Montée de Version Authentik `2026.5.0` ➔ `2026.8.0`** — Migration réussie du serveur SSO, du worker et réconciliation forcée du conteneur Outpost proxy (`ak-outpost-ims-outpost`). Procédure de redéploiement et piège du socket Docker documentés sur [Authentik](/services/authentik#montées-de-version--réconciliation-des-outposts-upgrade-202680).
  - **Mise à Jour Dozzle `v10.7.4`** — Passage en version **v10.7.4** de la console de visualisation live des logs Docker (`logs.ims-world.fr`). Voir [Dozzle](/services/dozzle).

  ### 🐛 Incidents & Post-Mortems

  - **Correction du Bypass SSO Dozzle (`logs.ims-world.fr`)** — Résolution du contournement du SSO Authentik causé par la priorité du routeur Traefik Dynamic File Provider (`vpn-only.yaml`) sur les labels du provider Docker. Ajout obligatoire du middleware `authentik-dozzle@docker`. Voir le [Post-Mortem du 24/08/2026](/history/incidents/2026-08-24-bypass-sso-dozzle-traefik-file-provider).
  - **Résolution de l'Incident DNS `share.ims-world.fr` (Zipline)** — Diagnostic et correction de l'erreur `503 No Available Server` causée par le masquage silencieux du wildcard DNS `*.ims-world.fr` par des enregistrements `_acme-challenge.share` TXT résiduels dans la zone OVH. Nettoyage préventif des résidus ACME sur l'ensemble de la zone. Voir le [Post-Mortem du 24/08/2026](/history/incidents/2026-08-24-dns-wildcard-shadowing-share-ims-world-fr).
</Update>

<Update label="23/08/2026" description="Cluster Proxmox VE ims-cluster (Mac Mini + MS-01) & Durcissement Fail2ban Ntfy">
  ### 🖥️ Cluster Proxmox VE (`ims-cluster`)

  - **Création du Cluster Proxmox VE à 2 Nœuds** — Alignement de la version Proxmox VE en **9.2.11** sur le MS-01 et installation sur le Mac Mini 2012 (`pve-macmini.ims-world.fr`). Création du cluster `ims-cluster` et jointure du Mac Mini (Quorum 2/2 votes, réplication des comptes PAM et clés SSH). Voir [Mac Mini](/infrastructure/mac-mini) et [MS-01 Proxmox Host](/infrastructure/proxmox-host).
  - **Décision d'Architecture ADR-010** — Retrait de la migration du port SSH (port 22 conservé sur tous les nœuds) et maintien du hostname `pve` sur le MS-01 pour prévenir toute casse Corosync/PMXCFS du cluster. Voir [ADR-010](/history/adr/adr-010-maintien-port-ssh-22-et-hostname-cluster).

  ### 🛡️ Harmonisation & Durcissement Fail2ban (3 Hôtes)

  - **Déploiement Harmonisé Fail2ban & Alertes Ntfy** — Déploiement d'instances Fail2ban indépendantes sur les 3 hôtes (MS-01, Mac Mini et VM Coolify). Configuration du `jail.local` avec escalade progressive (`1h` à `1 semaine`), prison `recidive` (3 bannes en 24h ➔ 1 semaine) et transmission des alertes SSH en direct sur le topic Ntfy **`ims-alerts`**.
</Update>

<Update label="22/08/2026" description="Déploiement Détection d'Intrusions CrowdSec, Plugin Bouncer Traefik & Web UI Shield">
  ### 🛡️ Sécurité & Détection d'Intrusions (CrowdSec)

  - **Déploiement de l'Agent CrowdSec v1.7.8 & WAF AppSec** — Activation du moteur de détection L3/L4/L7 sur la VM 104 (`ims-coolify`). Configuration de l'acquisition des logs Docker (`acquis.yaml`), du pare-feu applicatif AppSec (Virtual Patching inband), et création des allowlists de protection anti-auto-ban (`tailscale` `100.64.0.0/10` et `home-lan` `192.168.1.0/24`).
  - **Integration du Plugin Bouncer Traefik v1.6.0** — Activation du filtrage dynamique en mode `stream` (sync 60s) sur les entrypoints HTTP/HTTPS avec garantie fail-open (`updateMaxFailure: -1`).
  - **Déploiement de l'Interface Web Shield (`shield.ims-world.fr`)** — Publication de la console de gestion locale CrowdSec Web UI protégée en `vpn-only` \+ SSO Authentik OIDC (rôle `ADMIN` strict). Voir [CrowdSec & Shield](/services/crowdsec).
</Update>

<Update label="21/08/2026" description="Refonte Complète & Monitoring Avancé (SMART, Uptime Kuma, Dashboards)">
  ### 📊 Métrologie & Supervision Avancée (Stack LGTM)

  - **Monitoring SMART Bare-Metal (`ms01-pve`)** — Déploiement du _textfile collector_ Node Exporter alimenté par `smartmon.sh` (cron 5 min) pour remonter la santé SMART, températures et heures de vol des disques physiques (`nvme0n1`, `sda` SSD, `sdb` HDD) avec respect strict du spin-down (`smartctl -n standby`).
  - **Scrape Uptime Kuma (Pull) & Métriques SSL** — Ingestion des métriques d'Uptime Kuma (`/metrics` Basic Auth) avec calcul des SLO 24h/7j/30j et suivi d'expiration des certificats SSL/TLS.
  - **Rétention 1 An Prometheus** — Passage de la rétention TSDB de 30 jours à **1 an** (`--storage.tsdb.retention.time=1y`) pour un coût de stockage estimé à ~5-6 Go/an.
  - **Nouveaux Dashboards Grafana Exécutifs** — Publication de 4 nouveaux dashboards de production : _Vue d'ensemble — IMS-WORLD_, _Uptime Kuma - Overview_, _Gestion des disques_, et _Traefik — Reverse Proxy_. Correction des labels sur le template _Node Exporter Full_ (ID `1860`). Voir [Stack Monitoring](/services/monitoring).
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

  - **Durcissement Démon Docker (`"userland-proxy": false`)** — Application de la recommandation officielle du **CIS Docker Benchmark** sur la VM Coolify. Basculement natif du noyau Linux (iptables `DNAT` \+ `MASQUERADE` \+ `net.ipv4.route_localnet`), éliminant la substitution d'IP par le binaire `docker-proxy`.
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
  - **Rétablissement & Fix Préventif** : Rétablissement à chaud sans reboot via `apt install linux-modules-extra-$(uname -r)` \+ `modprobe i915`, puis installation préventive du métapaquet générique `linux-modules-extra-generic` pour automatiser les futurs redémarrages noyau. Voir le [Post-Mortem du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri).
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
  - **Décommissionnement officiel du Mac Mini 2012** — L'ancien serveur principal Mac Mini 2012 a été officiellement déconnecté de la production suite à la migration réussie de l'ensemble des services applicatifs sur le MS-01. Il bascule en mode Standby Chaud de secours. Voir [Mac Mini 2012](/infrastructure/mac-mini).
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
  - **Patrimo en production** — Application Node.js sur `patrimo.ims-world.fr` en mode _Application Coolify Git Build_ avec déploiement continu à chaque push GitHub. Voir [Patrimo](/services/patrimo).
  - **Zipline en production** — Plateforme de partage de fichiers, captures d'écran (compatible ShareX) et raccourcissement de liens sur `share.ims-world.fr` avec SSO Authentik OIDC natif. Voir [Zipline](/services/zipline).
  - **VM Coolify en CPU mode `host` (ADR-005)** — Le CPU i5-12600H est exposé intégralement à la VM Coolify (`x86-64-v2`). Les binaires natifs exigeant `sharp` démarrant sans contournement. Voir [ADR-005](/history/adr/adr-005-cpu-vm-coolify-mode-host).
  - **Mise à niveau de Coolify en v4.3.2** — Amélioration de la gestion des Webhooks Git et des builds Compose.
  - **Relais mail Forgejo via Resend** — Notifications transactionnelles via `forgejo@ims-world.fr`.
</Update>

<Update label="13/08/2026" description="Stirling PDF en production">
  ### 📄 Outillage PDF Stateless

  - **Stirling PDF en production** — Boîte à outils PDF (fusion, découpe, conversion, OCR Tesseract v5) sur `pdf.ims-world.fr`, en mode _stateless_ (aucun document conservé après traitement). Authentification native désactivée (`DOCKER_ENABLE_SECURITY=false`). Voir [Stirling PDF](/services/stirling-pdf).
  - **Forward-Auth Authentik étendu** — L'Outpost Proxy Authentik (`ak-outpost-ims-outpost:9000`) s'intercale via Traefik en amont de Stirling PDF. Voir [Authentik](/services/authentik#outpost-proxy--forward-auth-traefik).
</Update>

<Update label="11/08/2026" description="Statuspage Uptime Kuma, IT-Tools & Immich">
  ### 📊 Monitoring, Tools & Médiathèque Photo

  - **Uptime Kuma & Alerting Ntfy** — Statuspage et monitoring actif sur `status.ims-world.fr` avec notifications push mobiles instantanées via Ntfy (templates LiquidJS pour états _down_ et _recovered_). Voir [Uptime Kuma](/services/uptime-kuma).
  - **IT-Tools en production** — Boîte à outils développeur (générateurs, convertisseurs, utilitaires réseau) sur `tools.ims-world.fr`. Voir [IT-Tools](/services/it-tools).
  - **Forward-Auth Authentik étendu** — Outpost Proxy Authentik configuré en amont de Uptime Kuma et IT-Tools.
  - **Immich en production** — Médiathèque photo & vidéo self-hosted sur `photos.ims-world.fr` avec sauvegarde automatique iOS/Android, recherche IA (CLIP) et reconnaissance faciale (61 880 assets indexés). Voir [Immich](/services/immich).
</Update>

<Update label="10/08/2026" description="Ntfy, Dozzle & Stack Monitoring LGTM">
  ### 🔔 Push Alerts, Logs Live & Stack LGTM

  - **Ntfy en production** — Serveur de notifications push sur `ntfy.ims-world.fr` (`ims-alerts`) servant de point de sortie unique pour toutes les alertes du homelab. Voir [Ntfy](/services/ntfy).
  - **Dozzle en production** — Viewer de logs Docker en direct sur `logs.ims-world.fr` protégé par Forward-Auth Authentik. Voir [Dozzle](/services/dozzle).
  - **Stack Monitoring LGTM** — Grafana, Loki et Prometheus sur `monitoring.ims-world.fr` avec agent Grafana Alloy en `remote-write` et SSO Authentik OIDC (`Grafana Admins`, `Editors`, `Viewers`). Voir [Monitoring](/services/monitoring).
  - **Alerting Grafana → Ntfy** — Webhook contact point poussant les alertes _firing_ (priorité 4) et _resolved_ (priorité 3) sur mobile.
  - **Règle d'or de routage Traefik/Coolify formalisée** — Documentation de l'obligation de laisser le champ _Domains_ vide dans l'UI Coolify dès qu'un service utilise un middleware sur-mesure (`vpn-only`, forward-auth) pour éviter l'exposition publique par un routeur parallèle automatique. Voir [Traefik Proxy](/reseau/traefik-proxy#️-règle-dor-de-routage--ui-coolify-vs-labels-compose-manuels).
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
  - **Afficheur Kiosk Raspberry Pi 3B\+** — Écran principal Wisecoco 7.84" (1280x400) et OLED 0.91" (128x32) dans un module 2U 3D pour le rack Labrax. Bouton poussoir GPIO (court = switch source, long 3s = extinction). Voir [Raspberry Pi 3B\+](/infrastructure/rpi-monitor).
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
  - **Déploiement du Socle Proxmox (Mi-juillet)** — Déploiement complet du socle Proxmox VE 9.2.3 : LXC 100 NAS (MergerFS \+ NFS \+ SMB), LXC 103 PBS (Proxmox Backup Server, datastore NFS), VM 104 Coolify (Docker \+ Coolify 4.1.2). Autostart et ordre de boot (NAS order=1 → PBS order=2 → Coolify order=3) validés par reboot physique.
</Update>
