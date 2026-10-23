---
title: "Stack Monitoring (LGTM)"
description: "Grafana, Loki, Prometheus & Grafana Alloy — métriques, centralisation de logs & monitoring SMART"
icon: "chart-line"
iconType: "duotone"
last_reviewed: "2026-10-24"
app_version: "v13.1.3"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active (Stack LGTM + Grafana Alloy)</Badge>

## Accès Rapides & Administration

<Tabs>
  <Tab title="🌐 Interfaces Web">
    <CardGroup cols={2}>
      <Card title="Grafana (Dashboards, Logs & Alerting)" icon="chart-line" href="https://monitoring.ims-world.fr">
        Interface centrale de métrologie, d'exploration de logs Loki et de gestion des règles d'alerte. Accessible sur `monitoring.ims-world.fr` (Tailnet + SSO Authentik).
      </Card>
      <Card title="Ntfy — Notifications Push" icon="bell" href="/services/ntfy">
        Les notifications d'alerte Grafana sont délivrées via Ntfy (service dédié, voir la fiche Ntfy).
      </Card>
    </CardGroup>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Se connecter à la VM Coolify
    ssh cmolotkoff@100.64.0.4

    # Accéder au dossier du service Monitoring
    cd /data/coolify/services/rrw19kmye6gng961igtzqpgw/

    # Logs des conteneurs (Grafana, Loki, Prometheus)
    docker compose logs -f --tail=100

    # Vérifier le statut de l'agent Alloy sur un hôte systemd
    sudo systemctl status alloy
    ```
  </Tab>
</Tabs>

---

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine** | `monitoring.ims-world.fr` |
| **Rôle** | Centralisation des Métriques (Prometheus), Logs (Loki) & Visualisation/Alerting (Grafana) |
| **Versions** | Grafana `13.1.3` / Loki `3.3.2` / Prometheus `v3.1.0` / Alloy `1.18.1` |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `rrw19kmye6gng961igtzqpgw` |
| **Chemin sur la VM** | `/data/coolify/services/rrw19kmye6gng961igtzqpgw/` |
| **Mode d'Ingestion** | **Push Remote-Write** (Alloy) + **Pull Scrape** (Uptime Kuma & Traefik) |
| **Rétention Métriques/Logs** | **1 An** (Prometheus TSDB `--storage.tsdb.retention.time=1y`) & **30j** (Loki `retention_period: 720h`) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

<Info>
**Extrapolation Rétention 1 An** : La taille TSDB constatée sur 30 jours est de **464 Mo**. L'extrapolation sur 1 an représente environ **5 à 6 Go** de stockage, ce qui est tout à fait négligeable sur le stockage SSD 4 To de la VM Coolify.
</Info>

---

## Architecture & Topologie

```mermaid
graph TB
    subgraph INGRESS ["🌐 Accès Web & Reverse Proxy (Tailnet Only)"]
        CLIENT_VPN["Client Tailnet (100.64.0.x)"]
        TRAEFIK["Traefik v3.7 (vpn-only.yaml File Provider)"]
        AUTHENTIK["Authentik SSO (auth.ims-world.fr OIDC)"]
    end

    subgraph CENTRAL_STACK ["📊 Stack Centrale Monitoring (VM 104 Docker)"]
        GRAFANA["Grafana v13.1.3 (Port 3000)"]
        PROMETHEUS["Prometheus v3.1.0 (Port 9090 — TSDB 1y)"]
        LOKI["Loki v3.3.2 (Port 3100 — Log Push)"]
    end

    subgraph SCRAPE_PULL ["📥 Scraping Pull Prometheus"]
        KUMA["Uptime Kuma (/metrics Basic Auth)"]
        TRAEFIK_METRICS["Traefik Proxy Engine (:8080)"]
    end

    subgraph AGENTS ["⚙️ Agents Grafana Alloy & Textfile Collectors (8 Hôtes)"]
        MS01_ALLOY["MS-01 Bare Metal (host: ms01-pve + smartmon.sh)"]
        MAC_ALLOY["Mac Mini Bare Metal (Alloy + smartmon.sh Cron 5m)"]
        NAS_ALLOY["LXC 100 IMS-NAS (Alloy systemd)"]
        PBS_ALLOY["LXC 103 IMS-PBS (Alloy systemd)"]
        COOL_ALLOY["VM 104 IMS-Coolify (Alloy + cAdvisor Docker)"]
        WORKER_ALLOY["LXC 105 Worker (host: ims-coolify-worker)"]
        RPI_ALLOY["Raspberry Pi Kiosk (Alloy systemd)"]
        TEST_ALLOY["VM 101 vm-test (Alloy systemd)"]
    end

    CLIENT_VPN -->|HTTPS| TRAEFIK
    TRAEFIK --> GRAFANA
    GRAFANA --> AUTHENTIK

    MS01_ALLOY -->|Push LAN 192.168.1.52:9090/3100| PROMETHEUS
    MS01_ALLOY -->|Push LAN| LOKI

    MAC_ALLOY -->|Push LAN 192.168.1.52:9090/3100| PROMETHEUS
    MAC_ALLOY -->|Push LAN| LOKI

    WORKER_ALLOY -->|Push LAN 192.168.1.52:9090/3100| PROMETHEUS
    WORKER_ALLOY -->|Push LAN| LOKI

    TEST_ALLOY -->|Push LAN 192.168.1.52:9090/3100| PROMETHEUS
    TEST_ALLOY -->|Push LAN| LOKI

    RPI_ALLOY -->|Push LAN 192.168.1.52:9090/3100| PROMETHEUS
    RPI_ALLOY -->|Push LAN| LOKI

    NAS_ALLOY -->|Push Isolé 10.10.10.2:9090/3100| PROMETHEUS
    NAS_ALLOY -->|Push Isolé| LOKI

    PBS_ALLOY -->|Push Isolé 10.10.10.2:9090/3100| PROMETHEUS
    PBS_ALLOY -->|Push Isolé| LOKI

    COOL_ALLOY -->|Push Local 10.10.10.2:9090/3100| PROMETHEUS
    COOL_ALLOY -->|Push Local| LOKI

    PROMETHEUS -.->|Scrape /metrics| KUMA
    PROMETHEUS -.->|Scrape :8080| TRAEFIK_METRICS
```

### 1. Ingestion Hybride : Push Remote-Write + Pull Uptime Kuma/Traefik

- **Push Remote-Write (Alloy)** : Les **8 agents Alloy** (MS-01, Mac Mini, VM Coolify, LXC NAS, LXC PBS, RPi Kiosk, Worker LXC 105, VM 101 vm-test) poussent leurs métriques (`prometheus.remote_write`) et leurs logs (`loki.write`) directement vers la stack centrale via le LAN (`192.168.1.52`) ou le bridge isolé `vmbr1`. Le niveau de log des agents est configuré en `info` (abaissé depuis `debug` le 23/10/2026). La télémétrie ne transite **jamais par Tailscale**.
- **Tuning LXC (Désactivation `hwmon`)** : Le sous-collecteur `hwmon` de Node Exporter est désactivé sur les agents Alloy des conteneurs LXC (LXC 100, LXC 103, LXC 105). Les capteurs matériels physiques (`/sys/class/hwmon`) n'étant pas accessibles dans les conteneurs, cette exclusion supprime les erreurs d'accès et le bruit dans les logs Loki tout en maintenant la supervision thermique sur les hyperviseurs physiques.
- **Pull Uptime Kuma (Exception)** : Prometheus scrape l'endpoint natif `/metrics` d'Uptime Kuma via Basic Auth.
  - La clé API est stockée dans `config/uptime-kuma-api-key.txt` et montée en volume `:ro` dans le conteneur Prometheus (`/etc/prometheus/secrets/kuma-api-key`).
  - **Métriques lues** : `monitor_status`, `monitor_response_time_seconds`, `monitor_uptime_ratio`, `monitor_cert_days_remaining`, `monitor_cert_is_valid`.

<Warning>
**Bug de Création de Tag Uptime Kuma** : La création d'une étiquette (tag) dans l'IHM Uptime Kuma pendant que le conteneur tourne casse temporairement l'export `/metrics`.
**Fix** : Exécuter un redémarrage du conteneur après toute création d'étiquette :
```bash
docker restart uptime-kuma-il53bmpdybmss5q14sfy0umm
```
</Warning>

### 2. Monitoring SMART des Disques Physiques (`ms01-pve` & `pve-macmini`)

Seuls les deux hôtes Proxmox bare-metal (MS-01 et Mac Mini) ont un accès matériel direct aux disques physiques (`nvme0n1`, SSD 4To, HDD Apple/Seagate 3To et SSD Apple 256 Go du Mac Mini). Les conteneurs LXC (NAS, PBS) ne remontent pas le numéro de série SMART.

- **Méthode** : *textfile collector* Node Exporter via Alloy, alimenté par le script communautaire `smartmon.sh` exécuté toutes les 5 minutes par cron sur MS-01 et Mac Mini.
- **Règle de Cron (`/etc/cron.d/smartmon`)** :
  ```bash
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  */5 * * * * root /usr/local/bin/smartmon.sh > /var/lib/node_exporter/textfile_collector/smartmon.prom.$$ && mv /var/lib/node_exporter/textfile_collector/smartmon.prom.$$ /var/lib/node_exporter/textfile_collector/smartmon.prom
  ```

<Info>
**Respect du Spin-Down des Disques** : Le script `smartmon.sh` utilise nativement `smartctl -n standby`. Si un disque HDD (`sda`/`sdb`) est en veille, le script **saute la lecture SMART sans réveiller le disque** (`smartmon_device_active = 0`). Cela préserve le mode veille configuré (30 min d'inactivité).
</Info>

### 3. Métriques Traefik & Formatage des Services

Prometheus scrape les métriques de Traefik sur `coolify-proxy:8080`.

<Warning>
**Formatage des Noms de Services Coolify** : Les noms de services générés par Coolify contiennent des hashes et suffixes variables (ex. `https-N-<hash>-`, `@docker`, `@file`). Les dashboards Grafana utilisent une expression régulière `label_replace` en deux passes pour nettoyer ces libellés et afficher des noms lisibles.
</Warning>

### 4. Healthchecks Docker Nfs

```yaml
  prometheus:
    healthcheck:
      test: ['CMD', 'wget', '--no-verbose', '--tries=1', '--spider', 'http://localhost:9090/-/healthy']
  loki:
    healthcheck:
      test: ['CMD', 'wget', '--no-verbose', '--tries=1', '--spider', 'http://localhost:3100/ready']
  grafana:
    healthcheck:
      test: ['CMD-SHELL', 'wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1']
```

---

## Dashboards Grafana

### Galerie des Tableaux de Bord

<Frame caption="Dashboard Master — Vue d'ensemble IMS-WORLD">
  ![Dashboard Master Vue d'ensemble — IMS-WORLD](/assets/grafana-dashboard-overview.png)
</Frame>

<Frame caption="Dashboard — Gestion des disques & Santé SMART">
  ![Dashboard Gestion des disques & SMART](/assets/grafana-dashboard-disks.png)
</Frame>

<Frame caption="Dashboard — Traefik Reverse Proxy Engine">
  ![Dashboard Traefik — Reverse Proxy](/assets/grafana-dashboard-traefik.png)
</Frame>

<Frame caption="Inventaire des Dashboards Grafana & Tags">
  ![Vue d'ensemble des Dashboards et Tags](/assets/grafana-dashboards-list.png)
</Frame>

---

### Inventaire des Dashboards en Production

| Dashboard | Source | Description & Indicateurs Clés |
|---|---|---|
| **Vue d'ensemble — IMS-WORLD** | Custom | **Master Dashboard Exécutif** : Statut global des services (Uptime Kuma), taux de stockage, santé SMART disques, certificats SSL `<30j`, matrice CPU/RAM/Disque par hôte. |
| **Gestion des disques** | Custom | **Capacité & Santé Disques** : Taux d'occupation global, capacité totale/libre (6.27 TiB / 3.81 TiB), répartition Froid/Chaud, tendance 90j, I/O, inventaire physique, santé SMART et températures. |
| **Traefik — Reverse Proxy** | Custom | **Trafic Web & Proxy Engine** : Requêtes/s globales, taux d'erreurs 4xx/5xx (fenêtre 15m pour lisser le bruit), connexions ouvertes, top 10 services les plus demandés, latences. |
| **Uptime Kuma - Overview** | Custom | **Disponibilité Applicative** : Statut des 17 services, temps de réponse, validité et jours restants des certificats SSL, SLO (24h/7j/30j), historique chronologique des coupures. |
| **Node Exporter Full** | Grafana ID `1860` (Fixé) | **Vue Système par Hôte** : CPU, mémoire, I/O disque, réseau. *Note : Variables corrigées pour matcher les vrais labels (`job=integrations/unix`, `host=`).* |
| **Docker monitoring** | Grafana ID `15798` + Custom | **Métriques Conteneurs** : Consommation CPU/RAM/Network par conteneur Docker via cAdvisor. |
| **Logs — Vue d'ensemble** | Custom (`logs-overview.json`) | **Explorateur de Logs Loki** : Filtrable par `$host`, `$job`, `$container` et `$level` (ERROR, WARN, INFO). *Note opérationnelle : Pour afficher les logs systemd-journal au lieu de masquer les entrées non-conteneur, régler la variable `$container` sur `.*` (All).* |

---

## Alerting & Contact Point Ntfy

Grafana gère l'évaluation des règles d'alerte et transmet directement les notifications push à **[Ntfy](/services/ntfy)** via un contact point Webhook.

### 1. Configuration du Contact Point Webhook
- **URL Cible** : `https://ntfy.ims-world.fr/ims-alerts`
- **Méthode HTTP** : `POST`
- **Header d'autorisation** : `Bearer <token_ntfy_grafana>`
- **Canal de Diffusion** : Topic `ims-alerts` (push direct sur mobiles et notifications web).

### 2. Règles d'Alerte Grafana (Group `default`)

| Règle | Requête PromQL (Instant) | Condition | Pending | Description & Action |
|---|---|---|---|---|
| **Disque SMART dégradé** | `smartmon_device_smart_healthy == 0` | Passage de 1 à 0 (`== 0`) | 0s | **Alerte Urgente** : Déclenchement immédiat avec mention explicite du nom et modèle du disque défaillant (`device` / `model`). |
| **CPU élevé** | `100 - (avg by (host) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` | > 90% | 5m | Alerte de charge processeur soutenue sur 5 minutes. |
| **RAM élevée** | `(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100` | > 90% | 5m | Alerte de saturation mémoire physique. |
| **Disque plein** | `(1 - node_filesystem_avail_bytes{fstype!~"tmpfs\|overlay"} / node_filesystem_size_bytes) * 100` | > 85% | 10m | Alerte d'espace disque disponible critique. |
| **Host down** | `time() - timestamp(node_load1) > 120` | > 120s | 1m | Alerte d'absence de métriques / nœud injoignable. |
| **Container crash-loop** | `changes(container_start_time_seconds{image!="", host="ims-coolify"}[15m])` | > 2 | 0s | Alerte de redémarrages en boucle d'un conteneur Docker. |
