---
title: "ADR-001 — Stack Monitoring LGTM & Maintien de Dozzle"
description: "Décision d'architecture concernant l'adoption de la stack LGTM/Alloy, l'abandon de Beszel et le maintien de Dozzle pour le live-tailing"
icon: "chart-line"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-12, révisé le 2026-08-17)*

---

## Contexte

L'infrastructure IMS-WORLD s'étend sur plusieurs nœuds physiques et virtuels (Host Proxmox MS-01, VM Coolify 104, LXC NAS 100, LXC PBS 103, Raspberry Pi Kiosk, Mac Mini Standby). Il était nécessaire de disposer d'une visibilité centralisée sur la santé du matériel, les métriques conteneurs et les journaux applicatifs avec un mécanisme d'alerting réactif.

Plusieurs options d'observabilité ont été évaluées :
1. **Beszel** : Solution de monitoring légère et moderne.
2. **Stack Netdata** : Solution d'inspection temps réel.
3. **Dozzle** : Visionneuse de logs Docker en temps réel.
4. **Stack LGTM (Loki, Grafana, Prometheus) + Grafana Alloy** : Standard industriel d'observabilité.

---

## Décisions

### 1. Adoption de la Stack LGTM (Grafana + Loki + Alloy)
Toute la métrologie, la collecte de logs et l'alerting du homelab sont unifiés au sein de la stack Grafana hébergée sur `monitoring.ims-world.fr` :
- **Grafana (v13.1.3)** : Interface unique d'exploration, dashboards PromQL/LogQL et gestion des règles d'alerte.
- **Prometheus** : Base temporelle pour les métriques systèmes et conteneurs (`node_exporter`, `cadvisor`).
- **Loki** : Centralisation et rétention de 30 jours des logs systemd et conteneurs Docker.
- **Grafana Alloy** : Agent universel déployé en service systemd sur les hôtes distants, poussant ses données en Push Remote-Write via le réseau isolé `vmbr1` (`10.10.10.2`).

### 2. Abandon Définitif de Beszel
Le projet **Beszel** est officiellement abandonné. Beszel s'est avéré limité pour la corrélation métriques/logs et l'alerting avancé. L'agent et le serveur sont supprimés pour réduire la surface d'attaque et éliminer la redondance.

### 3. Maintien de Dozzle (`logs.ims-world.fr`)
Bien que Loki assure le stockage et la recherche d'historique de logs, **Dozzle est maintenu en production sur `logs.ims-world.fr`**. 

Dozzle répond à un besoin d'exploitation distinct et complémentaire :
- **Live-Tailing Instantané** : Dozzle permet d'inspecter les logs d'un conteneur en direct (websocket temps réel) en 1 clic depuis son navigateur, sans composer de requête LogQL dans Grafana.
- **Empreinte minimale** : Léger, sécurisé par le middleware Authentik Forward-Auth Outpost et sans base de données.

---

## Synthèse de la Dualité d'Observabilité

| Outil | Domaine | Rôle & Usage Principal | Accès |
|---|---|---|---|
| **Dozzle** | `logs.ims-world.fr` | **Debug en temps réel** (Live-tailing 1 clic, streaming WebSockets) | Forward-Auth Outpost |
| **Stack LGTM (Grafana / Loki)** | `monitoring.ims-world.fr` | **Métrologie & Audit** (Historique 30j, dashboards PromQL/LogQL, alertes Ntfy) | SSO OIDC Natif |

---

## Conséquences

### Positives
- **Source de vérité unique** pour les métriques et l'historique de logs (Grafana/Loki).
- **Alerting unifié** : Grafana centralise l'évaluation des règles d'alerte et pousse vers Ntfy via Webhook.
- **Sécurité réseau** : Ingestion télémétrique cantonnée à la carte virtuelle isolée `10.10.10.2` (`vmbr1`).
- **Confort d'exploitation** : Conservation d'un outil de debug ultra-rapide en direct (Dozzle) couplé à une plateforme d'analyse poussée (Grafana).

### Négatives / Contraintes
- Empreinte mémoire de la stack LGTM sur la VM Coolify (~2 Go RAM).
- Nécessite d'installer et maintenir le binaire `alloy` sur chaque nœud distant.
