---
title: "WhatsUpDocker (WUD) — Veille des Mises à Jour Docker"
description: "Surveillance sémantique (SemVer) des montées de version des conteneurs avec alertes Ntfy et interface web privée"
icon: "docker"
iconType: "brands"
last_reviewed: "2026-09-14"
app_version: "v9.0.2"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active</Badge>

## Accès rapides & administration

<Tabs>
  <Tab title="🌐 Interface Web">
    <Card title="WhatsUpDocker Web UI" icon="docker" href="https://wud.ims-world.fr">
      Tableau de bord de suivi des versions des conteneurs sur `wud.ims-world.fr` (accessible sous Tailnet `100.64.0.0/10`).
    </Card>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Se connecter à la VM Coolify
    ssh cmolotkoff@100.64.0.4

    # Consulter les logs d'exécution et de scan de WUD
    docker logs -f --tail=100 wud-qwe5jrqlqtwqneevgkf6mwr9

    # Inspecter le stockage d'état SQLite
    ls -la /data/coolify/services/qwe5jrqlqtwqneevgkf6mwr9/store/
    ```
  </Tab>
</Tabs>

---

## Fiche service

| Propriété | Valeur |
|---|---|
| **Domaine** | `wud.ims-world.fr` |
| **Rôle** | Veille sémantique des tags Docker (SemVer) & notification push sur nouvelles versions |
| **Version** | `getwud/wud:9.0.2` (tag figé) |
| **Hôte d'orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `qwe5jrqlqtwqneevgkf6mwr9` |
| **Chemin persistant** | `/data/coolify/services/qwe5jrqlqtwqneevgkf6mwr9/store/` |
| **Exposition & sécurité** | **VPN-Only (Tailnet)** via `vpn-only.yaml` (`100.64.0.0/10` & `192.168.1.0/24`) + Auth native WUD v9 |
| **Accès socket Docker** | Monté en **lecture seule** (`/var/run/docker.sock:ro`) |
| **Canal de notification** | Serveur Ntfy privé (`https://ntfy.ims-world.fr`), topic `ims-alerts` (Bearer Auth) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Sécurité & routage réseau (VPN-Only)

L'accès à l'interface `https://wud.ims-world.fr` respecte strictement la politique d'isolation du homelab :

1. **Règle de routage Traefik** : aucun label de routeur n'est défini dans le fichier Compose de WUD, et le champ **Domains** dans l'UI Coolify est laissé vide. Le routeur `wud-admin` et son service sont déclarés exclusivement dans le provider dynamique `/data/coolify/proxy/dynamic/vpn-only.yaml`.
2. **Filtrage d'accès IP** : les requêtes externes hors Tailnet sont interceptées en amont et rejetées en **HTTP 403 Forbidden**.
3. **DNS Split-Horizon Headscale** : l'enregistrement `wud.ims-world.fr` est déclaré dans la section `extra_records` de Headscale et pointe sur l'IP Tailscale locale `100.64.0.4`.
4. **Authentification native** : l'accès est protégé au niveau applicatif par le compte administrateur `cmolotkoff` initialisé dans SQLite via les variables d'environnement chiffrées de Coolify.

---

## Fonctionnement de la détection sémantique (SemVer)

Contrairement aux outils de type Watchtower qui comparent uniquement le condensat SHA256 d'un tag existant, WUD interroge les API des registres (Docker Hub, GHCR, Quay) pour énumérer l'ensemble des tags disponibles :

- **Tags fixes** : si un service est épinglé en `1.32.0`, WUD analyse les nouveaux tags publiés et vous alerte dès qu'une version `1.32.1` (Patch) ou `1.33.0` (Minor) paraît.
- **Classification par criticité** : distinction visuelle entre patchs de sécurité, montées mineures et versions majeures à risque de rupture.
- **Planification raisonnée** : le scan automatique s'exécute une fois par jour à 08h00 (`0 8 * * *`) afin d'éviter tout épuisement du quota de requêtes anonymes Docker Hub.
- **Absence de modification automatique** : WUD est un outil d'observabilité pure. Il n'applique aucune modification sur les conteneurs en production.

---

## Personnalisation et exclusion de conteneurs

Vous pouvez ajuster le comportement de surveillance conteneur par conteneur à l'aide de labels Docker :

```yaml
labels:
  # Exclure un conteneur de la surveillance (ex: stack interne Coolify)
  - "wud.watch=false"

  # Activer la surveillance du digest pour les tags non-SemVer (ex: alpine, latest)
  - "wud.watch.digest=true"

  # Spécifier une temporisation avant alerte (ex: attendre 24h après publication)
  - "wud.watch.delay=24h"
```
