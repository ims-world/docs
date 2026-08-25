---
title: "Mac Mini (Nœud 2 Cluster ims-cluster)"
description: "Second nœud du cluster Proxmox VE — Hyperviseur physique de secours et support de basculement"
icon: "apple"
iconType: "duotone"
last_reviewed: "2026-08-23"
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
| **Compte Admin SSH** | `cmolotkoff` (Clés SSH Ed25519, `sudo` NOPASSWD, Port 22) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Installation & Déploiement Proxmox VE

### 1. Caractéristiques de l'Installation
- **Architecture EFI Native** : Pas de puce de sécurité Apple T2, démarrage EFI natif sans patch de noyau nécessaire.
- **Dépôts Proxmox** : Suppression des dépôts payants `pve-enterprise.sources` et `ceph.sources` (format deb822 Debian Trixie `.sources`) et activation du dépôt gratuit `pve-no-subscription`.
- **Mise à Jour** : Alignement strict sur la version **Proxmox VE 9.2.11** (identique au MS-01).

### 2. Intégration au Cluster (`ims-cluster`)
- Le Mac Mini a rejoint le cluster Proxmox VE créé sur le MS-01 via la commande `pvecm add`.
- **Partage de Configuration Cluster** : Synchronisation automatique des comptes utilisateur (`/etc/pve/user.cfg`), des rôles RBAC (`cmolotkoff@pam`) et des clés SSH autorisées entre les deux hyperviseurs.

---

## 🛡️ Sécurité & Protection Host (Fail2ban & Ntfy)

Comme sur le host MS-01 et la VM Coolify, un service **Fail2ban** (`fail2ban.service`) local a été déployé et durci dans `/etc/fail2ban/jail.local` pour intercepter les tentatives d'intrusion SSH :

```ini
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5

bantime.increment = true
bantime.factor    = 2
bantime.maxtime   = 1w

action = %(action_)s
         ntfy

[recidive]
enabled   = true
mode      = normal
banaction = %(banaction_allports)s
bantime   = 1w
findtime  = 1d
maxretry  = 3
```

- **Politique d'Escalade** : Durée de bannissement progressive (1h ➔ 2h ➔ 4h... jusqu'à 1 semaine).
- **Prison Récidivistes (`recidive`)** : 3 bannes en 24h entraînent un bannissement d'une semaine sur tous les ports.
- **Alertes Ntfy** : Chaque bannissement déclenche une alerte instantanée vers le topic `ims-alerts` via l'action `/etc/fail2ban/action.d/ntfy.conf` avec un jeton d'accès scopé.

---

## 📊 Supervision & Agent Alloy (Stack LGTM)

Le Mac Mini héberge un agent **Grafana Alloy systemd** (`macmini-alloy`) déployé à l'identique du pattern bare-metal du MS-01 :
- **Node Exporter & CPU/RAM/Disque** : Collecte des métriques bare-metal du système Proxmox VE 9.2.11.
- **Collecteur SMART (`smartmon.sh`)** : Tâche cron 5m (`/etc/cron.d/smartmon`) pour le suivi d'usure du SSD interne Apple 256 Go (`ata-APPLE_SSD_SM0256F`).
- **Logs Système Loki** : Centralisation des journald/syslog vers Loki (`10.10.10.2:3100`). Voir [Stack Monitoring](/services/monitoring).

---

## Rôle & Prochaines Étapes

- **Support de Basculement** : Utilisé comme nœud récepteur pour la migration à chaud/à froid de conteneurs et VM.
- **Feuille de Route** : Installation du conteneur **LXC Home Assistant** (mode bridge direct `vmbr0` pour la découverte mDNS/SSDP) et déploiement du **QDevice Corosync** sur le Raspberry Pi pour sécuriser le quorum.

---

<CardGroup cols={2}>
  <Card title="Hyperviseur Principal (MS-01)" icon="server" href="/infrastructure/proxmox-host">
    Fiche technique et administration du nœud 1 de ims-cluster.
  </Card>
  <Card title="Politique de Sauvegarde" icon="shield-check" href="/infrastructure/politique-sauvegardes">
    Sauvegardes PBS et protection des VM/LXC du cluster.
  </Card>
</CardGroup>
