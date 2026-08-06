---
title: "Vaultwarden"
description: "Gestionnaire de mots de passe (implémentation Bitwarden compatible)"
icon: "shield-halved"
iconType: "duotone"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active</Badge>

## Accès & Sauvegarde Coffre-Fort

<Tabs>
  <Tab title="🌐 Interface Web Vaultwarden">
    <Card title="Vaultwarden Web Vault" icon="shield-halved" href="https://vault.ims-world.fr">
      Coffre-fort de mots de passe compatible Bitwarden. Accessible sur `vault.ims-world.fr`.
    </Card>
  </Tab>
  <Tab title="💾 Backup SQLite WAL Sécurisé">
    ```bash
    # Arrêt bref du container pour figer la mémoire partagée WAL
    docker stop vaultwarden-i5ae953riutbot9afjcboptb
    
    # Archivage du dossier data persistent
    sudo tar czf ~/vaultwarden-data.tar.gz -C /data/coolify/services/i5ae953riutbot9afjcboptb/data .
    
    # Redémarrage du service
    docker start vaultwarden-i5ae953riutbot9afjcboptb
    ```
  </Tab>
</Tabs>

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine** | `{domains.vaultwarden}` |
| **Rôle** | Gestionnaire de Mots de Passe Chiffré |
| **Base de Données** | SQLite (mode Write-Ahead Log WAL) |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `i5ae953riutbot9afjcboptb` |
| **Chemin sur la VM** | `/data/coolify/services/i5ae953riutbot9afjcboptb/` |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Topologie Vaultwarden & Stockage WAL

```mermaid
graph TD
    subgraph CLIENTS ["📱 Extension Navigateur / App Bitwarden / Web UI"]
        REQ["Requêtes HTTPS (vault.ims-world.fr)"]
    end

    subgraph PROXY ["🚦 Traefik Proxy Engine"]
        TLS["Certificat SSL Wildcard Let's Encrypt"]
    end

    subgraph VAULT_CONTAINER ["🔒 Vaultwarden Container (VM 104 Docker)"]
        ENGINE["Vaultwarden Rust Engine"]
        CFG["config.json (Domaine Figé: vault.ims-world.fr)"]

        subgraph SQLITE_WAL ["💾 Stockage Persistent SQLite"]
            DB["db.sqlite3 (Base Principale)"]
            SHM["db.sqlite3-shm (Shared Memory)"]
            WAL["db.sqlite3-wal (Write-Ahead Log)"]
        end

        subgraph RSA_KEYS ["🔑 Keys & Attachments"]
            RSA["rsa_key.pem / rsa_key.pub"]
            ATT["attachments/ (PJ chiffrees)"]
        end
    end

    REQ --> TLS
    TLS --> ENGINE
    ENGINE <--> CFG
    ENGINE <--> DB
    DB --- SHM
    DB --- WAL
    ENGINE <--> RSA
    ENGINE <--> ATT

    classDef client fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef proxy fill:#F97316,stroke:#FB923C,color:#fff;
    classDef vault fill:#1a2b3c,stroke:#F97316,color:#fff;
    class REQ client;
    class TLS proxy;
    class ENGINE,CFG,DB,SHM,WAL,RSA,ATT vault;
```

<Warning>
Mapping Authentik custom requis pour `email_verified: true` — sans lui, le SSO OIDC casse silencieusement.
</Warning>
