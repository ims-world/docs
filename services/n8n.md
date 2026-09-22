---
title: "n8n (Automation)"
description: "Plateforme d'automatisation de flux de travail, webhooks et intégration d'APIs, orchestrée sur Coolify"
icon: "diagram-project"
iconType: "duotone"
last_reviewed: "2026-09-22"
app_version: "latest (1.x)"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Nouveau Service Actif</Badge>

## Accès Rapides & Administration

<Tabs>
  <Tab title="🌐 Interface Web">
    <Card title="n8n Web UI" icon="diagram-project" href="https://automation.ims-world.fr">
      Éditeur visuel de workflows, déclencheurs webhooks et orchestration d'APIs sur `automation.ims-world.fr`.
    </Card>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Se connecter en SSH à la VM Coolify (VM 104)
    ssh cmolotkoff@100.64.0.4

    # Lister les conteneurs n8n
    docker ps | grep n8n

    # Inspecter les logs d'exécution du moteur n8n
    docker logs -f --tail=100 n8n-app

    # Sauvegarde à chaud de la base PostgreSQL n8n
    docker exec -t n8n-postgres pg_dump -U n8n -d n8n | gzip > /tmp/backup_n8n_$(date +%F).sql.gz
    ```
  </Tab>
</Tabs>

---

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine Web** | `automation.ims-world.fr` |
| **Rôle** | Automatisation de flux, récepteur de webhooks, intégrations API et scripts domotiques |
| **Image Docker Principale** | `docker.n8n.io/n8nio/n8n:latest` |
| **Base de Données** | PostgreSQL 16 (`postgres:16-alpine`) |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104, `192.168.1.52`) |
| **Zone Réseau & Exposition** | **Zone 1 (Public WAN)** — Chiffrement TLS 1.3 Let's Encrypt (DNS-01 OVH) + Bouncer CrowdSec |
| **Authentification** | **Native n8n** (User Management avec 2FA / MFA TOTP obligatoire) |
| **Gestion des Webhooks** | Endpoints `/webhook/*` et `/webhook-test/*` publics traversants |
| **Stockage Persistant** | Volumes Docker nommés (`n8n_data` et `postgres_data`) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Architecture & Topologie

```mermaid
graph TB
    subgraph INGRESS ["🌐 Accès Web WAN & Webhooks Externes (Zone 1)"]
        USER["👤 Administrateur (Navigateur Web)"]
        EXTERNAL_SVC["☁️ Services Externes (GitHub, Stripe, Telegram)"]
        HOMELAB_SVC["🏠 Services Internes (Home Assistant, Ntfy)"]
        TRAEFIK["🛡️ Traefik v3 (coolify-proxy - 100.64.0.4)<br/>TLS Let's Encrypt + Plugin CrowdSec"]
    end

    subgraph N8N_STACK ["📦 Stack n8n (VM 104 Docker)"]
        N8N_CORE["⚙️ Moteur n8n (Port 5678)<br/>docker.n8n.io/n8nio/n8n:latest"]
        DB[("🐘 PostgreSQL 16<br/>postgres:16-alpine")]
        VOL_N8N["📁 Volume n8n_data (/home/node/.n8n)"]
        VOL_DB["📁 Volume postgres_data (/var/lib/postgresql/data)"]
    end

    USER -->|HTTPS 443 (Console UI)| TRAEFIK
    EXTERNAL_SVC -->|POST /webhook/*| TRAEFIK
    HOMELAB_SVC -->|POST /webhook/*| TRAEFIK

    TRAEFIK -->|Reverse Proxy HTTP :5678| N8N_CORE
    N8N_CORE <-->|Réseau bridge 'internal' :5432| DB
    N8N_CORE --- VOL_N8N
    DB --- VOL_DB

    classDef web fill:#0284C7,stroke:#0369A1,color:#fff;
    classDef n8n fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef ext fill:#F97316,stroke:#FB923C,color:#fff;
    class USER,TRAEFIK web;
    class N8N_CORE,DB,VOL_N8N,VOL_DB n8n;
    class EXTERNAL_SVC,HOMELAB_SVC ext;
```

---

## 🔒 Choix d'Architecture : Pourquoi l'Authentification Native ?

Dans n8n, le connecteur **SSO SAML / OIDC natif** est une fonctionnalité réservée à l'offre commerciale **n8n Enterprise**. En version gratuite auto-hébergée (Community), l'intégration d'un SSO tiers nécessiterait d'intercaler un proxy inverse Forward-Auth (comme l'Outpost Authentik).

Cependant, intercaler un Forward-Auth devant n8n introduit un risque majeur : **la rupture des webhooks**. Tout service externe (GitHub, Stripe, Telegram, Home Assistant) envoyant un payload HTTP sur `/webhook/...` serait redirigé vers la mire d'authentification SSO Authentik (HTTP 302) et échouerait.

**La solution adoptée est donc l'authentification native de n8n :**
1. **Accès Interface Administrateur** : Protégé par compte email/mot de passe robuste et **validation 2FA par clé TOTP** (compatible Vaultwarden ou votre application d'authentification).
2. **Webhooks Publics Traversants** : Les requêtes envoyées vers `https://automation.ims-world.fr/webhook/...` atteignent directement le moteur d'exécution de n8n sans friction.
3. **Protection Périmétrique** : L'exposition sur l'Internet public bénéficie de la protection globale du **plugin bouncer CrowdSec** sur Traefik (bannissement immédiat des scans et forces brutes).

---

## Déploiement sur Coolify (Docker Compose)

Créez une nouvelle ressource de type **Docker Compose Service** dans Coolify et collez la configuration ci-dessous :

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n-app
    restart: unless-stopped
    environment:
      - N8N_HOST=automation.ims-world.fr
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://automation.ims-world.fr/
      - GENERIC_TIMEZONE=Europe/Paris
      - TZ=Europe/Paris
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB:-n8n}
      - DB_POSTGRESDB_USER=${POSTGRES_USER:-n8n}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - EXECUTIONS_DATA_PRUNE=true
      - EXECUTIONS_DATA_MAX_AGE=168
      - EXECUTIONS_DATA_PRUNE_MAX_COUNT=50000
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - coolify
      - internal
    depends_on:
      postgres:
        condition: service_healthy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.n8n.rule=Host(`automation.ims-world.fr`)"
      - "traefik.http.routers.n8n.entrypoints=https"
      - "traefik.http.routers.n8n.tls=true"
      - "traefik.http.routers.n8n.tls.certresolver=letsencrypt"
      - "traefik.http.services.n8n.loadbalancer.server.port=5678"

  postgres:
    image: postgres:16-alpine
    container_name: n8n-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB:-n8n}
      - POSTGRES_USER=${POSTGRES_USER:-n8n}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - internal
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -h localhost -U ${POSTGRES_USER:-n8n} -d ${POSTGRES_DB:-n8n}"]
      interval: 5s
      timeout: 5s
      retries: 10

volumes:
  n8n_data:
  postgres_data:

networks:
  coolify:
    external: true
  internal:
    driver: bridge
```

---

## 🔑 Variables d'Environnement Requises (Secrets Coolify)

Dans l'onglet **Environment Variables** de la ressource Coolify, déclarez les variables suivantes :

| Variable | Description & Règle de Génération | Secret ? |
|---|---|:---:|
| `POSTGRES_DB` | Nom de la base de données : `n8n` | ❌ Non |
| `POSTGRES_USER` | Utilisateur PostgreSQL : `n8n` | ❌ Non |
| `POSTGRES_PASSWORD` | Mot de passe fort aléatoire (`openssl rand -base64 24`) | ✅ **Oui** |
| `N8N_ENCRYPTION_KEY` | Clé maîtresse AES de chiffrement des credentials (`openssl rand -hex 32`) | ✅ **Oui** |

<Warning>
**Sauvegardez impérativement la variable `N8N_ENCRYPTION_KEY`** :
Cette clé sert à chiffrer toutes les clés API, jetons et mots de passe enregistrés dans vos flux n8n. Si cette clé est perdue lors d'une réinstallation, l'ensemble des identifiants enregistrés deviendra illisible dans la base de données.
</Warning>

---

## ⚙️ Explication des Paramètres d'Hygiène & Performance

- **`EXECUTIONS_DATA_PRUNE=true`** : Active la purge automatique des journaux d'exécution terminée. Sans ce paramètre, la base de données PostgreSQL saturera rapidement le stockage SSD du serveur.
- **`EXECUTIONS_DATA_MAX_AGE=168`** : Conserve l'historique des exécutions pendant **7 jours** (168 heures) avant suppression.
- **`EXECUTIONS_DATA_PRUNE_MAX_COUNT=50000`** : Limite stricte de 50 000 entrées d'historique maximum en mémoire.
- **`WEBHOOK_URL=https://automation.ims-world.fr/`** : Indique à n8n comment formater les URLs publiques de webhooks générées dans l'interface pour les intégrations tierces.
- **Isolation Réseau (`internal`)** : Le conteneur PostgreSQL n'est attaché qu'au réseau `internal`. Il ne rejoint pas le réseau `coolify` partagé, garantissant une étanchéité totale face à Traefik et aux autres conteneurs.

---

## Procédure de Premier Démarrage & Onboarding

<Steps>
  <Step title="Déploiement initial sur Coolify">
    Dans Coolify, cliquez sur **Deploy**. Coolify initialise le volume PostgreSQL, attend le passage au statut sain (`healthy`), puis démarre le conteneur n8n.
  </Step>

  <Step title="Création du Compte Propriétaire">
    Ouvrez `https://automation.ims-world.fr` dans votre navigateur.
    Remplissez l'écran d'onboarding initial :
    - Adresse email administrateur
    - Prénom / Nom
    - Mot de passe fort (généré et stocké dans [Vaultwarden](/services/vaultwarden))
  </Step>

  <Step title="Activation Immédiate du 2FA TOTP (Indispensable)">
    Le service étant exposé sur le WAN public :
    1. Rendez-vous dans **Paramètres** (icône d'engrenage en bas à gauche) ➔ **Profil utilisateur**.
    2. Cliquez sur **Activer l'authentification à deux facteurs (2FA)**.
    3. Scannez le QR Code avec votre gestionnaire (Vaultwarden) et validez le code à 6 chiffres.
    4. Enregistrez précieusement les codes de secours générés.
  </Step>

  <Step title="Test d'un Webhook">
    Créez un workflow de test avec un nœud déclencheur **Webhook** (méthode `GET` ou `POST`, chemin `test-ping`).
    Passez le workflow en **Actif** et testez l'appel depuis un terminal :
    ```bash
    curl -I https://automation.ims-world.fr/webhook/test-ping
    ```
    La réponse doit retourner immédiatement un code `200 OK`.
  </Step>
</Steps>

---

## 💾 Procédures d'Exploitation & Sauvegardes

### Sauvegarde à Chaud de la Base PostgreSQL
Pour effectuer un snapshot complet de la configuration, des flux et des identifiants :

```bash
# Se connecter en SSH sur la VM 104
ssh cmolotkoff@100.64.0.4

# Exécuter le pg_dump dans le conteneur PostgreSQL
docker exec n8n-postgres pg_dump -U n8n -d n8n -F c -b -v -f /tmp/n8n_backup.dump

# Copier le dump vers l'hôte
docker cp n8n-postgres:/tmp/n8n_backup.dump ./n8n_backup_$(date +%F).dump
```

### Restauration d'une Sauvegarde
En cas de corruption ou de reprise sur incident :

```bash
docker cp ./n8n_backup.dump n8n-postgres:/tmp/n8n_backup.dump
docker exec -it n8n-postgres pg_restore -U n8n -d n8n -v --clean /tmp/n8n_backup.dump
```
