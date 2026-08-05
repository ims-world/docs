---
title: "Vue d'ensemble"
description: "Infrastructure self-hosted IMS-WORLD — architecture, principes et état actuel"
---

![Architecture Homelab IMS-WORLD — Vue Globale](/assets/hero-banner.png)

<CardGroup cols={4}>
  <Card title="1 Hôte Proxmox" icon="server" href="/infrastructure/proxmox-host">
    Minisforum MS-01 (12 Cores / 16 Threads / 32 Go RAM)
  </Card>
  <Card title="2 LXC & 1 VM" icon="box" href="/infrastructure/vm-coolify">
    IMS-NAS (100), IMS-PBS (103), VM Coolify (104)
  </Card>
  <Card title="100% Chiffré" icon="shield-check" href="/reseau/matrice-securite-exposition">
    ACME DNS-01 & Headscale Tailnet
  </Card>
  <Card title="0€ SaaS / Mois" icon="euro-sign" href="/history/chronologie">
    100% Self-Hosted & matériel possédé
  </Card>
</CardGroup>

## Principes Directeurs

L'infrastructure IMS-WORLD repose sur trois principes non négociables :

<CardGroup cols={3}>
  <Card title="100% Local" icon="house">
    Aucune dépendance à un cloud tiers pour les services critiques. Auto-hébergement complet sur le cluster physique.
  </Card>
  <Card title="0€ Récurrent" icon="euro-sign">
    Uniquement du matériel possédé et des logiciels open-source. Aucun abonnement SaaS.
  </Card>
  <Card title="Lean & Efficient" icon="feather">
    Complexité activement challengée. Pas de sur-ingénierie — chaque brique ajoutée doit se justifier.
  </Card>
</CardGroup>

## 🗄️ Infrastructure Physique & Rack 3D

<CardGroup cols={2}>
  <Card title="Châssis & Rack Labrax 3D" icon="cubes" href="/infrastructure/labrax">
    Châssis 3D-printé, ventilateur Noctua G2, switch NETGEAR et caddies 3.5" Dell.
  </Card>
  <Card title="Afficheur Kiosk & Module 2U" icon="display" href="/infrastructure/rpi-monitor">
    Raspberry Pi 3B+, écrans Wisecoco 7.84" LCD + OLED 0.91" et bouton poussoir GPIO.
  </Card>
</CardGroup>

## Schéma d'Architecture Macro Globale

```mermaid
graph TB
    subgraph INGRESS ["🌐 Accès Externe & DNS"]
        USERS["Clients Internet"]
        DNS_OVH["DNS Public OVH (*.ims-world.fr)"]
        LE_ACME["Let's Encrypt (DNS-01 ACME)"]
    end

    subgraph BBOX_NET ["📡 Bbox Routeur WAN"]
        BBOX["Port-Forward 80/443 (192.168.1.52)"]
    end

    subgraph TAILNET_ZONE ["🔐 Network Overlay (Tailscale 100.64.0.0/10)"]
        VPN_CLIENTS["Appareils Authentifiés Tailnet"]
        HEADPLANE["Headplane GUI (vpn.ims-world.fr/admin)"]
        MIDDLEWARE_VPN["Traefik Middleware vpn-only"]
    end

    subgraph PVE_CLUSTER ["🖥️ Hyperviseur Proxmox MS-01 (192.168.1.41)"]
        PVE_HOST["Proxmox VE 9.2.3 Host (192.168.1.41)"]
        NAS_LXC["IMS-NAS (LXC 100 — 192.168.1.50)"]
        PBS_LXC["IMS-PBS (LXC 103 — 192.168.1.51)"]
        VM_COOLIFY["IMS-Coolify (VM 104 — 192.168.1.52)"]
    end

    subgraph ISO_NET ["🔒 Bridge NFS Isolé (vmbr1: 10.10.10.0/24)"]
        NFS_NAS["Export NAS NFS (10.10.10.1)"]
        NFS_PBS["PBS Datastore (10.10.10.3)"]
        NFS_VM["Montage VM Coolify (10.10.10.2)"]
    end

    subgraph SATELLITES ["🗄️ Nœuds Satellites & Display"]
        MAC_MINI["Mac Mini Standby (100.64.0.7)"]
        RPI_MON["Raspberry Pi Kiosk (100.64.0.12)"]
    end

    USERS --> DNS_OVH
    DNS_OVH --> BBOX
    BBOX --> VM_COOLIFY
    LE_ACME --- VM_COOLIFY

    VPN_CLIENTS --> HEADPLANE
    VPN_CLIENTS --> MIDDLEWARE_VPN
    MIDDLEWARE_VPN --> VM_COOLIFY

    NFS_VM <--> NFS_NAS
    NFS_PBS <--> NFS_NAS
    PVE_HOST -.-> PBS_LXC

    RPI_MON -.-> PVE_HOST
    MAC_MINI -.-> BBOX

    classDef wan fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef vpn fill:#F97316,stroke:#FB923C,color:#fff;
    classDef host fill:#1a2b3c,stroke:#F97316,color:#fff;
    class USERS,DNS_OVH,BBOX wan;
    class VPN_CLIENTS,HEADPLANE vpn;
    class PVE_HOST,NAS_LXC,PBS_LXC,VM_COOLIFY host;
```

## État Actuel des Composants

<Tabs>
  <Tab title="🟢 Services en Production">
    | Service | Domaine | Description | Page associée |
    |---|---|---|---|
    | **Authentik** | `auth.ims-world.fr` | Provider d'identité centralisé (SSO / OIDC + 2FA) | [Authentik](/services/authentik) |
    | **Vaultwarden** | `vault.ims-world.fr` | Coffre-fort de mots de passe compatible Bitwarden | [Vaultwarden](/services/vaultwarden) |
    | **HomeFlix** | `homeflix.ims-world.fr` | Stack médias complète (Jellyfin, *arr, qBittorrent, Gluetun) | [HomeFlix](/services/homeflix) |
    | **Headscale + Headplane** | `vpn.ims-world.fr` | Control plane VPN Tailscale self-hosted | [Headscale](/services/headscale-headplane) |
  </Tab>
  <Tab title="🖥️ Infrastructure Physique & Hyperviseur">
    | Composant | Rôle | Statut | Page associée |
    |---|---|---|---|
    | **Labrax** | Rack serveur physique 3D, switch NETGEAR, alim PicoPSU | 🟢 Production | [Rack Labrax](/infrastructure/labrax) |
    | **Minisforum MS-01** | Hyperviseur Proxmox VE 9.2.3 —  hôte principal | 🟢 Production | [Proxmox Host](/infrastructure/proxmox-host) |
    | **Mac Mini 2014** | Hôte de secours | 🟡 Standby | [Mac Mini](/infrastructure/mac-mini) |
    | **Raspberry Pi 3B+** | Affichage Kiosk & Module 2U | 🟢 Production | [RPi Monitor](/infrastructure/rpi-monitor) |
    | **IMS-NAS (LXC 100)** | Stockage NFS + SMB (MergerFS) | 🟢 Production | [IMS-NAS](/infrastructure/ims-nas) |
    | **IMS-PBS (LXC 103)** | Sauvegardes Proxmox Backup Server | 🟢 Production | [IMS-PBS](/infrastructure/ims-pbs) |
    | **IMS-Coolify (VM 104)** | Orchestration Docker (Traefik v3.7) | 🟢 Production | [VM Coolify](/infrastructure/vm-coolify) |
  </Tab>
</Tabs>

## Guides Opérationnels & Procédures

<CardGroup cols={2}>
  <Card title="Je dépanne un problème" icon="wrench" href="/procedures/depannage-courant">
    Tous les pièges récurrents déjà rencontrés et leur solution.
  </Card>
  <Card title="Plan de Reprise (PRA / DRP)" icon="shield-alert" href="/procedures/plan-reprise-activite-pra">
    Procédure d'urgence et reconstruction intégrale en cas de sinistre.
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
  <Card title="Feuille de Route & TODOs" icon="list-check" href="/procedures/roadmap">
    Liste des chantiers techniques et optimisations à venir.
  </Card>
  <Card title="Historique du projet" icon="clock-rotate-left" href="/history/chronologie">
    Chronologie complète, décisions et déviations par rapport au plan initial.
  </Card>
</CardGroup>
