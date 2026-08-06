---
title: "Changelog"
description: "Chronologie du projet de migration Mac Mini → MS-01"
---

## Semaine du 06/08/2026 — Audit sécurité `vpn-only` validé

### 🔧 Améliorations

- **Étanchéité `vpn-only` confirmée sur la stack HomeFlix** — L'audit du middleware `vpn-only` est terminé : qBittorrent, Radarr, Sonarr et Prowlarr sont accessibles uniquement depuis le tailnet (`100.64.0.0/10`) et renvoient `403 Forbidden` depuis le WAN public. Concrètement, ces interfaces d'administration restent joignables via le VPN sans risque d'exposition publique. Voir [Traefik Proxy](/reseau/traefik-proxy) et [Matrice de Sécurité](/reseau/matrice-securite-exposition).
- **Roadmap mise à jour** — L'étape « Audit de Sécurité Middleware `vpn-only` » est marquée comme effectuée. Voir [Roadmap](/procedures/roadmap).

Le reste de l'infrastructure et des services publics (Authentik, Vaultwarden, HomeFlix, Headscale/Headplane) tourne en état stable depuis les livraisons de la semaine précédente.

## Semaine du 05/08/2026 — Transcodage matériel HomeFlix

### 🆕 Nouveautés

- **Transcodage matériel Jellyfin (QuickSync)** — L'iGPU Intel Iris Xe du MS-01 est désormais attribuée en passthrough PCIe à la VM Coolify. HomeFlix (Jellyfin) accélère le transcodage H.264, HEVC et AV1 en matériel, ce qui réduit fortement la charge CPU pendant la lecture et permet plus de flux simultanés. Voir [Host Proxmox](/infrastructure/proxmox-host#gpu-igpu-iris-xe-passthrough-vm-coolify) et [HomeFlix](/services/homeflix).

### 🔧 Améliorations

- **Roadmap mise à jour** — L'étape « Passthrough GPU » est marquée comme effectuée sur la roadmap du projet. Voir [Roadmap](/procedures/roadmap).

## Semaine du 04/08/2026 — Résilience matérielle & monitoring hors-bande

### 🆕 Nouveautés

- **Afficheur Kiosk Raspberry Pi 3B+** — Un Raspberry Pi 3B+ dédié pilote l'affichage physique du rack Labrax via un écran principal Wisecoco 7.84" (1280x400) et un second écran OLED 0.91" (128x32) encastrés dans un module 2U 3D. Un navigateur headless fait tourner en boucle la bannière IMS et des pages web en mode lisibilité, contrôlé par un bouton poussoir GPIO (appui court = change source, long 3s = éteint). Voir [Raspberry Pi 3B+ & Écrans](/infrastructure/rpi-monitor).
- **Désinstallation définitive de Cap** — Abandon de l'expérimentation du service d'enregistrement d'écran Cap, retiré du homelab. Voir [Cap (Service Supprimé)](/services/cap).

### 🔧 Améliorations

- **Module 3D imprimable pour le Raspberry Pi** — La fiche du Raspberry Pi 3B+ référence le modèle MakerWorld « Screen module for 10-inch rack — Raspberry Pi 2U », avec ses rendus CAD 3D (vue de façade et vue isométrique intérieure). Voir [Raspberry Pi 3B+ & Écrans](/infrastructure/rpi-monitor).
- **Tarpit SSH sur le Mac Mini** — Le port 22 du Mac Mini pointe désormais vers Endlessh (tarpit anti-bot) pour piéger les scans automatisés. L'accès SSH légitime passe par le port `4242`.
- **Résolution DNS de l'ancien Coolify** — L'ancienne instance Coolify du Mac Mini reste joignable via `coolify-old.ims-world.fr` grâce à un enregistrement DNS dans Headscale, le temps de la phase de validation post-cutover.
- **Matrice de sécurité interactive** — La cartographie des zones de confiance (WAN public, LAN, tailnet, admin) est désormais présentée sous forme d'onglets navigables avec cartes cliquables vers chaque service et sa politique d'exposition. Voir [Matrice de Sécurité & d'Exposition](/reseau/matrice-securite-exposition).
- **Arbre de décision de dépannage** — La page de dépannage inclut désormais un diagramme interactif qui guide symptôme par symptôme vers la cause probable et sa solution. Voir [Dépannage courant](/procedures/depannage-courant).

## Semaine du 03/08/2026 — Stabilisation post-cutover

### 🆕 Nouveautés

- **VPN self-hosted disponible** — L'accès distant au tailnet passe désormais par notre propre control plane, avec une interface d'administration web pour gérer utilisateurs, clés et appareils. Voir [Headscale + Headplane](/services/headscale-headplane).
- **Portail de demandes HomeFlix** — Jellyseerr est en ligne sur `videoclub.ims-world.fr` pour demander films et séries directement, en complément du streaming sur `homeflix.ims-world.fr`. Voir [HomeFlix](/services/homeflix).

### 🔧 Améliorations

- **Traefik mis à jour en v3.7** — Le reverse proxy public a été passé de la v3.6.23 à la v3.7 pendant le cutover, sans interruption perceptible côté utilisateur.
- **Certificats renouvelés automatiquement** — Le renouvellement des certificats HTTPS est de nouveau fonctionnel sur tous les domaines publics après la mise en place du challenge DNS-01. Plus d'avertissements de certificats expirés.
- **SSO Authentik consolidé** — L'identité centrale (`auth.ims-world.fr`) est désormais servie depuis la nouvelle infrastructure, avec le branding par domaine préservé pour chaque service.

### 🐛 Corrections

- **HomeFlix : téléchargements qBittorrent** — Les erreurs d'accès à l'interface qBittorrent derrière le proxy sont corrigées ; le WebUI est de nouveau accessible normalement.
- **Headscale : reconnexion des appareils** — La boucle de reconnexion OIDC constatée en début de bascule est résolue ; tous les appareils du tailnet se reconnectent sans intervention.
## 🎉 02/08/2026 — Cutover complet

Les 4 services essentiels (Authentik, Vaultwarden, HomeFlix, Headscale/Headplane) sont basculés en production sur le MS-01. Port-forward Bbox corrigé et basculé. Découverte structurelle majeure : le port-forward route tout le trafic public d'un coup, pas de bascule partielle possible (voir [Traefik Proxy](/reseau/traefik-proxy)).

Cascade de 8 blocages résolus dans la même session : accès console (Chrome), SSH manquant, label réseau Traefik manquant, port-forward mal ciblé, cache DNS transitoire, crash-loop OIDC Headscale, certificats DNS-01 en cours de négociation, warning cosmétique Coolify.

Mise à jour Traefik v3.6.23 → v3.7 effectuée en avance sur le planning, sans incident réel.

## 01/08/2026 — Préparation Headscale (Phases A-C)

Migration de données complète (config, `noise_private.key`, base SQLite) — tous les utilisateurs et appareils confirmés présents. Décision actée de ne pas tester via sous-domaine `-ng` (le `server_url` de Headscale est une identité, pas un routage). Piège du dossier fantôme rencontré une 3ème fois, avec complication inédite (nécessité de recréer le container).

Expérimentation du service Cap testée puis abandonnée définitivement (service retiré du homelab).

## 31/07/2026 — DNS-01 et mise à jour Traefik

Configuration du challenge DNS-01 (OVH) sur le proxy MS-01 — 3 blocages résolus (email ACME manquant, permissions `acme.json`, panne du résolveur DNS OVH contournée via `8.8.8.8`). Découverte que la vraie prod (Mac Mini) souffrait du même problème de renouvellement de certificat depuis 5 jours, corrigé par la même occasion.

## 30/07/2026 — HomeFlix migré et validé

Migration la plus critique du projet : 1.6 To, 426 hardlinks préservés, 9 services. Vérification pré-migration avec `rdfind`. Restructuration stockage (config/cache sur SSD, médias sur NAS). Grand troubleshooting qBittorrent (3 fausses pistes avant la cause réelle : `HostHeaderValidation`). Piège `du` vs `df` découvert et documenté.

## 28/07/2026 — Vaultwarden migré et validé

Découverte du piège `config.json` à domaine figé. Technique d'accès facilité via ACL POSIX mise en place pour la suite du projet.

## 23/07/2026 — Authentik migré et validé

Premier service stateful migré. Protocole de migration affiné à cette occasion (dump/restore Postgres, découverte du branding par domaine et de l'emplacement réel des médias `/data/media`).

## Mi-juillet 2026 — Infrastructure de base

Déploiement complet du socle Proxmox : LXC NAS (MergerFS + NFS + SMB), LXC PBS (Proxmox Backup Server, datastore via NFS), VM Coolify (Docker + Coolify 4.1.2). Autostart et ordre de boot validés par reboot réel. Décision stratégique majeure : migrer seulement 4 services essentiels avant cutover anticipé, plutôt que la stack complète (~15 services) — réduit la fenêtre de risque et la contrainte disque mono-HDD.

<Info>
Journaux de session complets et détaillés disponibles séparément pour l'historique exhaustif (diagnostics pas-à-pas, commandes exactes, fausses pistes explorées). Cette page résume les décisions et jalons, pas le détail opérationnel.
</Info>
