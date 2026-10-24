---
title: "Uptime Kuma"
description: "Sondes de disponibilité des services et des hôtes, avec alertes sur Ntfy et export des mesures vers Prometheus."
icon: "heart-pulse"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v2.5.4"
---

## Accès

<CardGroup cols={2}>
  <Card title="Uptime Kuma" icon="heart-pulse" href="https://status.ims-world.fr">
    Sondes, historique, notifications.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
cd /data/coolify/services/il53bmpdybmss5q14sfy0umm/ && docker compose logs -f --tail=100
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `status.ims-world.fr` |
| Accès | Public ; forward-auth Authentik (administrateurs) |
| Hôte | VM 104 |
| Coolify | UUID `il53bmpdybmss5q14sfy0umm`, chemin `/data/coolify/services/il53bmpdybmss5q14sfy0umm/` |
| Image | `louislam/uptime-kuma:2.5.4` |
| Données | Base SQLite |
| Sauvegarde | Avec la VM 104 |
| Supervision | Exporte ses mesures vers Prometheus |
| Mise en service | 11/08/2026 |

## Fonctionnement

Uptime Kuma tourne dans le réseau Docker de la VM 104, pas dans le tailnet. Trois familles de sondes :

| Famille | Cible | Pourquoi |
| --- | --- | --- |
| Services publics | Domaine public en HTTPS (Authentik, Vaultwarden, Headscale, Jellyfin, Jellyseerr, Immich, Ntfy…) | Teste le chemin complet, certificat compris |
| Services restreints au tailnet | Nom du conteneur sur le réseau Docker | Par le domaine, la sonde serait refusée par `vpn-only` (fausse panne) |
| Hôtes et services hors Docker | Port TCP, ou HTTPS pour Home Assistant | Un ping prouve que la machine répond, pas le service |

Sondes internes :

| Service | Adresse |
| --- | --- |
| qBittorrent | `http://gluetun-w39uebmcnse7yctsft8hzed8:8080` (qBittorrent partage le réseau de Gluetun) |
| Prowlarr | `http://prowlarr-w39uebmcnse7yctsft8hzed8:9696` |
| Radarr | `http://radarr-w39uebmcnse7yctsft8hzed8:7878` |
| Sonarr | `http://sonarr-w39uebmcnse7yctsft8hzed8:8989` |
| Grafana | `http://grafana-rrw19kmye6gng961igtzqpgw:3000` |
| Coolify | `http://coolify:8080` |

Sondes d'infrastructure : `100.64.0.9:8006` (Proxmox), `100.64.0.2:8007` (PBS), `192.168.1.50:445` (SMB), `https://home.ims-world.fr` (Home Assistant, depuis le 24/10/2026).

## Alertes

Notifications sur le topic Ntfy `ims-alerts`, jeton dédié, priorité urgente. Modèles LiquidJS :

```liquid
{% if status == "Down" %}PANNE{% elsif status == "Up" %}RÉTABLI{% else %}{{ status }}{% endif %} — {{ name }}
```

```liquid
{{ name }} ({{ hostnameOrURL }})
{{ msg }}
{% if heartbeatJSON %}Latence : {{ heartbeatJSON.ping }} ms{% endif %}
```

## Export vers Prometheus

Prometheus lit `/metrics` avec une authentification basique ; la clé est dans `config/uptime-kuma-api-key.txt` du service Supervision, montée en lecture seule.

```yaml
# prometheus.yml (service Supervision)
- job_name: 'uptime-kuma'
  basic_auth:
    username: 'uptimekuma'
    password_file: /etc/prometheus/secrets/kuma-api-key
  static_configs:
    - targets: ['uptime-kuma-il53bmpdybmss5q14sfy0umm:3001']
```

Mesures exportées : `monitor_status`, `monitor_response_time_seconds`, `monitor_uptime_ratio`, `monitor_cert_days_remaining`, `monitor_cert_is_valid`. Elles alimentent les tableaux « Uptime Kuma - Overview » et « Vue d'ensemble » de [Grafana](/services/monitoring).

## Données et sauvegarde

Base SQLite sauvegardée avec la VM 104.

## Exploitation

Ajouter une sonde pour chaque nouveau service (étape de [Déployer un service](/procedures/deploiement-service)).

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `/metrics` en erreur (`Added label X is not included in initial labelset`) | Étiquette créée pendant que le conteneur tourne | `docker restart uptime-kuma-il53bmpdybmss5q14sfy0umm` |
| Fausse panne sur un service restreint | Sonde par le domaine, refusée par `vpn-only` | Sonder le conteneur directement |

## Voir aussi

- [Supervision](/services/monitoring)
- [Ntfy](/services/ntfy)
