---
title: "Feuille de Route & Liste TODO"
description: "Chantiers techniques restants, tâches d'optimisation et suivi des améliorations homelab"
icon: "list-check"
iconType: "duotone"
---

<Info>
Cette page recense l'ensemble des **chantiers techniques restants et tâches d'amélioration** identifiés pour faire évoluer l'infrastructure homelab IMS-WORLD post-cutover.
</Info>

## 📌 Liste des Chantiers & TODOs Prioritaires

### 1. 📊 Monitoring Global (Grafana + Loki + Prometheus)
- **Constat** : Le Raspberry Pi 3B+ est actuellement un afficheur Kiosk passif (voir [Raspberry Pi 3B+](/infrastructure/rpi-monitor)), et non une sonde de métrologie.
- **Tâche** : Déployer la stack Grafana, Loki et Prometheus avec l'agent **Grafana Alloy** sur chaque hôte (MS-01, VM Coolify, Mac Mini, RPi) pour collecter logs, températures, CPU/RAM et métriques d'usure SMART.

### 2. 💾 Sauvegardes des Conteneurs LXC 100 (NAS) & 103 (PBS) — 🟢 Effectué
- **Statut** : Configuré et validé ("Run now" fonctionnels). Jobs `vzdump` planifiés à 03h00 (LXC 103, mode `snapshot`) et 05h00 (LXC 100, mode `stop`) vers le stockage NVMe local (`/var/lib/vz/dump/`), conformément au principe anti-circularité. Voir la [Politique de Sauvegarde](/infrastructure/politique-sauvegardes).

### 3. 🛡️ Firewall Proxmox 3 Niveaux
- **Constat** : Le pare-feu natif de Proxmox VE (niveau Nœud → Datacenter → Guest) n'est pas encore activé.
- **Tâche** : Définir les règles de filtrage au niveau de l'hôte MS-01, tester la ré-ouverture SSH/GUI Proxmox immédiate et valider l'étanchéité avant toute nouvelle exposition.

### 4. 🎬 Passthrough GPU (iGPU Intel Iris Xe) — 🟢 Effectué
- **Statut** : iGPU Intel Iris Xe attribuée en passthrough PCIe (`hostpci0`) à la VM Coolify (VM 104) pour le transcodage matériel QuickSync (H.264/HEVC/AV1) de Jellyfin. Voir [Proxmox Host](/infrastructure/proxmox-host#gpu-igpu-iris-xe-passthrough-vm-coolify).

### 5. ⚡ Intégration du SSD 4To (Phase 4 Stockage)
- **Tâche** : Installer le SSD SATA 4To dans l'emplacement dédié du rack [Labrax](/infrastructure/labrax) et faire basculer le pool `/mnt/storage-hot` (bases de données Immich, Forgejo, Authentik) du HDD Seagate vers le SSD.

### 6. 🔑 Sécurisation des Credentials OVH du Proxy Traefik
- **Tâche** : Extraire les identifiants d'API OVH (challenge DNS-01 Let's Encrypt) du fichier docker-compose clair et les basculer dans un fichier d'environnement restreint (`.env`).

### 7. 🔒 Audit de Sécurité Middleware `vpn-only` — 🟢 Effectué
- **Statut** : Étanchéité du middleware `vpn-only` validée sur l'ensemble de la stack (qBittorrent, Radarr, Sonarr, Prowlarr) — accès restreint au Tailnet 100.64.0.0/10 et bloqué depuis le WAN (403 Forbidden). Voir [Traefik Proxy](/reseau/traefik-proxy).

### 8. 🏷️ Renommage Éventuel du Domaine Control Plane VPN
- **Tâche** : Évaluer le renommage de `vpn.ims-world.fr` vers un nom plus représentatif (ex: `controlplane.ims-world.fr`), en prenant en compte la reconfiguration obligatoire de tous les appareils enregistrés sur le Tailnet.

### 9. 💾 Extension Capacitive NAS — Ajout HDD 8 To SATA
- **Constat** : Le pool de stockage capacitif `storage` sur le NAS (3 To ext4 actuel) est soumis à une forte pression continue liée au volume de la médiathèque HomeFlix et à la réintégration des photos RAW PhotoPrism (`originals/`).
- **Tâche** : Installer un disque dur HDD 8 To SATA additionnel sur le port SATA libre identifié de la carte PCIe ASM1166 et l'intégrer au pool MergerFS du NAS LXC 100 pour offrir une marge capacitive durable. Voir [IMS-NAS](/infrastructure/ims-nas).

### 10. 🚀 Migration à Froid des Services Secondaires (~11 Services)
- **Tâche** : Migrer progressivement les applications encore hébergées sur l'ancien Mac Mini vers la VM Coolify du MS-01 : Beszel, Zipline, Forgejo, PhotoPrism, Immich, Ntfy, Home Assistant, Patrimo, Sentryx.

### 11. 🔒 Réactivation de `only_start_if_oidc_is_available` sur Headscale
- **Constat** : Le paramètre `only_start_if_oidc_is_available` avait été passé à `false` pendant le cutover pour contourner une dépendance circulaire au démarrage.
- **Tâche** : Repasser le paramètre à `true` dans la config Headscale maintenant que `auth.ims-world.fr` est parfaitement stable sur le MS-01.

### 12. 🔐 Régénération de la Clé d'Authentification Headscale PBS
- **Tâche** : Régénérer la clé d'API et le jeton d'authentification Headscale utilisés lors de l'intégration initiale du conteneur PBS (LXC 103).

### 13. 🧹 Décommissionnement Définitif du Mac Mini
- **Tâche** : Une fois la période de validation post-cutover achevée et l'ensemble des 11 services secondaires migrés, procéder au retrait propre du Mac Mini 2014 du réseau et retirer les `extra_records` dans Headscale (`coolify-old.ims-world.fr`).
