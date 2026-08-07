---
title: "Headscale + Headplane"
description: "Control plane Tailscale self-hosted, avec interface d'administration"
icon: "network-wired"
iconType: "duotone"
---

import TailscaleTable from "/snippets/tailscale-table.mdx";
import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active</Badge>

## Accès Rapides & Administration

<Tabs>
  <Tab title="🌐 Interfaces Web">
    <Card title="Headplane Admin Console" icon="network-wired" href="https://vpn.ims-world.fr">
      Interface de gestion des utilisateurs, clés d'authentification et appareils du réseau Tailnet.
    </Card>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Lister les nœuds enregistrés sur le Tailnet
    docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale nodes list

    # Créer une clé API pour la console Headplane
    docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale apikeys create --expiration 999d
    ```
  </Tab>
  <Tab title="🗺️ IP des Nœuds du Tailnet (100.64.0.0/10)">
    <TailscaleTable />
  </Tab>
</Tabs>

---

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine** | `{domains.headscale}` |
| **Versions** | `headscale/headscale:v0.28.0` + `ghcr.io/tale/headplane:0.6.2` |
| **Base de Données** | SQLite |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `i136ix2bmrrbeovnyrh1o72w` |
| **Chemin sur la VM** | `/data/coolify/services/i136ix2bmrrbeovnyrh1o72w/` |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

---

## Architecture & Topologie

```mermaid
graph TB
    subgraph WAN_PUB ["🌐 Domaine Public (vpn.ims-world.fr)"]
        TRAEFIK["Traefik Proxy (DNS-01 TLS)"]
    end

    subgraph HEADSCALE_STACK ["🔐 Control Plane Stack (VM 104 Docker)"]
        HEADSCALE["Headscale v0.28.0 (Control Plane Server)"]
        HEADPLANE["Headplane 0.6.2 (Web Management GUI)"]
        NOISE_KEY["noise_private.key (Identité Cryptographique)"]
        DB_SQLITE["db.sqlite (Devices, Users, Keys)"]
    end

    subgraph OIDC_AUTH ["🔒 Authentik SSO"]
        AUTH_SRV["auth.ims-world.fr (OIDC Provider)"]
    end

    subgraph TAILNET_NODES ["📱 Nœuds du Tailnet WireGuard (100.64.0.0/10)"]
        MAC["Mac Mini Standby (100.64.0.7)"]
        PVE["Proxmox Host MS-01 (100.64.0.9)"]
        PBS["PBS Storage (100.64.0.2)"]
        COOL["Coolify VM (100.64.0.4)"]
        RPI["Raspberry Pi Kiosk (100.64.0.12)"]
        MOBILE["Clients Mobiles & Laptops"]
    end

    TRAEFIK --> HEADSCALE
    TRAEFIK --> HEADPLANE
    HEADPLANE --> AUTH_SRV
    HEADSCALE --> AUTH_SRV

    HEADSCALE -.->|MagicDNS & Coordination WireGuard| MAC
    HEADSCALE -.->|MagicDNS & Coordination WireGuard| PVE
    HEADSCALE -.->|MagicDNS & Coordination WireGuard| PBS
    HEADSCALE -.->|MagicDNS & Coordination WireGuard| COOL
    HEADSCALE -.->|MagicDNS & Coordination WireGuard| RPI
    HEADSCALE -.->|MagicDNS & Coordination WireGuard| MOBILE

    MAC <==>|Tunnels Directs Peer-to-Peer WireGuard| COOL
    PVE <==>|Tunnels Directs Peer-to-Peer WireGuard| COOL
    MOBILE <==>|Tunnels Directs Peer-to-Peer WireGuard| PBS

    classDef srv fill:#F97316,stroke:#FB923C,color:#fff;
    classDef key fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef node fill:#1a2b3c,stroke:#F97316,color:#fff;
    class HEADSCALE,HEADPLANE,AUTH_SRV srv;
    class NOISE_KEY,DB_SQLITE key;
    class MAC,PVE,PBS,COOL,RPI,MOBILE node;
```

<Warning>
Headscale est un control plane qui coordonne des connexions WireGuard peer-to-peer (Mesh VPN), et non un VPN centralisé classique.
</Warning>

---

## Composants & Fichiers Critiques

| Fichier / Élement | Rôle & Usage | Conséquence si Altéré |
|---|---|---|
| `noise_private.key` | Identité cryptographique du serveur Headscale | Tous les clients verraient un "nouveau serveur" non reconnu |
| `db.sqlite` | Base SQLite des nœuds, utilisateurs et clés d'accès | Perte instantanée de tous les appareils enregistrés |
