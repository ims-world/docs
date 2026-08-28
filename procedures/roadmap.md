---
title: "Feuille de Route & Liste TODO"
description: "Suivi centralisé des chantiers prioritaires, roadmap de résilience et backlog d'évolution de l'infrastructure"
icon: "list-check"
iconType: "duotone"
last_reviewed: "2026-08-24"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Mis à Jour le 24/08/2026</Badge>

<Info>
Cette page constitue le **journal central de suivi des chantiers et de la feuille de route** du homelab IMS-WORLD. Elle regroupe l'ensemble des tâches ouvertes classées par domaine d'intervention (Résilience, Sécurité, Supervision, Nouveaux Services, Matériel) ainsi que l'historique des jalons réalisés.
</Info>

---

## 📊 Matrice d'Avancement des Chantiers

| Domaine | Chantier | Priorité | Hôte Cible | Statut |
|---|---|---|---|---|
| **Résilience & Quorum** | QDevice Corosync 3ᵉ vote | <Badge color="red">🔴 Priorité 1</Badge> | Raspberry Pi 3B+ | ⏳ En attente |
| **Résilience & Alerting** | Supervision & Alerting hors-MS-01 | <Badge color="red">🔴 Priorité 1</Badge> | Mac Mini / RPi / VPS | ⏳ En attente |
| **Supervision & Métrologie** | Agent Alloy systemd sur Mac Mini | <Badge color="green">🟢 Effectué</Badge> | Mac Mini (`100.64.0.6`) | ✅ 24/08/2026 |
| **Alerting** | Notifications Ntfy sur échec backup | <Badge color="amber">🟡 Moyen Terme</Badge> | MS-01 / PBS | ⏳ En attente |
| **Sauvegardes & PRA** | Politique de sauvegarde PBS incluant le nœud Mac Mini (`pve-macmini`) | <Badge color="amber">🟡 Moyen Terme</Badge> | Cluster / PBS | ⏳ En attente |
| **Sécurité** | Credentials OVH dans fichier `.env` | <Badge color="amber">🟡 Moyen Terme</Badge> | VM 104 (Coolify Proxy) | ⏳ En attente |
| **Sécurité** | DMZ & Bastion SSH d'administration | <Badge color="blue">🟦 À Évaluer</Badge> | Réseau / DMZ | 💡 Étude |
| **Sécurité** | Firewall Proxmox VE 3 niveaux | <Badge color="amber">🟡 Moyen Terme</Badge> | MS-01 & Mac Mini | ⏳ En attente |
| **Sécurité** | Intel vPro / AMT (Gestion Out-of-Band) | <Badge color="blue">🟦 À Évaluer</Badge> | MS-01 Bare-Metal | 💡 Étude |
| **Docker & Hygiène** | Pinning des tags Docker (suppression `:latest`) | <Badge color="amber">🟡 Moyen Terme</Badge> | VM 104 (Coolify) | ⏳ En attente |
| **Docker & Hygiène** | Scan & Alertes mises à jour (Diun Ntfy) | <Badge color="amber">🟡 Moyen Terme</Badge> | VM 104 (Coolify) | ⏳ En attente |
| **Supervision** | Exporteur Prometheus Jellyfin | <Badge color="blue">🟦 Nouveaux Services</Badge> | VM 104 (Coolify) | ⏳ En attente |
| **UX & Proxy** | Pages d'erreur custom Traefik (404/502/503/504) | <Badge color="green">🟢 Effectué</Badge> | Traefik Proxy | ✅ 26/08/2026 |
| **Proxy & Ingress** | PoC & Évaluation Caddy v2 (xcaddy + CrowdSec + OVH) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify Proxy) | 💡 Étude / PoC |
| **Nouveaux Services** | DNS Secondaire (AdGuard Home / Pi-hole) | <Badge color="blue">🟦 Nouveaux Services</Badge> | Mac Mini / RPi | ⏳ En attente |
| **Nouveaux Services** | Compression & Optimisation d'Images (Imgcompress) | <Badge color="blue">🟦 Nouveaux Services</Badge> | VM 104 (Coolify) | ⏳ En attente |
| **Stockage** | Migration stockage Zipline vers SSD 4To | <Badge color="amber">🟡 Moyen Terme</Badge> | LXC 100 / VM 104 | ⏳ En attente |
| **Forge Git** | Synchronisation credentials miroirs GitHub | <Badge color="amber">🟡 Moyen Terme</Badge> | Forgejo (VM 104) | ⏳ En attente |
| **Nouveaux Services** | Homepage (Dashboard de navigation unifié) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **Sécurité Applicative** | Strix / Usestrix (Analyse de sécurité & dépendances) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **Sécurité Code** | Semgrep App (Analyse statique de code SAST) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **Nouveaux Services** | Dawarish (Suivi & timeline de géolocalisation) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **Nouveaux Services** | Keep It Shot (Gestion & OCR de captures d'écran) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **Nouveaux Services** | Cap.so / Cap io (Studio d'enregistrement d'écran vidéo) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **IA & Sécurité** | Cyber Strike IA (Simulation d'attaques cyber IA) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **Matériel & Rack** | Extension physique du rack Labrax 10" | <Badge color="amber">🟡 Moyen Terme</Badge> | Rack Physique | ⏳ En attente |
| **Stockage** | Extension capacitive HDD 4To / 8To Neuf | <Badge color="blue">🟦 À Évaluer</Badge> | NAS LXC 100 | 💡 Achat futur |
| **Sécurité** | Détection d'intrusions NIDS & Sentryx | <Badge color="blue">🟦 À Évaluer</Badge> | Réseau / VM 104 | 💡 Étude |

---

## 🔴 1. Chantiers Critiques & Haute Priorité (Quorum & Résilience)

### 1.1 🖥️ Quorum Cluster & QDevice Corosync (Raspberry Pi 3B+) <Badge color="red">🔴 Priorité 1</Badge>
- **Constat** : Le cluster Proxmox VE `ims-cluster` comporte 2 nœuds (MS-01 et Mac Mini). Sans un troisième vote d'arbitrage, la perte de l'un des deux hôtes entraîne la perte de quorum sur le nœud survivant.
- **Tâche** : Déployer le démon **QDevice Corosync** (`corosync-qnetd` / port TCP `5403`) sur le Raspberry Pi 3B+ (`ims-rpi-monitor`) pour accorder le 3ᵉ vote d'arbitrage et sécuriser le quorum en cas d'extinction de l'un des deux hyperviseurs.

### 1.2 🚨 Supervision & Alerting d'Uptime Hors-Hôte (Panne MS-01) <Badge color="red">🔴 Priorité 1</Badge>
- **Constat** : Toute la stack de supervision (Prometheus, Grafana, Loki, Uptime Kuma, Ntfy) est hébergée sur la VM 104 du MS-01. Si le MS-01 subit une panne matérielle ou électrique totale, aucune alerte ne peut être émise.
- **Tâche** : Déployer un mécanisme d'alerting léger hors-hôte (sur le Mac Mini, le Raspberry Pi ou un micro VPS externe) pour surveiller la joignabilité du MS-01 et envoyer une alerte push si l'hôte principal s'éteint.

### 1.3 🔔 Notifications Ntfy sur Échec des Sauvegardes Proxmox VE / PBS <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Configurer les cibles de notification natifs (*Notification Targets*) sur Proxmox VE (MS-01) et Proxmox Backup Server (LXC 103) pour émettre une alerte Ntfy automatique immédiate sur le topic `ims-alerts` en cas d'échec d'un job de sauvegarde `vzdump` ou `pbs`.

### 1.4 🛡️ Mise à Jour de la Politique de Sauvegarde PBS pour le Nœud Mac Mini (`pve-macmini`) <Badge color="amber">🟡 Moyen Terme</Badge>
- **Constat** : Suite à l'intégration du Mac Mini en tant que Nœud 2 actif du cluster `ims-cluster`, les tâches de sauvegarde planifiées de Proxmox Backup Server (PBS LXC 103) et `vzdump` doivent être révisées pour couvrir les VM/LXC hébergés ou migrés sur ce nœud.
- **Tâche** : Déclarer le nœud `pve-macmini` dans le Datacenter PVE (jobs de sauvegarde `all` / multi-nœuds) et ajuster la politique de rétention et d'exclusion (LXC 100 NAS maintenu en exclusion manuelle stop-only). Voir [Politique de Sauvegardes](/infrastructure/politique-sauvegardes).

---

## 🔒 2. Sécurité & Durcissement Système

### 2.1 🔑 Credentials OVH du Proxy Traefik dans Fichier `.env` <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Extraire les identifiants d'API OVH (DNS-01 Let's Encrypt) du fichier `/data/coolify/proxy/docker-compose.yml` et les placer dans un fichier d'environnement restreint (`.env` avec `chmod 600`).

### 2.2 🏰 Exploration d'une DMZ & Bastion SSH <Badge color="blue">🟦 À Évaluer</Badge>
- **Tâche** : Étudier l'architecture d'un bastion d'administration SSH isolé en DMZ pour centraliser, authentifier et auditer l'ensemble des accès shell d'infrastructures distants.

### 2.3 🛡️ Activation du Firewall Proxmox VE 3 Niveaux <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Activer et configurer le pare-feu natif de Proxmox VE (Nœud ➔ Datacenter ➔ Guest) sur MS-01 et Mac Mini avec règles d'étanchéité sur les bridges `vmbr0` et `vmbr1`.

### 2.4 ⚙️ Configuration Out-of-Band Intel vPro / AMT <Badge color="blue">🟦 À Évaluer</Badge>
- **Tâche** : Configurer le module d'aménagement à distance Intel vPro / AMT sur le Minisforum MS-01 pour conserver la prise de main KVM matérielle bas niveau même OS éteint.

---

## 🐳 3. Supervision, Hygiène Docker & Métrologie

### 3.1 🏷️ Pinning des Tags Docker (Suppression des Tags `:latest`) <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Figer les images Docker sur des versions sémantiques précises (SemVer) pour Stirling PDF et Zipline afin d'éliminer le risque de rupture accidentelle lors d'un `docker pull`.

### 3.2 📊 Extension de l'Agent Alloy sur le Mac Mini (`pve-macmini`) <Badge color="green">🟢 Effectué le 24/08/2026</Badge>
- **Statut** : Agent Grafana Alloy systemd réintégré avec succès sur le Mac Mini (`100.64.0.6`).
- **Composants** : Node Exporter (CPU, RAM, disque), collecteur SMART (`smartmon.sh` cron 5m sur SSD Apple 256 Go) et transmission des logs journald/syslog vers Loki (`10.10.10.2:3100`). Voir [Mac Mini](/infrastructure/mac-mini) et [Stack Monitoring](/services/monitoring).

### 3.3 📦 Scan & Alertes des Mises à Jour Docker (Diun) <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Déployer **Diun** (*Docker Image Update Notifier*) sur la VM Coolify pour surveiller les registres Docker et pousser une notification Webhook sur Ntfy dès qu'une version stable est publiée.

### 3.4 🎬 Exporteur Prometheus Dédié Jellyfin (`jellyfin-exporter`) <Badge color="blue">🟦 Nouveaux Services</Badge>
- **Tâche** : Déployer `jellyfin-exporter` sur la VM Coolify pour remonter à Grafana les métriques en temps réel des lectures actives (sessions transcodées vs direct play, débits, codecs).

### 3.5 🖼️ Pages d'Erreur & Indisponibilité Custom Traefik <Badge color="green">🟢 Effectué le 26/08/2026</Badge>
- **Statut** : Déploiement du conteneur helper Nginx (`vciwi7dolcl0hw1mffvjcfha`, `nginx:1.27-alpine`) et configuration du Dynamic File Provider `/data/coolify/proxy/dynamic/error-pages.yaml`.
- **Fonctionnalités** : Interception des 404 & 403 (masquage discret des 403 en 404), redirection dynamique `{status}` des erreurs 5xx et routeur `catchall-error-pages`. Voir [Traefik (Coolify Proxy)](/reseau/traefik-proxy#-gestion-globale-des-pages-derreur-custom-404-403-masqué--5xx).

### 3.6 ⚡ PoC & Évaluation de Caddy v2 (Remplacement Proxy Traefik v3) <Badge color="blue">🟦 À Évaluer — Étude PoC</Badge>
- **Constat** : Caddy v2 offre une empreinte mémoire très faible (~30 Mo RAM), une configuration synthétique via `Caddyfile` (éliminant la verbosité des labels Docker et la complexité des priorités File vs Docker Provider) et est proposé comme alternative officielle dans Coolify v4.
- **Tâche** : Réaliser un PoC de build d'une image custom Caddy (`xcaddy` avec plugins `caddy-dns/ovh` et `caddy-crowdsec-bouncer`), tester la réécriture du `Caddyfile` avec filtres `vpn-only` + `forward_auth` Authentik, et évaluer la bascule du proxy dans Coolify.

---

## 🆕 4. Nouveaux Services & Projets Applicatifs

### 4.1 🛡️ Serveur DNS Secondaire (AdGuard Home / Pi-hole) <Badge color="blue">🟦 Nouveaux Services</Badge>
- **Tâche** : Déployer une instance DNS locale secondaire (sur Mac Mini ou RPi) pour garantir la résolution DNS interne et le filtrage publicitaire même en cas de maintenance du MS-01.

### 4.2 🖼️ Compression & Optimisation d'Images (Imgcompress) <Badge color="blue">🟦 Nouveaux Services</Badge>
- **Tâche** : Déployer une solution d'optimisation automatisée d'images (Imgcompress ou équivalent) pour réduire le poids des visuels avant intégration.

### 4.3 💾 Migration Stockage Zipline vers Tier SSD `storage-hot` <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Migrer le répertoire de stockage Zipline du HDD vers le SSD 4To Samsung 870 EVO (`/mnt/storage-hot`) pour accélérer le traitement des uploads ShareX.

### 4.4 🐙 Sync Credentials GitHub (Forgejo) <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Configurer les jetons d'accès et identifiants de synchronisation automatique sur les 6 dépôts miroirs GitHub hébergés sur Forgejo.

### 4.5 🏠 Dashboard de Navigation Homelab Unifié (Homepage) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Déployer **Homepage** sur la VM Coolify pour disposer d'un portail d'accueil moderne, centralisant l'accès à tous les services homelab avec intégrations d'état (ping, métriques, statuts Docker/Traefik).

### 4.6 🦅 Analyse & Audit de Sécurité des Dépendances (Strix / Usestrix) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Évaluer et déployer la plateforme **Strix** pour auditer la sécurité des dépendances logicielles et détecter les vulnérabilités CVE applicatives.

### 4.7 🔍 Analyse Statique de Code SAST (Semgrep App) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Déployer l'instance self-hosted de **Semgrep App** sur la VM Coolify pour automatiser l'analyse statique de sécurité (SAST) du code source des projets développés en interne.

### 4.8 🗺️ Suivi & Timeline de Géolocalisation Personnelle (Dawarish) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Déployer **Dawarish** (alternative self-hosted à Google Location History) pour historiser et visualiser sur une carte interactive les déplacements et la chronologie de géolocalisation.

### 4.9 📸 Gestionnaire & OCR de Captures d'Écran (Keep It Shot) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Déployer **Keep It Shot** pour organiser, indexer et exécuter la reconnaissance optique de caractères (OCR) sur les captures d'écran et médias de travail.

### 4.10 🎥 Studio d'Enregistrement d’Écran & Édition Vidéo (Cap.so / Cap io) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Évaluer l'hébergement de **Cap.so (Cap io)** pour enregistrer, éditer et partager rapidement des démonstrations et séquences vidéo directement depuis le navigateur.

### 4.11 ⚔️ Simulation & Évaluation d'Attaques Cyber par IA (Cyber Strike IA) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Évaluer et intégrer la plateforme **Cyber Strike IA** pour simuler des scénarios d'attaques cyber et éprouver la résilience de l'infrastructure homelab face à des menaces automatisées.

---

## 🗄️ 5. Matériel, Physique & Stockage Long Terme

### 5.1 🛠️ Extension Rack Labrax 10" <Badge color="amber">🟡 Moyen Terme</Badge>
- **Tâche** : Achever le montage physique et le câblage propre de l'extension de châssis 10 pouces du rack [Labrax](/infrastructure/labrax).

### 5.2 💾 Achat HDD 4To / 8To Neuf <Badge color="blue">🟦 À Évaluer</Badge>
- **Tâche** : Acquérir un disque dur HDD supplémentaire pour étendre le pool capacitif et/ou instaurer un miroir de parité sur le NAS.

### 5.3 🔍 Sonde NIDS / Sentryx <Badge color="blue">🟦 À Évaluer</Badge>
- **Tâche** : Évaluer la mise en service d'une sonde de détection d'intrusions réseau (NIDS) et du projet Sentryx.

---

## 🟢 6. Chantiers Récents Effectués & Archives

<AccordionGroup>
  <Accordion title="🖥️ Cluster Proxmox VE ims-cluster & Fail2ban Harmonisé — 23/08/2026">
    Installation de Proxmox VE 9.2.11 sur Mac Mini, création du cluster à 2 nœuds `ims-cluster` (Quorum 2/2 votes), formalisation de l'ADR-010 et déploiement harmonisé de Fail2ban avec alertes Ntfy sur MS-01, Mac Mini et VM Coolify. Voir [Mac Mini](/infrastructure/mac-mini) et [ADR-010](/history/adr/adr-010-maintien-port-ssh-22-et-hostname-cluster).
  </Accordion>

  <Accordion title="🛡️ Détection d'Intrusions CrowdSec v1.7.8 & WAF AppSec — 22/08/2026">
    Déploiement de l'agent CrowdSec, du plugin bouncer Traefik (mode stream fail-open `updateMaxFailure: -1`), du WAF AppSec (196 règles inband), des allowlists `tailscale`/`home-lan` et de l'IHM locale **Shield** (`shield.ims-world.fr`). Voir [CrowdSec](/services/crowdsec).
  </Accordion>

  <Accordion title="📊 Stack Monitoring LGTM (Grafana / Loki / Prometheus / Alloy) — 10-21/08/2026">
    Déploiement de la stack complète LGTM sur la VM Coolify (UUID `rrw19kmye6gng961igtzqpgw`), avec agents Alloy systemd sur MS-01, VM Coolify, LXC NAS, LXC PBS et Raspberry Pi Kiosk. Intégration du monitoring SMART bare-metal et création de 4 dashboards exécutifs Grafana. Voir [Monitoring](/services/monitoring).
  </Accordion>

  <Accordion title="🔒 Mode SNAT Tailscale Bypass & CIS Docker Benchmark (ADR-009) — 19-20/08/2026">
    Désactivation de `userland-proxy` et exécution de `--snat-subnet-routes=false` pour la préservation absolue des IP sources réelles `100.64.0.x` sous Traefik. Voir [ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only).
  </Accordion>

  <Accordion title="⚡ Passthrough GPU Iris Xe & Tier Stockage SSD 4To — 18/08/2026">
    Passthrough iGPU Intel Iris Xe sur VM 104 pour transcodage QuickSync Jellyfin et bascule du tier chaud `/mnt/storage-hot` (Immich, Forgejo) sur le SSD 4To Samsung 870 EVO. Voir [IMS-NAS](/infrastructure/ims-nas).
  </Accordion>
</AccordionGroup>
