---
title: "ADR-001 — Stack Monitoring LGTM & Grafana Alloy"
description: "Décision d'architecture concernant la centralisation des métriques, logs et alerte"
icon: "chart-line"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-12)*

---

## Contexte

L'infrastructure IMS-WORLD s'étend sur plusieurs nœuds physiques et virtuels (Host Proxmox MS-01, VM Coolify 104, LXC NAS 100, LXC PBS 103, Raspberry Pi Kiosk, Mac Mini Standby). Il était nécessaire de disposer d'une visibilité centralisée sur la santé du matériel, les métriques conteneurs et les journaux applicatifs avec un mécanisme d'alerting réactif.

Plusieurs options ont été évaluées :
1. **Beszel** : Solution de monitoring légère et moderne.
2. **Stack Netdata** : Solution d'inspection temps réel.
3. **Stack LGTM (Loki, Grafana, Prometheus) + Grafana Alloy** : Standard industriel d'observabilité.

---

## Décision

Nous avons retenu la **Stack LGTM hébergée sur la VM IMS-Coolify** avec **Grafana Alloy** comme agent télémétrique unique sur chaque nœud distant.

- **Grafana (v13.1.3)** : Interface unique d'exploration, dashboards et gestion des règles d'alerte.
- **Prometheus** : Base temporelle pour les métriques systèmes et conteneurs (`node_exporter`, `cadvisor`).
- **Loki** : Centralisation et rétention de 30 jours des logs systemd et conteneurs Docker.
- **Grafana Alloy** : Agent universel déployé en service systemd sur les hôtes distants, poussant ses données en Push Remote-Write via le réseau isolé `vmbr1` (`10.10.10.2`).

---

## Conséquences

### Positives
- **Source de vérité unique** pour les logs et les métriques de tout le homelab.
- **Alerting unifié** : Grafana centralise l'évaluation des règles d'alerte et pousse vers Ntfy via Webhook.
- **Sécurité réseau** : Ingestion télémétrique cantonnée à la carte virtuelle isolée `10.10.10.2` (pas d'exposition WAN).

### Négatives / Contraintes
- Empreinte mémoire de la stack LGTM sur la VM Coolify (~2 Go RAM).
- Nécessite d'installer et maintenir le binaire `alloy` sur chaque nœud distant.
