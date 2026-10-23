---
title: "Mac Mini (Nœud 2 Cluster ims-cluster)"
description: "Second nœud du cluster Proxmox VE — Hyperviseur physique de secours et support de basculement"
icon: "apple"
iconType: "duotone"
last_reviewed: "2026-10-23"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active (Nœud 2 Cluster)</Badge>

<Info>
Le **Mac Mini 2012** (Macmini6,1/6,2) constitue le **second nœud d'hypervision physique** du cluster Proxmox VE (`ims-cluster`). Il apporte de la tolérance de panne et héberge les conteneurs de haute disponibilité en cas de maintenance du nœud principal MS-01.
</Info>

---

## Fiche Technique & Spécifications Matérielles

| Propriété | Valeur |
|---|---|
| **Modèle Matériel** | Apple Mac Mini (Late 2012 — Macmini6,1/6,2) |
| **Processeur (CPU)** | Intel Core i5 Dual-Core (4 threads) |
| **Mémoire RAM** | **16 Go** DDR3 |
| **Stockage Système** | SSD SATA Apple `SM0256F` (256 Go) |
| **Carte Réseau** | Gigabit Ethernet Broadcom `tg3` (Support Linux natif) |
| **OS / Hyperviseur** | **Proxmox VE 9.2.11** (Noyau 6.8+ Debian 13 Trixie) |
| **Cluster Proxmox** | Membre actif du cluster **`ims-cluster`** (2/2 votes, Quorate: Yes) |
| **Hostname FQDN** | `pve-macmini.ims-world.fr` |
| **IP LAN Native** | `192.168.1.42` |
| **IP Tailscale VPN** | {ips.macmini} (`pve-macmini`) |
| **Compte Admin SSH** | Clé uniquement (`PasswordAuthentication no`, `PermitRootLogin prohibit-password`, `cmolotkoff` avec `sudo` journalisé `log_subcmds`, Port 22) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Installation & Déploiement Proxmox VE

### 1. Caractéristiques de l'Installation
- **Architecture EFI Native** : Pas de puce de sécurité Apple T2, démarrage EFI natif sans patch de noyau nécessaire.
- **Dépôts Proxmox** : Suppression des dépôts payants `pve-enterprise.sources` et `ceph.sources` (format deb822 Debian Trixie `.sources`) et activation du dépôt gratuit `pve-no-subscription`.
- **Mise à Jour** : Alignement strict sur la version **Proxmox VE 9.2.11** (identique au MS-01). Mises à jour maintenues manuelles (aucun redémarrage ou update automatique sur les hyperviseurs).

### 2. Intégration au Cluster (`ims-cluster`)
- Le Mac Mini a rejoint le cluster Proxmox VE créé sur le MS-01 via la commande `pvecm add`.
- **Partage de Configuration Cluster** : Synchronisation automatique des comptes utilisateur (`/etc/pve/user.cfg`), des rôles RBAC (`cmolotkoff@pam`) et des clés SSH autorisées entre les deux hyperviseurs.

---

## 🛡️ Sécurité & Protection Host (Fail2ban, SSH & Ntfy)

Le socle SSH standardisé `/etc/ssh/sshd_config.d/00-ims-ops.conf` et **Fail2ban** (`fail2ban.service`) protègent le nœud `pve-macmini` :
- **SSH Durci** : Connexions par mot de passe désactivées (`PasswordAuthentication no`), X11 désactivé, `PermitRootLogin prohibit-password` pour autoriser le root par clé SSH entre nœuds du cluster. Clé orpheline `audit@ims-world` retirée.
- **Fail2ban avec escalade** : Configuration `/etc/fail2ban/jail.local` avec prisons `sshd` et `recidive`. Alertes transmises à Ntfy directement via l'IP LAN de Traefik (`192.168.1.52`).

---

## 📊 Supervision & Agent Alloy (Stack LGTM)

Le Mac Mini héberge un agent **Grafana Alloy systemd** (`macmini-alloy`) déployé à l'identique du pattern bare-metal du MS-01 :
- **Node Exporter & CPU/RAM/Disque** : Collecte des métriques bare-metal du système Proxmox VE 9.2.11.
- **Collecteur SMART (`smartmon.sh`)** : Tâche cron 5m (`/etc/cron.d/smartmon`) pour le suivi d'usure du SSD interne Apple 256 Go (`ata-APPLE_SSD_SM0256F`).
- **Logs Système Loki** : Centralisation des journald/syslog vers Loki (`10.10.10.2:3100`). Voir [Stack Monitoring](/services/monitoring).

---

## 🐳 Conteneurs & Workloads Hébergés

Le Nœud 2 `pve-macmini` héberge des workloads d'exécution isolés du cluster :

| ID / Hostname | Type | Distribution / OS | Adresse IP | Role & Description | Statut |
|---|---|---|---|---|---|
| **LXC 105** (`pve-macmini-worker_LXC-105`) | LXC Unprivileged | Debian 13 (Trixie) | `192.168.1.198` | **Coolify Worker** — Moteur d'exécution Docker distant (voir [LXC 105](/infrastructure/lxc-coolify-worker)) | <Badge color="green">🟢 Actif</Badge> |
| **VM 106** (`haos18.2`) | VM KVM Dédiée | Home Assistant OS 18.2 | `192.168.1.92` / `100.64.0.5` | **Home Assistant** — Serveur domotique et passerelle IoT locale (voir [Home Assistant](/infrastructure/vm-home-assistant)) | <Badge color="green">🟢 Actif</Badge> |

---

## Rôle & Prochaines Étapes

- **Support de Basculement** : Utilisé comme nœud récepteur pour la migration à chaud/à froid de conteneurs et VM.
- **Feuille de Route** : Intégration de la VM 106 dans les sauvegardes PBS et déploiement du **QDevice Corosync** sur le Raspberry Pi pour sécuriser le quorum.

---

<CardGroup cols={2}>
  <Card title="Hyperviseur Principal (MS-01)" icon="server" href="/infrastructure/proxmox-host">
    Fiche technique et administration du nœud 1 de ims-cluster.
  </Card>
  <Card title="Politique de Sauvegarde" icon="shield-check" href="/procedures/politique-sauvegardes">
    Sauvegardes PBS et protection des VM/LXC du cluster.
  </Card>
</CardGroup>
