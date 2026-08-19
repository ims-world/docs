---
title: "Protocole de Déploiement d'un Service"
description: "Méthodologie standardisée d'installation, de validation et de mise en production d'un service"
---

## Philosophie & Standard de Déploiement

<Info>
Ce protocole définit la méthode éprouvée pour déployer et mettre en production une nouvelle application sur l'infrastructure IMS-WORLD sans interruption de service ni faille de sécurité.
</Info>

---

## 🗺️ Workflow des 4 Phases de Déploiement

```mermaid
flowchart TD
    subgraph PHASE_A ["Phase 1: Préparation (Coolify & Stockage)"]
        A1["Créer la stack dans Coolify"]
        A2["Configurer les variables & montages NFS/SSD"]
        A3["Définir les réseaux & labels Traefik"]
    end

    subgraph PHASE_B ["Phase 2: Provisioning & Données"]
        B1["Préparer la base de données (Postgres/MySQL/SQLite)"]
        B2["Appliquer les ACL POSIX sur les dossiers"]
        B3["Initialiser les volumes de stockage"]
    end

    subgraph PHASE_C ["Phase 3: Validation Isolée (-ng)"]
        C1["Assigner sous-domaine temporaire: app-ng.ims-world.fr"]
        C2["Démarrer la stack & vérifier les logs Docker"]
        C3["Valider OIDC Authentik, HTTPS TLS & SSL"]
    end

    subgraph PHASE_D ["Phase 4: Mise en Production (Cutover)"]
        D1["Basculer vers le domaine final: app.ims-world.fr"]
        D2["Supprimer le sous-domaine de test -ng"]
        D3["Intégrer le service à la Matrice de Sécurité"]
    end

    PHASE_A --> PHASE_B
    PHASE_B --> PHASE_C
    PHASE_C --> PHASE_D

    classDef prep fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef test fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef prod fill:#1a2b3c,stroke:#0F6E56,color:#fff;
    class A1,A2,A3,B1,B2,B3 prep;
    class C1,C2,C3 test;
    class D1,D2,D3 prod;
```

---

## 📋 Checklist Opérationnelle

### Phase 1 — Choix du Mode de Routage Traefik

- **Option A — Service Public WAN ou avec SSO OIDC Natif (ex: Immich, Vaultwarden, Forgejo)** :
  - [ ] Saisir le FQDN définitif (`https://app.ims-world.fr`) dans le champ **Domains** de l'UI Coolify.
  - [ ] Laisser Coolify générer le routeur Traefik automatique.

- **Option B — Service d'Administration Privé Restreint au Tailnet (`vpn-only`) (ex: qBittorrent, Sonarr, Radarr, Prowlarr, Coolify, Headplane)** :
  - [ ] **Laisser le champ Domains vide dans l'UI Coolify** (pour éviter la concurrence de routeurs).
  - [ ] Supprimer tout label de routage Traefik (`traefik.http.routers...`) du fichier Compose.
  - [ ] Déclarer le router et le service dans `/data/coolify/proxy/dynamic/vpn-only.yaml`.
  - [ ] Suivre la procédure [Sécuriser un Service avec vpn-only](/procedures/securiser-service-vpn-only).

### Phase 2 — Validation et Tests (`-ng`)
- [ ] Utiliser un domaine de qualification temporaire (`nom-ng.ims-world.fr`).
- [ ] Vérifier la génération du certificat Let's Encrypt via le challenge DNS-01.

### Phase 3 — Mise en Production & Audit
- [ ] Valider le FQDN définitif (`nom.ims-world.fr`).
- [ ] Tester le filtrage étanche :
  - **403 Forbidden** depuis le WAN (4G/5G mobile).
  - **200 OK** depuis un appareil authentifié sur le Tailnet (`100.64.0.0/10`).
- [ ] Mettre à jour la [Matrice de Sécurité](/reseau/matrice-securite-exposition) et le [Changelog](/history/chronologie).
