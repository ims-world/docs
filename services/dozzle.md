---
title: "Dozzle — Logs Docker en Direct"
description: "Visualisation live des logs de tous les containers Docker, protégée par Authentik"
icon: "list"
iconType: "duotone"
last_reviewed: "2026-08-12"
app_version: "v10.6.15"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active</Badge>

## Accès Rapides & Administration

<Tabs>
  <Tab title="🌐 Interface Web">
    <Card title="Dozzle Web UI" icon="list" href="https://logs.ims-world.fr">
      Vue live et en direct des logs de tous les conteneurs Docker de la VM IMS-Coolify sur `logs.ims-world.fr`.
    </Card>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Se connecter à la VM Coolify
    ssh cmolotkoff@100.64.0.4

    # Accéder au dossier du service Dozzle
    cd /data/coolify/services/ejdn7jiuwiyixrmp8nffjkcj/

    # Logs du service Dozzle lui-même
    docker compose logs -f --tail=100
    ```
  </Tab>
</Tabs>

---

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine** | `logs.ims-world.fr` |
| **Rôle** | Consultation instantanée des logs Docker (VM IMS-Coolify) |
| **Version** | `amir20/dozzle:v10.6.15` |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `ejdn7jiuwiyixrmp8nffjkcj` |
| **Chemin sur la VM** | `/data/coolify/services/ejdn7jiuwiyixrmp8nffjkcj/` |
| **Exposition** | Publique, protégée par **Authentik Forward-Auth Outpost** |
| **Accès Socket Docker** | Monté en **lecture seule** (`/var/run/docker.sock:ro`) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Sécurité (Authentik Forward-Auth)

Dozzle ne possédant pas de système d'authentification natif, l'accès à `logs.ims-world.fr` est sécurisé en amont au niveau du reverse proxy Traefik via l'**Outpost Forward-Auth Authentik** (`ak-outpost-ims-outpost:9000`). 

Toute requête est interceptée par Traefik et nécessite une session Authentik active avant d'atteindre l'interface Dozzle. Pour le détail du fonctionnement de l'Outpost Proxy, consulter la fiche [Authentik](/services/authentik#outpost-proxy--forward-auth).

---

## Dozzle vs. Grafana / Loki — pourquoi les deux existent

Dozzle et la stack Grafana/Loki ne remplissent pas le même rôle, volontairement :

| Besoin | Dozzle (`logs.ims-world.fr`) | Grafana / Loki (`monitoring.ims-world.fr`) |
|---|---|---|
| **Voir en direct ce qui se passe maintenant, en un clic** | ✅ Excellent | Correct (mode Live d'Explore), plus de friction |
| **Historique au-delà de la durée de vie du container** | ❌ Non | ✅ 30 jours de rétention |
| **Recherche/agrégation multi-hôtes** | ❌ Non (VM Coolify uniquement) | ✅ LogQL, `$host`/`$job`/`$container` |
| **Alerting sur contenu de logs** | ❌ Non | ✅ via règles Grafana |
| **Corrélation avec métriques (CPU/RAM au moment d'une erreur)** | ❌ Non | ✅ dashboards croisés |

Garder les deux plutôt que de forcer l'un à remplacer l'autre. Pour la décision d'architecture détaillée, voir l'[ADR-001 — Stack Monitoring LGTM & Maintien de Dozzle](/history/adr/adr-001-stack-monitoring-lgtm).
