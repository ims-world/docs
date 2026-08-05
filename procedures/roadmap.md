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

### 2. 💾 Sauvegardes des Conteneurs LXC 100 (NAS) & 103 (PBS)
- **Constat** : Actuellement, seuls les snapshots de la VM 104 (Coolify) sont sauvegardés par PBS. Le LXC NAS et le LXC PBS constituent deux points de défaillance uniques (SPOF) non sauvegardés (voir [Plan de Reprise PRA](/procedures/plan-reprise-activite-pra)).
- **Tâche** : Configurer des jobs `vzdump` quotidiens vers le stockage NVMe local du host MS-01 (`/var/lib/vz/dump/`), conformément au principe anti-circularité.

### 3. 🛡️ Firewall Proxmox 3 Niveaux
- **Constat** : Le pare-feu natif de Proxmox VE (niveau Nœud → Datacenter → Guest) n'est pas encore activé.
- **Tâche** : Définir les règles de filtrage au niveau de l'hôte MS-01, tester la ré-ouverture SSH/GUI Proxmox immédiate et valider l'étanchéité avant toute nouvelle exposition.

### 4. 🎬 Passthrough GPU (iGPU Intel Iris Xe)
- **Constat** : Jellyfin transcode actuellement sur CPU (6 vCPU de la VM 104).
- **Tâche** : Activer IOMMU (`intel_iommu=on`) dans le GRUB de l'hôte MS-01, mapper le périphérique `/dev/dri/renderD128` dans la VM 104 et activer le transcodage matériel QuickSync (H.264/HEVC/AV1) dans Jellyfin.

### 5. ⚡ Intégration du SSD 4To (Phase 4 Stockage)
- **Tâche** : Installer le SSD SATA 4To dans l'emplacement dédié du rack [Labrax](/infrastructure/labrax) et faire basculer le pool `/mnt/storage-hot` (bases de données Immich, Forgejo, Authentik) du HDD Seagate vers le SSD.

### 6. 🔑 Sécurisation des Credentials OVH du Proxy Traefik
- **Tâche** : Extraire les identifiants d'API OVH (challenge DNS-01 Let's Encrypt) du fichier docker-compose clair et les basculer dans un fichier d'environnement restreint (`.env`).

### 7. 🔒 Audit de Sécurité Middleware `vpn-only`
- **Tâche** : Re-tester systématiquement l'étanchéité du middleware `vpn-only` sur les interfaces web de Sonarr et Prowlarr depuis une IP publique WAN externe (comme déjà validé pour qBittorrent et Radarr).

### 8. 🏷️ Renommage Éventuel du Domaine Control Plane VPN
- **Tâche** : Évaluer le renommage de `vpn.ims-world.fr` vers un nom plus représentatif (ex: `controlplane.ims-world.fr`), en prenant en compte la reconfiguration obligatoire de tous les appareils enregistrés sur le Tailnet.

### 9. 🚀 Migration à Froid des Services Secondaires (~11 Services)
- **Tâche** : Migrer progressivement les applications encore hébergées sur l'ancien Mac Mini vers la VM Coolify du MS-01 : Beszel, Zipline, Forgejo, Photoprism, Immich, Ntfy, Home Assistant, Patrimo, Sentryx.

### 10. 🔒 Réactivation de `only_start_if_oidc_is_available` sur Headscale
- **Constat** : Le paramètre `only_start_if_oidc_is_available` avait été passé à `false` pendant le cutover pour contourner une dépendance circulaire au démarrage.
- **Tâche** : Repasser le paramètre à `true` dans la config Headscale maintenant que `auth.ims-world.fr` est parfaitement stable sur le MS-01.

### 11. 🔐 Régénération de la Clé d'Authentification Headscale PBS
- **Tâche** : Régénérer la clé d'API et le jeton d'authentification Headscale utilisés lors de l'intégration initiale du conteneur PBS (LXC 103).

### 12. 🧹 Décommissionnement Définitif du Mac Mini
- **Tâche** : Une fois la période de validation post-cutover achevée et l'ensemble des 11 services secondaires migrés, procéder au retrait propre du Mac Mini 2014 du réseau et retirer les `extra_records` dans Headscale (`coolify-old.ims-world.fr`).
