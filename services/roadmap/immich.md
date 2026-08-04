---
title: "Immich (Feuille de Route 2026)"
description: "Gestion et sauvegarde de photos/vidéos self-hosted — alternative à Google Photos"
icon: "image"
iconType: "duotone"
---

<Info>
⏳ **Statut : Planifié (Feuille de Route 2026)** — Application de gestion et sauvegarde automatique de photos/vidéos mobiles, avec reconnaissance faciale IA et recherche vectorielle.
</Info>

## Fiche Technique Prévisionnelle

| Propriété | Valeur Projetée |
|---|---|
| **Domaine Visé** | `photos.ims-world.fr` |
| **Composants** | `immich-server` + `immich-microservices` + `immich-machine-learning` + PostgreSQL (pgvector) + Redis |
| **Stockage** | Données volumineuses hébergées sur le NAS (`/mnt/nas-storage/immich`) |
| **Authentification** | SSO OIDC via [Authentik](/services/authentik) |
| **Statut** | ⏳ Feuille de Route |

## Architecture Projetée

```mermaid
graph TD
    subgraph WAN ["🌐 Accès Web & Mobile"]
        APP["App Mobile Immich / Web (photos.ims-world.fr)"]
        TRAEFIK["Traefik Proxy (TLS 1.3)"]
    end

    subgraph IMMICH_STACK ["🖼️ Stack Immich (VM 104 Docker)"]
        SERVER["Immich Server (API)"]
        ML["Immich Machine Learning (IA / CLIP / Facial)"]
        REDIS["Redis (Queue de traitement)"]
        DB["PostgreSQL + Extension pgvector"]
    end

    subgraph NAS ["📁 Stockage NAS (/mnt/nas-hot)"]
        UPLOADS["/mnt/nas-hot/immich/library"]
        THUMBS["/mnt/nas-hot/immich/thumbs"]
    end

    APP --> TRAEFIK
    TRAEFIK --> SERVER
    SERVER <--> REDIS
    SERVER <--> DB
    SERVER <--> ML
    SERVER <--> UPLOADS
    SERVER <--> THUMBS
```
