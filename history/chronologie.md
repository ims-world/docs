---
title: "Changelog"
description: "Chronologie du projet de migration Mac Mini → MS-01"
---

## Semaine du 04/08/2026 — Résilience matérielle & monitoring hors-bande

### 🆕 Nouveautés

- **Mac Mini basculé en hôte de secours (Standby chaud)** — L'ancien serveur de production reste sous tension dans le rack, prêt à reprendre la charge en cas de panne majeure du MS-01. Une procédure de bascule d'urgence (Bbox port-forward + relance des conteneurs archivés) est documentée. Voir [Mac Mini 2014 (Hôte Standby)](/infrastructure/mac-mini).
- **Sonde de monitoring Raspberry Pi 3B+** — Un Raspberry Pi indépendant surveille désormais l'hyperviseur, le NAS et la VM Coolify via healthchecks ICMP/HTTP, avec alertes Ntfy en cas de panne. Le monitoring continue de fonctionner même si Proxmox est éteint ou en reboot. Voir [Raspberry Pi 3B+ & Écrans](/infrastructure/rpi-monitor).
- **Écran de statut en façade du rack** — Un afficheur LCD/OLED intégré au panneau supérieur du rack Labrax montre en temps réel le logo IMS, la température et la charge du cluster, piloté par le Raspberry Pi.

### 🔧 Améliorations

- **Module 3D imprimable pour le Raspberry Pi de monitoring** — La fiche du Raspberry Pi 3B+ référence désormais le modèle MakerWorld « Screen module for 10-inch rack — Raspberry Pi 2U », qui permet d'intégrer proprement le Pi et son écran de statut en façade du rack Labrax. Voir [Raspberry Pi 3B+ & Écrans](/infrastructure/rpi-monitor).
- **Tarpit SSH sur le Mac Mini** — Le port 22 du Mac Mini pointe désormais vers Endlessh (tarpit anti-bot) pour piéger les scans automatisés. L'accès SSH légitime passe par le port `4242`.
- **Résolution DNS de l'ancien Coolify** — L'ancienne instance Coolify du Mac Mini reste joignable via `coolify-old.ims-world.fr` grâce à un enregistrement DNS dans Headscale, le temps de la phase de validation post-cutover.

<Info>
Prochaine étape planifiée : mise en service de [Cap](/services/cap) (enregistrement et partage d'écran self-hosted), toujours en attente de déploiement.
</Info>

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
- **Coolify : warning cosmétique** — Le message d'avertissement affiché dans l'admin Coolify après le cutover a été corrigé.

<Info>
Prochaine étape planifiée : mise en service de [Cap](/services/cap) (enregistrement et partage d'écran self-hosted), actuellement en attente de déploiement.
</Info>

## 🎉 02/08/2026 — Cutover complet

Les 4 services essentiels (Authentik, Vaultwarden, HomeFlix, Headscale/Headplane) sont basculés en production sur le MS-01. Port-forward Bbox corrigé et basculé. Découverte structurelle majeure : le port-forward route tout le trafic public d'un coup, pas de bascule partielle possible (voir [Traefik Proxy](/reseau/traefik-proxy)).

Cascade de 8 blocages résolus dans la même session : accès console (Chrome), SSH manquant, label réseau Traefik manquant, port-forward mal ciblé, cache DNS transitoire, crash-loop OIDC Headscale, certificats DNS-01 en cours de négociation, warning cosmétique Coolify.

Mise à jour Traefik v3.6.23 → v3.7 effectuée en avance sur le planning, sans incident réel.

## 01/08/2026 — Préparation Headscale (Phases A-C)

Migration de données complète (config, `noise_private.key`, base SQLite) — tous les utilisateurs et appareils confirmés présents. Décision actée de ne pas tester via sous-domaine `-ng` (le `server_url` de Headscale est une identité, pas un routage). Piège du dossier fantôme rencontré une 3ème fois, avec complication inédite (nécessité de recréer le container).

Migration Cap commencée en parallèle (dump MySQL + données MinIO récupérés), mise en pause avant le premier démarrage.

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
