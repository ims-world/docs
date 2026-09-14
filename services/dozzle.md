---
title: "Dozzle — Logs Docker en Direct"
description: "Visualisation live des logs de tous les containers Docker, protégée par Authentik"
icon: "list"
iconType: "duotone"
last_reviewed: "2026-09-14"
app_version: "v11.0.1"
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
| **Version** | `amir20/dozzle:v11.0.1` |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `ejdn7jiuwiyixrmp8nffjkcj` |
| **Chemin sur la VM** | `/data/coolify/services/ejdn7jiuwiyixrmp8nffjkcj/` |
| **Exposition & Sécurité** | **VPN-Only + SSO Authentik** (Filtrage IP `100.64.0.0/10` & `192.168.1.0/24` + Forward-Auth) |
| **Accès Socket Docker** | Monté en **lecture seule** (`/var/run/docker.sock:ro`) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Sécurité & Authentification (VPN-Only + SSO Authentik)

L'accès à `logs.ims-world.fr` est sécurisé à deux niveaux complémentaires :
1. **Isolation Réseau (`vpn-only`)** : Filtrage IP au niveau du reverse proxy Traefik via `/data/coolify/proxy/dynamic/vpn-only.yaml`.
2. **Authentification SSO Authentik (`authentik-dozzle@docker`)** : Authentification obligatoire avant d'accéder à l'interface de logs Dozzle.

<Info>
**Priorité du Routeur File Provider** : Dozzle étant déclaré dans `vpn-only.yaml` (provider **file**), son routeur Traefik s'applique prioritairement sur les labels Docker. Le middleware SSO y est explicitement rattaché sous la forme `authentik-dozzle@docker`. Voir le [Post-Mortem d'Incident du 24/08/2026](/history/incidents/2026-08-24-bypass-sso-dozzle-traefik-file-provider).
</Info>

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
