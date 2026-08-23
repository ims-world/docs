---
title: "ADR-010 : Maintien du Port SSH 22 et du Hostname 'pve' en Cluster"
description: "Décision d'architecture renonçant à la migration du port SSH 22 et au renommage du hostname MS-01 pour préserver la stabilité du cluster Proxmox VE"
icon: "bookmark"
iconType: "duotone"
last_reviewed: "2026-08-23"
---

<Badge color="green">🟢 Décision Acceptée & Implémentée</Badge>

## Contexte & Enjeu

Lors du chantier de sécurisation de l'infrastructure et de l'intégration du **Mac Mini** au cluster Proxmox VE (`ims-cluster`), deux modifications d'infrastructures avaient été envisagées :

1. **Renommage du Hostname du Nœud Principal MS-01** : Passer de `pve` à `pve-ms01` pour des raisons d'harmonisation cosmétique avec `pve-macmini`.
2. **Migration du Port SSH Système** : Passer le port SSH d'administration système du port **`22`** standard vers le port **`4242`**.

---

## Analyse des Risques & Impact Technique

### 1. Risque lié au Renommage de Hostname (`pve` ➔ `pve-ms01`)
Dans un cluster Proxmox VE actif avec Corosync et le système de fichiers distribué PMXCFS (`/etc/pve/`), le nom d'hôte est profondément imbriqué dans :
- La structure des répertoires de configuration des VM/LXC (`/etc/pve/nodes/<hostname>/`).
- Le fichier de configuration du quorum Corosync (`/etc/pve/corosync.conf`).
- La base RRDcached et les certificats TLS de nœud.

La documentation officielle Proxmox déconseille formellement le renommage à chaud d'un nœud de cluster. Plusieurs cas de rupture totale de quorum et d'orphelinat de VM sont recensés sur les forums officiels Proxmox.

### 2. Risque lié à la Migration du Port SSH (22 ➔ 4242)
Les nœuds du cluster Proxmox VE s'appuient sur SSH root direct (`/root/.ssh/authorized_keys`) pour :
- Les réplications et migrations de conteneurs/VM à chaud entre nœuds.
- La console Web Shell distante et les commandes d'administration `pvecm`.

Modifier le port SSH par défaut exigerait de surcharger la configuration SSH inter-nœuds (`/root/.ssh/config`) sur l'ensemble du cluster. De plus, la politique de sécurité réseau IMS-WORLD applique déjà un **blocage strict au niveau du routeur Bbox** (0 port forwarding SSH vers le WAN). Le SSH d'administration n'est joignable que via le **LAN domestique (`192.168.1.0/24`)** ou le **VPN Overlay Tailscale (`100.64.0.0/10`)**.

---

## Décisions Actées

1. **Maintien du Hostname `pve`** sur l'hyperviseur MS-01. Aucun renommage à chaud ne sera effectué. L'harmonisation éventuelle sera repoussée à une réinstallation complète future si nécessaire.
2. **Maintien du Port SSH 22 Standard** sur l'ensemble des nœuds du cluster (MS-01 et Mac Mini) ainsi que sur les VM Linux.
3. **Sécurisation Alternative** : La protection de l'accès SSH repose intégralement sur :
   - L'isolation réseau stricte (0 exposition WAN Bbox).
   - L'authentification par clés SSH cryptographiques Ed25519 (mots de passe désactivés).
   - La pile **Fail2ban harmonisée sur 3 hôtes** avec escalade de ban et notifications Ntfy en direct.

---

## Conséquences

- **Positif** : Stabilité maximale du cluster Proxmox VE `ims-cluster`, maintenance simplifiée des migrations de VM/LXC inter-nœuds sans surcharge de scripts SSH custom.
- **Positif** : Alignement strict sur les recommandations d'architecture Proxmox VE.
