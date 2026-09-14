---
title: "Feuille de Route & Liste TODO"
description: "Suivi centralisé des chantiers prioritaires, roadmap de résilience et backlog d'évolution de l'infrastructure"
icon: "list-check"
iconType: "duotone"
last_reviewed: "2026-09-14"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Mis à Jour le 14/09/2026</Badge>

<Info>
  Cette page constitue le **journal central de suivi des chantiers et de la feuille de route** du homelab IMS-WORLD. Elle regroupe l'ensemble des tâches ouvertes classées par domaine d'intervention (Résilience, Sécurité, Supervision, Nouveaux Services, Matériel) ainsi que l'historique des jalons réalisés.
</Info>

---

## 📊 Matrice d'Avancement des Chantiers

| Domaine | Chantier | Priorité | Hôte Cible | Statut |
| --- | --- | --- | --- | --- |
| **Résilience & Quorum** | QDevice Corosync 3ᵉ vote | <Badge color="red">🔴 Priorité 1</Badge> | Raspberry Pi 3B\+ | ⏳ En attente |
| **Résilience & Alerting** | Supervision & Alerting hors-MS-01 | <Badge color="red">🔴 Priorité 1</Badge> | Mac Mini / RPi / VPS | ⏳ En attente |
| **Supervision & Métrologie** | Agent Alloy systemd sur Mac Mini | <Badge color="green">🟢 Effectué</Badge> | Mac Mini (`100.64.0.6`) | ✅ 24/08/2026 |
| **Supervision & Métrologie** | Agent Alloy sur le Worker Coolify LXC 105 | <Badge color="amber">🟡 Moyen Terme</Badge> | LXC 105 (`192.168.1.198`) | ⏳ En attente |
| **Alerting** | Notifications Ntfy sur échec backup | <Badge color="amber">🟡 Moyen Terme</Badge> | MS-01 / PBS | ⏳ En attente |
| **Sauvegardes & PRA** | Politique de sauvegarde PBS incluant le nœud Mac Mini (`pve-macmini`) | <Badge color="amber">🟡 Moyen Terme</Badge> | Cluster / PBS | ⏳ En attente |
| **Sauvegardes & Supervision** | Audit des sauvegardes des VM et validation du monitoring | <Badge color="red">🔴 Priorité 1</Badge> | Cluster / PBS / VM | ⏳ En attente |
| **Gouvernance & Procédures** | Revue intégrale et validation de toutes les procédures | <Badge color="red">🔴 Priorité 1</Badge> | Ensemble de la doc | ⏳ En attente |
| **Sécurité** | Credentials OVH dans fichier `.env` | <Badge color="amber">🟡 Moyen Terme</Badge> | VM 104 (Coolify Proxy) | ⏳ En attente |
| **Gestion des Accès** | Création des accès pour Elo (Tailscale + VM dédiée) | <Badge color="amber">🟡 Moyen Terme</Badge> | Headscale / Proxmox | ⏳ En attente |
| **IAM & Sécurité** | Connexion propre d'Authentik avec Tailscale (OIDC) | <Badge color="green">🟢 Effectué</Badge> | Authentik / Tailscale | ✅ 13/09/2026 |
| **Sécurité** | DMZ & Bastion SSH d'administration | <Badge color="blue">🟦 À Évaluer</Badge> | Réseau / DMZ | 💡 Étude |
| **Sécurité** | Firewall Proxmox VE 3 niveaux | <Badge color="amber">🟡 Moyen Terme</Badge> | MS-01 & Mac Mini | ⏳ En attente |
| **Sécurité** | Intel vPro / AMT (Gestion Out-of-Band) | <Badge color="blue">🟦 À Évaluer</Badge> | MS-01 Bare-Metal | 💡 Étude |
| **Docker & Hygiène** | Pinning des tags Docker (suppression `:latest`) | <Badge color="amber">🟡 Moyen Terme</Badge> | VM 104 (Coolify) | ⏳ En attente |
| **Docker & Hygiène** | Scan & Alertes mises à jour (Diun Ntfy) | <Badge color="amber">🟡 Moyen Terme</Badge> | VM 104 (Coolify) | ⏳ En attente |
| **Supervision** | Exporteur Prometheus Jellyfin | <Badge color="blue">🟦 Nouveaux Services</Badge> | VM 104 (Coolify) | ⏳ En attente |
| **UX & Proxy** | Pages d'erreur custom Traefik (404/502/503/504) | <Badge color="green">🟢 Effectué</Badge> | Traefik Proxy | ✅ 26/08/2026 |
| **Proxy & Ingress** | PoC & Évaluation Caddy v2 (xcaddy \+ CrowdSec \+ OVH) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify Proxy) | 💡 Étude / PoC |
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
| **IA & Automatisation** | Page Agent (Alibaba — Agent d'automation browser) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **IA & Productivity** | Meetilty (Gestion & transcription de réunions) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Plus tard |
| **Domotique & Services** | Finalisation et documentation de la VM Home Assistant (HA) | <Badge color="amber">🟡 Moyen Terme</Badge> | Proxmox VE | ⏳ En attente |
| **Automatisation & IaC** | Scripts Ansible de création automatique de VM | <Badge color="amber">🟡 Moyen Terme</Badge> | Proxmox VE (MS-01) | ⏳ En attente |
| **Automatisation & Web** | Portail web de génération de VM temporaires (self-service) | <Badge color="blue">🟦 À Évaluer</Badge> | VM 104 (Coolify) | 💡 Étude / PoC |
| **Automatisation & Ansible** | Playbook post-provisioning (users, sudo NOPASSWD, base) | <Badge color="amber">🟡 Moyen Terme</Badge> | VM & Nœuds PVE | ⏳ En attente |
| **Matériel & Rack** | Extension physique du rack Labrax 10" | <Badge color="amber">🟡 Moyen Terme</Badge> | Rack Physique | ⏳ En attente |
| **Stockage** | Extension capacitive HDD 4To / 8To Neuf | <Badge color="blue">🟦 À Évaluer</Badge> | NAS LXC 100 | 💡 Achat futur |
| **Sécurité** | Détection d'intrusions NIDS & Sentryx | <Badge color="blue">🟦 À Évaluer</Badge> | Réseau / VM 104 | 💡 Étude |

---

## 🔴 1. Chantiers Critiques & Haute Priorité (Quorum & Résilience)

### 1.1 🖥️ Quorum Cluster & QDevice Corosync (Raspberry Pi 3B\+) <Badge color="red">🔴 Priorité 1</Badge>

- **Constat** : Le cluster Proxmox VE `ims-cluster` comporte 2 nœuds (MS-01 et Mac Mini). Sans un troisième vote d'arbitrage, la perte de l'un des deux hôtes entraîne la perte de quorum sur le nœud survivant.
- **Tâche** : Déployer le démon **QDevice Corosync** (`corosync-qnetd` / port TCP `5403`) sur le Raspberry Pi 3B\+ (`ims-rpi-monitor`) pour accorder le 3ᵉ vote d'arbitrage et sécuriser le quorum en cas d'extinction de l'un des deux hyperviseurs.

### 1.2 🚨 Supervision & Alerting d'Uptime Hors-Hôte (Panne MS-01) <Badge color="red">🔴 Priorité 1</Badge>

- **Constat** : Toute la stack de supervision (Prometheus, Grafana, Loki, Uptime Kuma, Ntfy) est hébergée sur la VM 104 du MS-01. Si le MS-01 subit une panne matérielle ou électrique totale, aucune alerte ne peut être émise.
- **Tâche** : Déployer un mécanisme d'alerting léger hors-hôte (sur le Mac Mini, le Raspberry Pi ou un micro VPS externe) pour surveiller la joignabilité du MS-01 et envoyer une alerte push si l'hôte principal s'éteint.

### 1.3 🔔 Notifications Ntfy sur Échec des Sauvegardes Proxmox VE / PBS <Badge color="amber">🟡 Moyen Terme</Badge>

- **Tâche** : Configurer les cibles de notification natifs (_Notification Targets_) sur Proxmox VE (MS-01) et Proxmox Backup Server (LXC 103) pour émettre une alerte Ntfy automatique immédiate sur le topic `ims-alerts` en cas d'échec d'un job de sauvegarde `vzdump` ou `pbs`.

### 1.4 🛡️ Mise à Jour de la Politique de Sauvegarde PBS pour le Nœud Mac Mini (`pve-macmini`) <Badge color="amber">🟡 Moyen Terme</Badge>

- **Constat** : Suite à l'intégration du Mac Mini en tant que Nœud 2 actif du cluster `ims-cluster`, les tâches de sauvegarde planifiées de Proxmox Backup Server (PBS LXC 103) et `vzdump` doivent être révisées pour couvrir les VM/LXC hébergés ou migrés sur ce nœud.
- **Tâche** : Déclarer le nœud `pve-macmini` dans le Datacenter PVE (jobs de sauvegarde `all` / multi-nœuds) et ajuster la politique de rétention et d'exclusion (LXC 100 NAS maintenu en exclusion manuelle stop-only). Voir [Politique de Sauvegardes](/infrastructure/politique-sauvegardes).

### 1.5 💾 Audit des Sauvegardes de Toutes les VM et Validation du Monitoring <Badge color="red">🔴 Priorité 1</Badge>

- **Constat** : Les machines virtuelles hébergées sur le cluster requièrent une garantie opérationnelle sur l'état de leurs sauvegardes et leur observabilité en temps réel.
- **Tâche** : Vérifier la bonne exécution des jobs de sauvegarde PBS (`vzdump` / snapshot) pour l'ensemble des VM. Tester des restaurations à blanc. S'assurer que chaque VM remonte ses métriques de santé et ses alertes dans la stack de monitoring (Prometheus, Grafana Alloy, Uptime Kuma et notifications Ntfy).

### 1.6 📋 Revue intégrale et validation de toutes les procédures opérationnelles <Badge color="red">🔴 Priorité 1</Badge>

- **Constat** : Les procédures opérationnelles (urgences, déploiements, sécurité, PRA, gestion des accès et enrôlement réseau) doivent rester strictement conformes aux configurations réelles de l'infrastructure après les récentes évolutions techniques (cluster 2 nœuds, intégration Authentik OIDC, template VM 8000).
- **Tâche** : Réviser et éprouver pas-à-pas l'intégralité des procédures documentées pour valider leur exactitude opérationnelle :
  - Tester et valider chaque commande CLI, chemin de fichier et variable dynamique (`ips`, `domains`).
  - Confirmer les prérequis d'exécution, les ports réseau et les politiques d'accès Authentik / Headscale.
  - Actualiser l'attribut `last_reviewed` dans le frontmatter de chaque fiche validée.
  - Corriger sans délai les éventuelles divergences ou commandes obsolètes.

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

### 2.5 👤 Création des Accès pour Elo (Tailscale & VM Dédiée) <Badge color="amber">🟡 Moyen Terme</Badge>

- **Constat** : Un accès distant sécurisé et restreint doit être configuré pour Elo afin de lui permettre d'accéder à sa propre machine virtuelle.
- **Tâche** : Provisionner l'utilisateur et son poste dans Headscale / Tailscale avec des ACL dédiées. Configurer sa VM sur Proxmox VE et sécuriser ses accès distants (SSH / RDP / Web).

### 2.6 🔑 Connexion Propre d'Authentik avec Tailscale / Headscale <Badge color="green">🟢 Effectué le 13/09/2026</Badge>

- **Statut** : Authentik est configuré en tant que provider OIDC pour Headscale avec attribution des droits d'accès au groupe **`membres`**. Le premier flux d'enrôlement et l'inscription initiale d'un utilisateur ont été testés et validés avec succès via la mire SSO Authentik.
- **Tâche résiduelle** : Ajuster les ACL Headscale pour restreindre les accès aux machines autorisées selon les profils utilisateurs (voir chantier [2.5](#25--création-des-accès-pour-elo-tailscale--vm-dédiée)).

---

## 🐳 3. Supervision, Hygiène Docker & Métrologie

### 3.1 🏷️ Pinning des Tags Docker (Suppression des Tags `:latest`) <Badge color="amber">🟡 Moyen Terme</Badge>

- **Tâche** : Figer les images Docker sur des versions sémantiques précises (SemVer) pour Stirling PDF, Zipline et IT-Tools afin d'éliminer le risque de rupture accidentelle lors d'un `docker pull`.

### 3.2 📊 Extension de l'Agent Alloy sur le Mac Mini (`pve-macmini`) <Badge color="green">🟢 Effectué le 24/08/2026</Badge>

- **Statut** : Agent Grafana Alloy systemd réintégré avec succès sur le Mac Mini (`100.64.0.6`).
- **Composants** : Node Exporter (CPU, RAM, disque), collecteur SMART (`smartmon.sh` cron 5m sur SSD Apple 256 Go) et transmission des logs journald/syslog vers Loki (`10.10.10.2:3100`). Voir [Mac Mini](/infrastructure/mac-mini) et [Stack Monitoring](/services/monitoring).

### 3.3 📦 Scan & Alertes des Mises à Jour Docker (Diun) <Badge color="amber">🟡 Moyen Terme</Badge>

- **Tâche** : Déployer **Diun** (_Docker Image Update Notifier_) sur la VM Coolify pour surveiller les registres Docker et pousser une notification Webhook sur Ntfy dès qu'une version stable est publiée.

### 3.4 🎬 Exporteur Prometheus Dédié Jellyfin (`jellyfin-exporter`) <Badge color="blue">🟦 Nouveaux Services</Badge>

- **Tâche** : Déployer `jellyfin-exporter` sur la VM Coolify pour remonter à Grafana les métriques en temps réel des lectures actives (sessions transcodées vs direct play, débits, codecs).

### 3.5 🖼️ Pages d'Erreur & Indisponibilité Custom Traefik <Badge color="green">🟢 Effectué le 26/08/2026</Badge>

- **Statut** : Déploiement du conteneur helper Nginx (`vciwi7dolcl0hw1mffvjcfha`, `nginx:1.27-alpine`) et configuration du Dynamic File Provider `/data/coolify/proxy/dynamic/error-pages.yaml`.
- **Fonctionnalités** : Interception des 404 & 403 (masquage discret des 403 en 404), redirection dynamique `{status}` des erreurs 5xx et routeur `catchall-error-pages`. Voir [Traefik (Coolify Proxy)](/reseau/traefik-proxy#-gestion-globale-des-pages-derreur-custom-404-403-masqué--5xx).

### 3.6 ⚡ PoC & Évaluation de Caddy v2 (Remplacement Proxy Traefik v3) <Badge color="blue">🟦 À Évaluer — Étude PoC</Badge>

- **Constat** : Caddy v2 offre une empreinte mémoire très faible (~30 Mo RAM), une configuration synthétique via `Caddyfile` (éliminant la verbosité des labels Docker et la complexité des priorités File vs Docker Provider) et est proposé comme alternative officielle dans Coolify v4.
- **Tâche** : Réaliser un PoC de build d'une image custom Caddy (`xcaddy` avec plugins `caddy-dns/ovh` et `caddy-crowdsec-bouncer`), tester la réécriture du `Caddyfile` avec filtres `vpn-only` \+ `forward_auth` Authentik, et évaluer la bascule du proxy dans Coolify.

### 3.7 📊 Extension de l'Agent Alloy sur le Worker Coolify LXC 105 <Badge color="amber">🟡 Moyen Terme</Badge>

- **Constat** : Le conteneur LXC 105 (`pve-macmini-worker_LXC-105`) exécute désormais des applications Docker distantes sous l'orchestration du Master Coolify v4.3.14. Ses métriques Docker et ses logs doivent être centralisés.
- **Tâche** : Déployer un agent **Grafana Alloy** (systemd ou conteneur) sur le LXC 105 (`192.168.1.198`) pour remonter la télémétrie des conteneurs applicatifs hébergés vers Prometheus (`10.10.10.2:9090`) et Loki. Voir [LXC 105 Worker](/infrastructure/lxc-coolify-worker) et [Stack Monitoring](/services/monitoring).

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

### 4.12 🌐 Agent d'Automatisation & Navigation Web (Page Agent — Alibaba) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Évaluer et tester le déploiement de **Page Agent** (agent d'automatisation et de navigation web développé par Alibaba) sur la VM Coolify pour interagir avec des interfaces web complexes.

### 4.13 🎙️ Gestion & Transcription de Réunions (Meetilty) <Badge color="blue">🟦 À Évaluer — Plus tard</Badge>
- **Tâche** : Évaluer l'auto-hébergement de **Meetilty** pour organiser, synthétiser et générer les comptes-rendus et transcriptions de réunions.

### 4.14 🏠 Finalisation et Documentation de la VM Home Assistant (HA) <Badge color="amber">🟡 Moyen Terme</Badge>

- **Constat** : La machine virtuelle Home Assistant (HA OS) est en cours de mise en service sur Proxmox VE et requiert une stabilisation technique ainsi qu'une documentation complète.
- **Tâche** : Finaliser la configuration système, le réseau (bridge direct `vmbr0` pour mDNS/SSDP) et les intégrations domotiques de la VM. Rédiger la fiche de documentation dédiée (architecture, sauvegardes et procédures de maintenance).

---

## 🤖 5. Automatisation, IaC & Provisioning Proxmox / Ansible

### 5.1 🚀 Scripts Ansible de création automatique de VM Proxmox <Badge color="amber">🟡 Moyen Terme</Badge>

- **Constat** : Le clonage manuel d'une machine virtuelle depuis le template 8000 via l'hyperviseur Proxmox VE (`qm clone`, `qm set`, `qm start`) nécessite plusieurs actions en ligne de commande.
- **Tâche** : Développer une collection de playbooks et rôles Ansible dédiés à l'IaC Proxmox (`community.general.proxmox_kvm` ou appels API/CLI PVE) pour provisionner automatiquement une VM :
  - Cloner le template Ubuntu 24.04 (`VMID 8000`).
  - Définir le nom d'hôte, l'ID de VM, la RAM, les cœurs CPU et la taille du disque.
  - Injecter le snippet Cloud-Init `vendor-data.yaml` pour l'enrôlement Tailscale/Headscale automatique.
  - Démarrer la VM et attendre qu'elle soit joignable sur le réseau VPN.
  - Ajouter automatiquement le nouvel hôte à l'inventaire dynamique Ansible.

### 5.2 🌐 Portail web de génération de VM temporaires (self-service) <Badge color="blue">🟦 À Évaluer</Badge>

- **Constat** : Les sessions de test, validations de concepts (PoC) et environnements éphémères nécessitent une création rapide sans ouvrir l'interface Proxmox et risquent de consommer inutilement des ressources s'ils sont oubliés.
- **Tâche** : Concevoir et déployer une interface web légère (portail self-service connecté à l'API Proxmox et aux playbooks Ansible) :
  - Formulaire simple pour demander une VM temporaire (nom, gabarit CPU/RAM, utilisateur SSH).
  - Attribution d'un temps de rétention ou durée de vie (*Time-To-Live* / TTL : 2h, 24h, 7j).
  - Tâche planifiée automatique détruisant la VM et libérant le stockage et la RAM à l'échéance.
  - Authentification centralisée via le SSO Authentik de l'infrastructure.

### 5.3 👤 Configuration Ansible post-provisioning (utilisateurs, sudo NOPASSWD, services de base) <Badge color="amber">🟡 Moyen Terme</Badge>

- **Constat** : Après la première initialisation d'une VM, la standardisation des comptes d'administration, les privilèges sudo et l'outillage système de base doivent être appliqués de façon idempotente et reproductible.
- **Tâche** : Créer un rôle Ansible réutilisable de configuration initiale de nœud :
  - **Gestion des utilisateurs** : Création de comptes utilisateurs avec leur shell par défaut et déploiement de leurs clés SSH autorisées.
  - **Privilèges sudo sans mot de passe** : Déploiement propre d'une règle `NOPASSWD` ciblée dans `/etc/sudoers.d/` pour les administrateurs désignés.
  - **Services et paquets de base** : Installation de l'outillage standard (`curl`, `git`, `htop`, `tmux`, `jq`, `net-tools`, `qemu-guest-agent`).
  - **Options activables à la demande** : Activation modulaire par variables d'inventaire de composants optionnels (Docker Engine, agent Grafana Alloy, Fail2ban, Node Exporter).

---

## 🗄️ 6. Matériel, Physique & Stockage Long Terme

### 6.1 🛠️ Extension Rack Labrax 10" <Badge color="amber">🟡 Moyen Terme</Badge>

- **Tâche** : Achever le montage physique et le câblage propre de l'extension de châssis 10 pouces du rack [Labrax](/infrastructure/labrax).

### 6.2 💾 Achat HDD 4To / 8To Neuf <Badge color="blue">🟦 À Évaluer</Badge>

- **Tâche** : Acquérir un disque dur HDD supplémentaire pour étendre le pool capacitif et/ou instaurer un miroir de parité sur le NAS.

### 6.3 🔍 Sonde NIDS / Sentryx <Badge color="blue">🟦 À Évaluer</Badge>

- **Tâche** : Évaluer la mise en service d'une sonde de détection d'intrusions réseau (NIDS) et du projet Sentryx.

---

## 🟢 7. Chantiers Récents Effectués & Archives


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
