---
title: "Vue d'ensemble"
description: "Infrastructure self-hosted IMS-WORLD — architecture, principes et état actuel"
---

![Architecture Homelab IMS-WORLD](/assets/hero-banner.png)

## Principes

L'infrastructure IMS-WORLD repose sur trois principes non négociables :

<CardGroup cols={3}>
  <Card title="100% local" icon="house">
    Aucune dépendance à un cloud tiers pour les services critiques. Auto-hébergement complet.
  </Card>
  <Card title="0€ récurrent" icon="euro-sign">
    Uniquement du matériel possédé et des logiciels open-source. Pas d'abonnement SaaS.
  </Card>
  <Card title="Lean" icon="feather">
    Complexité activement challengée. Pas de sur-ingénierie — chaque brique ajoutée doit se justifier.
  </Card>
</CardGroup>

## Schéma d'Architecture Globale

```mermaid
flowchart TB
    subgraph WAN ["🌐 Internet / WAN"]
        USERS["Utilisateurs / Clients Distants"]
        DNS["DNS Public OVH (*.ims-world.fr)"]
        ACME["Let's Encrypt (DNS-01 ACME)"]
    end

    subgraph ROUTER ["📡 Routeur / Bbox"]
        PF["Port Forward (80/443 NAT)"]
    end

    subgraph PROXMOX_CLUSTER ["🖥️ Cluster Proxmox VE ims-cluster (v9.2.11 — Quorum 2/2)"]
        subgraph NODE1 ["Nœud 1 (Leader) — Minisforum MS-01 (192.168.1.41 / 100.64.0.9)"]
            PVE_HOST["Proxmox Host (pve — 192.168.1.41)"]
            
            subgraph VMBR1 ["Bridge Isolé NFS (vmbr1: 10.10.10.0/24)"]
                direction LR
                NAS_ISO["NAS FUSE (10.10.10.1)"]
                PBS_ISO["PBS (10.10.10.3)"]
                COOL_ISO["Coolify VM (10.10.10.2)"]
            end

            subgraph GUESTS ["Guests de Production (MS-01)"]
                LXC_NAS["IMS-NAS (LXC 100 — 192.168.1.50)"]
                LXC_PBS["IMS-PBS (LXC 103 — 192.168.1.51)"]
                VM_COOLIFY["IMS-Coolify (VM 104 — 192.168.1.52)"]
            end
        end

        subgraph NODE2 ["Nœud 2 — Apple Mac Mini (192.168.1.42 / 100.64.0.6)"]
            MAC_MINI["Mac Mini (pve-macmini — 192.168.1.42)"]
        end

        PVE_HOST <==>|Corosync Cluster Link / pvecm| MAC_MINI
    end

    subgraph TAILNET ["🔐 Tailnet VPN (Headscale 100.64.0.0/10)"]
        CLIENTS["Appareils Distants (100.64.0.x)"]
    end

    subgraph SATELLITES ["📺 Affichage & Kiosk"]
        RPI["Raspberry Pi 3B+ Kiosk (100.64.0.12)"]
    end

    %% Connexions WAN et Réseau
    USERS -->|HTTPS| DNS
    DNS -->|IP Publique| PF
    PF -->|80/443| TRAEFIK
    TRAEFIK <-->|ACME DNS-01| ACME

    %% Routage Interne Traefik
    TRAEFIK --> AUTH
    TRAEFIK --> VAULT
    TRAEFIK --> HOMEFLIX
    TRAEFIK --> HEADSCALE

    %% Liaisons NFS Interne vmbr1
    COOL_ISO <-->|Montage NFS Datastores| NAS_ISO
    PBS_ISO <-->|Sauvegardes NFSv3| NAS_ISO

    %% Liaisons Tailscale
    CLIENTS <--> HEADSCALE
    HEADSCALE --> LXC_PBS
    HEADSCALE --> VM_COOLIFY

    classDef host fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef guest fill:#1a2b3c,stroke:#0F6E56,color:#fff;
    classDef wan fill:#2c3e50,stroke:#34495e,color:#fff;
    class PVE_HOST,LXC_NAS,LXC_PBS,VM_COOLIFY host;
    class TRAEFIK,AUTH,VAULT,HOMEFLIX,HEADSCALE guest;
```

## État actuel de l'infrastructure

<Info>
**Infrastructure de Production IMS-WORLD.** Le Minisforum MS-01 (Proxmox VE 9) est le cœur de production. Le Mac Mini assure la fonction de standby / secours.
</Info>

### Matériel

| Nœud | Rôle | Statut |
|---|---|---|
| [**Labrax**](/infrastructure/labrax) | Rack serveur physique 3D, switch NETGEAR GS308EV4, alim PicoPSU | 🟢 Production |
| **Minisforum MS-01** | Hyperviseur Proxmox VE — hôte principal | 🟢 Production |
| **Mac Mini 2014** | Hôte de secours | 🟡 Standby |
| **Raspberry Pi 3B+** | Monitoring | 🟢 Actif |

### Guests Proxmox (MS-01)

| Guest | VMID | Rôle | Réseau |
|---|---|---|---|
| **IMS-NAS** | LXC 100 | Stockage (NFS + SMB) | `vmbr0` + `vmbr1` |
| **IMS-PBS** | LXC 103 | Sauvegardes (Proxmox Backup Server) | `vmbr0` + `vmbr1` + Tailscale |
| **IMS-Coolify** | VM 104 | Orchestration Docker — héberge tous les services applicatifs | `vmbr0` + `vmbr1` + Tailscale |

<Card title="Détail de chaque nœud" icon="server" href="/infrastructure/proxmox-host">
  Voir la section Infrastructure pour les spécifications complètes.
</Card>

### Services applicatifs en production

| Service | Domaine | Rôle |
|---|---|---|
| [Authentik](/services/authentik) | `auth.ims-world.fr` | SSO / OIDC |
| [Vaultwarden](/services/vaultwarden) | `vault.ims-world.fr` | Gestionnaire de mots de passe |
| [HomeFlix](/services/homeflix) | `homeflix.ims-world.fr` + 5 sous-domaines | Stack médias (Jellyfin, *arr, qBittorrent) |
| [Headscale + Headplane](/services/headscale-headplane) | `vpn.ims-world.fr` | Control plane Tailscale self-hosted |

### Services additionnels (Feuille de route)

| Service | Statut |
|---|---|
| [Cap](/services/cap) | En attente de déploiement |
| Beszel, Zipline, Forgejo, Photoprism, Immich, Ntfy, Home Assistant, Patrimo, Sentryx | En attente de déploiement |

## ⚡ Accès Rapides — Interfaces Homelab

<CardGroup cols={3}>
  <Card title="Proxmox VE GUI" icon="server" href="https://192.168.1.41:8006">
    Hyperviseur PVE MS-01 (`192.168.1.41:8006`)
  </Card>
  <Card title="Coolify Admin" icon="rocket" href="https://coolify.ims-world.fr">
    Orchestration Docker (`coolify.ims-world.fr`)
  </Card>
  <Card title="Authentik SSO" icon="key" href="https://auth.ims-world.fr">
    Provider OIDC & SSO (`auth.ims-world.fr`)
  </Card>
  <Card title="Vaultwarden" icon="shield-halved" href="https://vault.ims-world.fr">
    Coffre Mots de Passe (`vault.ims-world.fr`)
  </Card>
  <Card title="HomeFlix" icon="clapperboard" href="https://homeflix.ims-world.fr">
    Jellyfin Streaming (`homeflix.ims-world.fr`)
  </Card>
  <Card title="Headplane Admin" icon="network-wired" href="https://vpn.ims-world.fr/admin">
    Tailnet GUI (`vpn.ims-world.fr/admin`)
  </Card>
</CardGroup>

## Réseau en un coup d'œil

<Card title="Architecture réseau complète" icon="network-wired" href="/reseau/architecture-reseau">
  Bridges Proxmox (`vmbr0`/`vmbr1`), Tailscale/Headscale, DNS, port-forward — tout le détail réseau.
</Card>

## Navigation rapide

<CardGroup cols={2}>
  <Card title="Je dépanne un problème" icon="wrench" href="/procedures/depannage-courant">
    Tous les pièges récurrents déjà rencontrés et leur solution.
  </Card>
  <Card title="Plan de Reprise (PRA)" icon="shield-alert" href="/procedures/plan-reprise-activite-pra">
    Procédure de reconstruction complète en cas de crash du serveur MS-01.
  </Card>
  <Card title="Matrice de Sécurité" icon="shield-halved" href="/reseau/matrice-securite-exposition">
    Cartographie complète des accès, de l'exposition publique et du filtrage Tailnet.
  </Card>
  <Card title="Je déploie un nouveau service" icon="arrow-right-arrow-left" href="/procedures/deploiement-service">
    Le protocole standard affiné sur 4 migrations réelles.
  </Card>
  <Card title="J'ajoute un disque au NAS" icon="hard-drive" href="/procedures/ajout-nouveau-disque">
    Extension MergerFS ou bascule storage-hot.
  </Card>
  <Card title="Historique du projet" icon="clock-rotate-left" href="/history/chronologie">
    Chronologie complète, décisions et déviations par rapport au plan initial.
  </Card>
</CardGroup>
