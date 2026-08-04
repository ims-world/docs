---
title: "Glossaire & Lexique Technique"
description: "Définitions des concepts, sigles et technologies de l'infrastructure homelab IMS-WORLD"
icon: "book-bookmark"
iconType: "duotone"
---

<Info>
Ce glossaire rassemble et détaille l'ensemble des termes techniques, acronymes et choix d'architecture utilisés à travers la documentation du homelab IMS-WORLD.
</Info>

## 🖥️ Virtualisation & Conteneurisation

| Terme | Définition & Contexte Homelab |
|---|---|
| **LXC (LXC Container)** | Conteneur d'OS virtuel Linux partageant directement le noyau (*kernel*) du serveur hôte Proxmox VE. Très économe en RAM et CPU. Utilisé pour **IMS-NAS (100)** et **IMS-PBS (103)**. |
| **LXC Privilégié** | Conteneur LXC exécuté avec les privilèges root de l'hôte (UID/GID non décalés). Nécessaire sur IMS-NAS et IMS-PBS pour permettre les montages NFS et l'accès direct aux points de montage FUSE. |
| **VM QEMU/KVM** | Machine Virtuelle avec isolation matérielle et noyau propre complet (ex: **IMS-Coolify VM 104** sous Ubuntu 24.04). Garantit une étanchéité totale pour le moteur Docker et Traefik. |
| **Pass-through `mp0`** | Montage direct d'un dossier de l'hôte Proxmox dans un conteneur LXC via le fichier de configuration `/etc/pve/lxc/<vmid>.conf`. |

## 💾 Stockage & Système de Fichiers

| Terme | Définition & Contexte Homelab |
|---|---|
| **FUSE MergerFS** | Système de fichiers d'union (FUSE) permettant d'unifier plusieurs disques physiques ou dossiers sous un point de montage unique (`/mnt/storage`). Permet l'extension de stockage sans RAID dur. |
| **Option `inodecalc=path-hash`** | Option d'optimisation MergerFS calculant les inœuds par hachage de chemin. Essentielle pour garantir des identifiants stables lors des partages NFS long terme. |
| **`storage-hot`** | Espace de stockage haute performance dédié aux bases de données et index applicatifs (Immich, Forgejo, Authentik). Destiné à basculer sur SSD en Phase 4. |
| **Datastore PBS** | Dossier de stockage dédupliqué géré par Proxmox Backup Server (`/mnt/pbs-datastore`), découpant les sauvegardes des VM/LXC en morceaux (*chunks*) chiffrés et dédupliqués. |
| **`vzdump`** | Outil de sauvegarde natif de Proxmox VE générant des archives compressées (`.vma.zst`) du système entier d'un LXC ou d'une VM. |

## 🌐 Réseau & Sécurité

| Terme | Définition & Contexte Homelab |
|---|---|
| **Challenge DNS-01 (ACME)** | Méthode d'obtention de certificats SSL Let's Encrypt valides en créant dynamiquement un enregistrement TXT chez l'hébergeur DNS (OVH). Permet de certifier des domaines et sous-domaines non exposés publiquement. |
| **Tailnet Overlay** | Réseau virtuel privé mesh (VPN) basé sur WireGuard et géré par le control plane **Headscale**. Permet l'interconnexion sécurisée des appareils distants sur le sous-réseau `100.64.0.0/10`. |
| **Protocol Noise** | Protocole d'échange de clés cryptographiques utilisé par Tailscale / Headscale pour authentifier les machines du VPN sans serveur centralisé lisant le trafic. |
| **Middleware `vpn-only`** | Règle de filtrage IP configurée dans Traefik restriction l'accès HTTPS aux seules adresses IPs clientes appartenant au sous-réseau Tailscale (`100.64.0.0/10`). |
| **Tarpit SSH (*Endlessh*)** | Service leurre écoutant sur le port 22 standard. Envoie une bannière SSH infinie et ultra-lente pour piéger et paralyser les robots de scan automatisés. |

## ⚡ Matériel & Alimentation

| Terme | Définition & Contexte Homelab |
|---|---|
| **Rack Labrax 3D** | Structure de rack 10 pouces imprimée en 3D (modèle MakerWorld Révision Core IMS-01) hébergeant l'ensemble des équipements informatiques. |
| **PicoPSU-160-XT** | Mini-alimentation ATX haute efficacité recevant du courant continu 12V DC externe et alimentant les disques durs SATA et accessoires du rack. |
| **Module 2U Raspberry Pi** | Panneau d'intégration 2U imprimé en 3D hébergeant le Raspberry Pi 3B+, l'écran LCD de statut et les ports Keystones RJ45 en façade. |
