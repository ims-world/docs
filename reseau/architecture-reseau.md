---
title: "Architecture réseau"
description: "Bridges Proxmox, Tailscale/Headscale, DNS, port-forward"
icon: "network-wired"
iconType: "duotone"
---

import { ips, domains } from "/snippets/variables.mdx";

## Diagramme Réseau Global

```mermaid
graph TB
    subgraph WAN ["🌐 Internet Public"]
        CLIENT_EXT["Client Internet"]
        OVH_DNS["DNS Wildcard OVH (*.ims-world.fr)"]
    end

    subgraph BBOX ["📡 Bbox / Routeur WAN (IP Publique)"]
        NAT80["NAT Port 80"]
        NAT443["NAT Port 443"]
        NAT2222["NAT Port 2222 (Git SSH)"]
    end

    subgraph PROXMOX_NET ["🖥️ Hyperviseur Proxmox MS-01 (192.168.1.41)"]
        subgraph VMBR0 ["Bridge LAN (vmbr0: 192.168.1.0/24)"]
            NAS_LAN["IMS-NAS (192.168.1.50)\n[SMB / Management]"]
            PBS_LAN["IMS-PBS (192.168.1.51)\n[PBS Web GUI]"]
            COOL_LAN["IMS-Coolify (192.168.1.52)\n[Traefik 80/443 & GUI 8000]"]
        end

        subgraph VMBR1 ["Bridge Isolé (vmbr1: 10.10.10.0/24) — Trafic NFS Interne"]
            NAS_ISO["NAS NFS Target (10.10.10.1)"]
            PBS_ISO["PBS NFS Client (10.10.10.3)"]
            COOL_ISO["Coolify NFS Client (10.10.10.2)"]
        end
    end

    subgraph TAILNET_HEADSCALE ["🔐 Network Overlay (Tailscale 100.64.0.0/10)"]
        HEADSCALE_SRV["Headscale Control Plane\n(vpn.ims-world.fr)"]
        PBS_TS["PBS Client (100.64.0.2)"]
        COOL_TS["Coolify Client (100.64.0.4)"]
        MAC_TS["Mac Mini Standby (100.64.0.7)"]
        PVE_TS["PVE Host (100.64.0.9)"]
        RPI_TS["Raspberry Pi Kiosk (100.64.0.12)"]
    end

    %% Flux WAN -> Bbox -> Traefik
    CLIENT_EXT -->|Requête HTTPS| OVH_DNS
    OVH_DNS -->|IP Publique Bbox| BBOX
    NAT80 -->|Port 80 -> 192.168.1.52:80| COOL_LAN
    NAT443 -->|Port 443 -> 192.168.1.52:443| COOL_LAN

    %% Flux NFS
    COOL_ISO <-->|Montage /mnt/nas-storage & /mnt/nas-hot| NAS_ISO
    PBS_ISO <-->|Sauvegarde /mnt/pbs-datastore| NAS_ISO

    %% Flux Tailscale VPN
    HEADSCALE_SRV --- PBS_TS
    HEADSCALE_SRV --- COOL_TS
    HEADSCALE_SRV --- MAC_TS
    HEADSCALE_SRV --- PVE_TS
    HEADSCALE_SRV --- RPI_TS

    classDef wan fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef lan fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef iso fill:#16A085,stroke:#0F6E56,color:#fff;
    classDef vpn fill:#1a2b3c,stroke:#0F6E56,color:#fff;
    class CLIENT_EXT,OVH_DNS,NAT80,NAT443 wan;
    class NAS_LAN,PBS_LAN,COOL_LAN lan;
    class NAS_ISO,PBS_ISO,COOL_ISO iso;
    class HEADSCALE_SRV,PBS_TS,COOL_TS,MAC_TS,PVE_TS,RPI_TS vpn;
```

## Bridges Proxmox

| Bridge | Plage réseau | Rôle & Usage |
|---|---|---|
| `vmbr0` | `192.168.1.0/24` | LAN principal — SMB, SSH, interfaces d'administration locales |
| `vmbr1` | `10.10.10.0/24` (sans gateway) | Réseau virtuel isolé — trafic NFS haute vitesse entre l'hôte et les guests (NAS ↔ PBS ↔ Coolify) |

## Plan d'Adressage IP Complet (LAN, NFS & Tailnet)

| Équipement / Guest | `vmbr0` (LAN) | `vmbr1` (NFS Isolé) | Tailscale IP | Hostname Tailnet | Rôle |
|---|---|---|---|---|---|
| **Host Proxmox (MS-01)** | `{ips.pveLan}` | — | `{ips.ms01}` | `ims-pve-host` | Hyperviseur Principal Proxmox VE 9 |
| **IMS-NAS (LXC 100)** | `{ips.nasLan}` | `10.10.10.1` | ⚠️ Aucun (LAN) | — | Stockage FUSE MergerFS & NFS/SMB |
| **IMS-PBS (LXC 103)** | `{ips.pbsLan}` | `10.10.10.3` | `{ips.pbs}` | `ims-pve-103-pbs` | Proxmox Backup Server |
| **IMS-Coolify (VM 104)** | `{ips.coolifyLan}` | `10.10.10.2` | `{ips.coolify}` | `ims-pve-104-coolify` | Moteur Docker, Traefik v3.7 & Coolify |
| **Raspberry Pi 3B+** | `192.168.1.x` | — | `{ips.rpi}` | `ims-rpi-monitor` | Affichage Kiosk 2U |
| **Mac Mini 2014** | `192.168.1.x` | — | `{ips.macmini}` | `macmini-standby` | Hôte Standby chaud |

## Headscale — Control Plane Tailscale Self-Hosted

| Propriété | Valeur |
|---|---|
| **Serveur Control Plane** | `{domains.headscale}` |
| **Orchestration** | VM IMS-Coolify (VM 104) |
| **Plage Tailnet** | `100.64.0.0/10` (IPv4), `fd7a:115c:a1e0::/48` (IPv6) |
| **MagicDNS** | `*.ts.ims-world.fr` |

## Port-Forward Bbox

| Règle Bbox | Port Externe | Port Interne | Cible |
|---|---|---|---|
| `Coolify_HTTP` | 80 | 80 | VM Coolify (`{ips.coolifyLan}`) |
| `Coolify_HTTPS` | 443 | 443 | VM Coolify (`{ips.coolifyLan}`) |
| `Forgejo_SSH` | 2222 | 2222 | VM Coolify (`{ips.coolifyLan}`) |

## DNS Public Wildcard (OVH)

- **Configuration Wildcard `*.ims-world.fr`** : La zone DNS publique chez OVH redirige tout le trafic du domaine wildcard `*.ims-world.fr` vers l'IP publique dynamique de la Bbox.
- **Routage Automatique Coolify & Traefik** : Cette configuration permet à Coolify d'attribuer et de déployer n'importe quel nouveau sous-domaine (`auth.ims-world.fr`, `homeflix.ims-world.fr`, `vault.ims-world.fr`, etc.) instantanément, sans avoir à créer manuellement un nouvel enregistrement A dans la console OVH à chaque nouveau service.
- **Certificats HTTPS** : Génération et renouvellement automatisés des certificats TLS Let's Encrypt via le challenge DNS-01 (API OVH).

## Enregistrements Spéciaux Headscale (`extra_records`)

Les enregistrements internes configurés dans `config.yaml` (`dns.extra_records`) de Headscale permettent de mapper des noms FQDN personnalisés uniquement résolubles par les appareils connectés au Tailnet :

```yaml
extra_records:
  - name: "coolify-old.ims-world.fr"
    value: "{ips.macmini}"   # accès de secours à l'ancienne instance Mac Mini
```

## Sécurité Réseau & Firewall Host

| Composant | Rôle |
|---|---|
| **CrowdSec** | Détection et filtrage des comportements malveillants |
| **Fail2ban** | Bannissement dynamique d'IP via `nftables` |
| **Endlessh** | Tarpit anti-bot sur le port 22 (accès SSH légitime sur le port **4242**) |
| **Firewall Proxmox (Roadmap)** | Filtrage 3 niveaux (Host → Datacenter → Guest) en cours de déploiement |
