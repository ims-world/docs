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
| **Exposition** | **VPN-Only** (Restreint au Tailnet `100.64.0.0/10` & LAN `192.168.1.0/24` via `vpn-only.yaml`) |
| **Accès Socket Docker** | Monté en **lecture seule** (`/var/run/docker.sock:ro`) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Sécurité & Isolation Réseau (`vpn-only`)

Dozzle ne possédant pas de système d'authentification natif, l'accès à `logs.ims-world.fr` est sécurisé au niveau du reverse proxy Traefik via le provider file centralisé **`vpn-only.yaml`** (filtrage `ipAllowList: [100.64.0.0/10, 192.168.1.0/24]`).

<Check>
**Isolation Étanche (HTTP 403 Forbidden sur le WAN)** : Tout accès depuis l'Internet public (4G/5G mobile hors VPN) est immédiatement bloqué en HTTP 403. Pour le détail de l'isolation et du DNS split-horizon Headscale (`extra_records`), voir la procédure [Sécuriser un Service avec vpn-only](/procedures/securiser-service-vpn-only).
</Check>

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
