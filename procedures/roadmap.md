---
title: "Feuille de Route & Liste TODO"
description: "Chantiers techniques prioritaires, roadmap de résilience et backlog d'évolution de l'infrastructure"
icon: "list-check"
iconType: "duotone"
last_reviewed: "2026-08-23"
---

import { ips, domains } from "/snippets/variables.mdx";

<Info>
Cette page recense l'ensemble de la **feuille de route technique et de la liste TODO** d'évolution de l'infrastructure homelab IMS-WORLD (sécurité, résilience, matériel et nouveaux services).
</Info>

---

## 🔴 1. Chantiers Critiques & Haute Priorité (Résilience & Quorum)

### 1.1 🖥️ Quorum Cluster & QDevice Corosync (Raspberry Pi 3B+)
- **Constat** : Le cluster Proxmox VE `ims-cluster` comporte 2 nœuds (MS-01 et Mac Mini). Sans un troisième vote de quorum, la panne de l'un des deux hôtes entraîne la perte de quorum sur le nœud survivant.
- **Tâche** : Déployer le démon **QDevice Corosync** (`corosync-qnetd` / port TCP `5403`) sur le Raspberry Pi 3B+ (`ims-rpi-monitor`) pour accorder le 3ᵉ vote d'arbitrage et sécuriser le quorum en cas de coupure de l'un des deux hyperviseurs.

### 1.2 🚨 Résilience & Monitoring d'Uptime Hors-Hôte (Panne MS-01)
- **Constat** : Toute la stack de supervision (Prometheus, Grafana, Loki, Uptime Kuma, Ntfy) est actuellement hébergée sur la VM 104 de l'hôte MS-01. Si le MS-01 subit une coupure électrique ou matérielle totale, aucune alerte ne peut être émise.
- **Tâche** : Déployer un mécanisme d'alerting léger hors-hôte (sur le Mac Mini, le Raspberry Pi ou un micro VPS externe) pour surveiller l'état du MS-01 et émettre une alerte push si l'hyperviseur principal s'éteint.

### 1.3 🔔 Notifications Ntfy sur Échec des Sauvegardes Proxmox VE / PBS
- **Tâche** : Configurer les cibles de notification natifs (*Notification Targets*) sur Proxmox VE (MS-01) et Proxmox Backup Server (LXC 103) pour envoyer une alerte Ntfy automatique immédiate sur le topic `ims-alerts` en cas d'échec d'un job de sauvegarde `vzdump` ou `pbs`.

---

## 🔒 2. Sécurité & Durcissement Système

### 2.1 🔑 Masquage des Credentials OVH du Proxy Traefik
- **Tâche** : Extraire les identifiants d'API OVH (challenge DNS-01 Let's Encrypt) du fichier `/data/coolify/proxy/docker-compose.yml` et les basculer dans un fichier `.env` restreint (`chmod 600`).

### 2.2 🏰 Exploration d'une DMZ & Bastion SSH
- **Tâche** : Évaluer et concevoir l'architecture d'un bastion d'administration SSH isolé en DMZ pour verrouiller et auditer les accès système distants.

### 2.3 🛡️ Activation du Firewall Proxmox VE 3 Niveaux
- **Tâche** : Configurer le pare-feu natif de Proxmox VE (niveau Nœud ➔ Datacenter ➔ Guest) sur le MS-01 et le Mac Mini, en définissant des règles strictes sur les bridges `vmbr0` et `vmbr1`.

### 2.4 ⚙️ Configuration & Gestion Out-of-Band (Intel vPro / AMT)
- **Tâche** : Configurer la technologie d'aménagement à distance Intel vPro / AMT sur le Minisforum MS-01 pour permettre la prise de main KVM matérielle bas niveau même lorsque le système d'exploitation est éteint.

---

## 🐳 3. Hygiène Docker, Services & Monitoring

### 3.1 🏷️ Pinning des Tags Docker (Suppression des Tags `:latest`)
- **Constat** : Certaines applications utilisent le tag générique `:latest` (Stirling PDF, Zipline).
- **Tâche** : Figer l'intégralité des images Docker sur des versions sémantiques précises (SemVer) pour éviter toute rupture imprévue lors des redémarrages.

### 3.2 📦 Scan & Alertes des Mises à Jour Docker (Diun)
- **Tâche** : Déployer **Diun** (*Docker Image Update Notifier*) sur la VM Coolify pour surveiller les registres Docker et émettre une alerte Webhook instantanée sur Ntfy dès qu'une version stable est publiée.

### 3.3 🎬 Exporteur Prometheus Dédié Jellyfin (`jellyfin-exporter`)
- **Tâche** : Déployer un conteneur exporteur Prometheus dédié à Jellyfin sur la VM Coolify pour alimenter Grafana avec les métriques en temps réel des lectures actives (transcodage vs direct play, débits).

### 3.4 🖼️ Pages d'Erreur & Indisponibilité Custom sur Traefik
- **Tâche** : Configurer des middlewares de gestion d'erreur Traefik (`errors`) pour servir des pages d'erreur HTML/CSS personnalisées aux couleurs de la marque en cas de 404 (Introuvable) ou 502/503/504 (Service en maintenance).

---

## 🆕 4. Nouveaux Services & Projets d'Évolution

### 4.1 🛡️ Serveur DNS Secondaire (AdGuard Home / Pi-hole)
- **Tâche** : Déployer une instance secondaire AdGuard Home ou Pi-hole (sur le Mac Mini ou le Raspberry Pi) pour fournir une résolution DNS locale redondante avec filtrage publicitaire et protection anti-tracking.

### 4.2 🖼️ Service d'Optimisation & Compression d'Images (Imgcompress)
- **Tâche** : Déployer une solution d'optimisation et de compression d'images automatisée (Imgcompress ou équivalent) pour réduire la taille des visuels avant intégration sur les plateformes.

### 4.3 💾 Migration du Stockage Zipline vers le Tier SSD `storage-hot`
- **Tâche** : Basculer le dossier d'assets et d'envois temporaires de Zipline depuis le stockage HDD principal vers le SSD 4To Samsung 870 EVO (`/mnt/storage-hot`) pour accélérer le traitement des uploads ShareX.

### 4.4 🐙 Synchronisation des Credentials Miroirs GitHub (Forgejo)
- **Tâche** : Configurer les jetons d'accès et identifiants de synchronisation automatique sur les 6 dépôts miroirs GitHub hébergés sur l'instance Forgejo.

---

## 🗄️ 5. Matériel, Physique & Stockage Long Terme

### 5.1 🛠️ Finalisation de l'Extension du Rack Labrax 10"
- **Tâche** : Achever le montage physique et le câblage propre de l'extension de châssis 10 pouces du rack [Labrax](/infrastructure/labrax).

### 5.2 💾 Achat & Extension Capacitive NAS (HDD 4To / 8To Neuf)
- **Tâche** : Acquérir et installer un disque dur HDD supplémentaire pour étendre le pool de stockage capacitif et/ou mettre en place un miroir de parité sur le NAS.

### 5.3 🔍 Surveillance d'Intrusion Réseau (NIDS / Sentryx)
- **Tâche** : Évaluer l'intégration d'une sonde de détection d'intrusions réseau (NIDS) et la mise en service du projet Sentryx.

---

## 🟢 6. Chantiers Récents Effectués & Archivés

<AccordionGroup>
  <Accordion title="📊 Stack Monitoring LGTM (Grafana / Loki / Prometheus / Alloy) — 10-21/08/2026">
    Déploiement de la stack complète LGTM sur la VM Coolify (UUID `rrw19kmye6gng961igtzqpgw`), avec agents Alloy systemd sur MS-01, VM Coolify, LXC NAS, LXC PBS et Raspberry Pi Kiosk. Intégration du monitoring SMART bare-metal et création de 4 dashboards exécutifs Grafana.
  </Accordion>

  <Accordion title="🛡️ Détection d'Intrusions CrowdSec v1.7.8 & WAF AppSec — 22/08/2026">
    Déploiement de l'agent CrowdSec, du plugin bouncer Traefik (mode stream fail-open `updateMaxFailure: -1`), du WAF AppSec (196 règles inband), des allowlists `tailscale`/`home-lan` et de l'IHM locale **Shield** (`shield.ims-world.fr`).
  </Accordion>

  <Accordion title="🖥️ Création du Cluster Proxmox VE ims-cluster & Fail2ban — 23/08/2026">
    Installation de Proxmox VE 9.2.11 sur Mac Mini, création du cluster à 2 nœuds `ims-cluster` (Quorum 2/2 votes), formalisation de l'ADR-010 et déploiement harmonisé de Fail2ban avec alertes Ntfy sur 3 hôtes.
  </Accordion>

  <Accordion title="🔒 Mode SNAT Tailscale Bypass & CIS Docker Benchmark (ADR-009) — 19-20/08/2026">
    Désactivation de `userland-proxy` et exécution de `--snat-subnet-routes=false` pour la préservation absolue des IP sources réelles `100.64.0.x` sous Traefik.
  </Accordion>

  <Accordion title="⚡ Passthrough GPU Iris Xe & Tier Stockage SSD 4To — 18/08/2026">
    Passthrough iGPU Intel Iris Xe sur VM 104 pour transcodage QuickSync Jellyfin et bascule du tier chaud `/mnt/storage-hot` (Immich, Forgejo) sur le SSD 4To Samsung 870 EVO.
  </Accordion>
</AccordionGroup>
