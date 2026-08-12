---
title: "IT-Tools"
description: "Boîte à outils développeur & IT en ligne"
icon: "toolbox"
iconType: "duotone"
last_reviewed: "2026-08-12"
app_version: "latest"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active</Badge>

## Accès Rapides & Administration

<Tabs>
  <Tab title="🌐 Interface Web">
    <Card title="IT-Tools Web UI" icon="toolbox" href="https://tools.ims-world.fr">
      Boîte à outils développeur (générateurs, convertisseurs, utilitaires réseau) sur `tools.ims-world.fr` (protégée par SSO Authentik).
    </Card>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Se connecter à la VM Coolify
    ssh cmolotkoff@100.64.0.4

    # Accéder au dossier du service IT-Tools
    cd /data/coolify/services/yefujwl3pxvum45edpsbsru7/

    # Inspecter les logs du conteneur
    docker compose logs -f --tail=100
    ```
  </Tab>
</Tabs>

---

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine** | `tools.ims-world.fr` |
| **Rôle** | Utilitaires développeur & IT en ligne |
| **Image Docker** | `corentinth/it-tools:latest` |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `yefujwl3pxvum45edpsbsru7` |
| **Chemin sur la VM** | `/data/coolify/services/yefujwl3pxvum45edpsbsru7/` |
| **Stockage** | Volume Docker nommé (`it-tools-data:/app/data`) |
| **Authentification** | **Forward-Auth Authentik Outpost** (`ak-outpost-ims-outpost:9000`) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Sécurité (Authentik Forward-Auth Outpost)

IT-Tools ne disposant pas d'un système d'authentification natif, l'accès à `tools.ims-world.fr` est sécurisé en amont au niveau du reverse proxy Traefik via l'**Outpost Forward-Auth Authentik** (`ak-outpost-ims-outpost:9000`).

Toute requête est interceptée par Traefik et nécessite une session Authentik active avant d'atteindre l'interface IT-Tools. Pour le fonctionnement détaillé du middleware, voir la fiche [Authentik](/services/authentik#outpost-proxy--forward-auth).
