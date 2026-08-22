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

### 5. ⚡ Intégration du SSD 4To (Phase 4 Stockage) — 🟢 Effectué
- **Statut** : Bascule effectuée avec succès le 18/08/2026. Le tier `storage-hot` (`immich-data`, `forgejo-data`) a été migré du HDD principal vers le SSD Samsung 870 EVO 4To raccordé en SATA sur le contrôleur ASM1166 (passthrough `mp1` → LXC 100), libérant 934 Go sur le HDD. Voir [IMS-NAS](/infrastructure/ims-nas#logique-de-découpage--stockage-capacitif-vs-données-chaudes) et la [Chronologie](/history/chronologie#18082026--bascule-du-tier-storage-hot-sur-ssd-4to-dédié-sata).

### 6. 🔑 Sécurisation des Credentials OVH du Proxy Traefik
- **Tâche** : Extraire les identifiants d'API OVH (challenge DNS-01 Let's Encrypt) du fichier docker-compose clair et les basculer dans un fichier d'environnement restreint (`.env`).

### 7. 🔒 Configuration `userland-proxy: false` & SNAT Tailscale Bypass (ADR-009) — 🟢 Effectué
- **Statut** : Implémenté et validé les 19-20/08/2026. La désactivation de `userland-proxy` dans `/etc/docker/daemon.json` (CIS Docker Benchmark) et l'exécution de `sudo tailscale set --snat-subnet-routes=false` ont restauré la préservation absolue des IP sources réelles (`100.64.0.x`) jusqu'au conteneur Traefik.
- **Point de contrôle futur** : Valider formellement la persistance de `--snat-subnet-routes=false` lors du prochain redémarrage à froid de maintenance du serveur physique MS-01. Voir [Post-Mortem 18-20/08/2026](/history/incidents/2026-08-18-blocage-traefik-vpn-only-docker-proxy).

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

### 14. 🔔 Notifications Nativités Proxmox VE & PBS
- **Tâche** : Configurer les cibles de notification natifs (*Notification Targets*) sur Proxmox VE (MS-01) et Proxmox Backup Server (LXC 103) pour recevoir les comptes-rendus d'exécution des jobs de sauvegarde `vzdump` / `pbs` et les alertes d'état système vers Ntfy / Mail.

### 15. 📦 Scan Automatique des Mises à Jour Docker (Diun)
- **Tâche** : Déployer **Diun** (*Docker Image Update Notifier*) sur la VM Coolify pour surveiller les registres Docker et émettre une alerte Webhook instantanée dès qu'une nouvelle version d'image conteneurisée est disponible.

### 16. 🛡️ Détection d'Intrusions CrowdSec, Plugin Traefik & Web UI Shield (🟢 Effectué le 22/08/2026)
- **Tâche** : Déployer le moteur de détection CrowdSec v1.7.8 sur la VM Coolify, le plugin bouncer Traefik (mode stream, fail-open `updateMaxFailure: -1`), l'AppSec WAF (196 règles inband), les allowlists anti-auto-ban (`tailscale` et `home-lan`), et l'interface Web d'administration Shield (`shield.ims-world.fr`). Voir [CrowdSec & Shield](/services/crowdsec).

### 17. 🎬 Visibilité Temps Réel des Flux Jellyfin (Exporteur Dédié)
- **Tâche** : Sélectionner et déployer un conteneur exporteur Prometheus dédié à Jellyfin (`jellyfin-exporter`) sur la VM Coolify pour remonter la métrologie en temps réel des lectures actives (sessions de transcodage vs direct play, utilisateurs connectés, codecs et débits).
