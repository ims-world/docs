---
title: "Changelog"
description: "Chronologie du projet de migration Mac Mini → MS-01"
---

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
