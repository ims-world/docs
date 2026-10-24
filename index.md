---
title: "Vue d'ensemble"
description: "Homelab auto-hébergé IMS-WORLD : deux nœuds Proxmox, une vingtaine de services derrière Traefik, accès d'administration par tailnet."
last_reviewed: "2026-10-25"
---

![Architecture du homelab IMS-WORLD](/assets/hero-banner.png)

## Principes

| Principe | Application |
| --- | --- |
| Auto-hébergé par défaut | Les services tournent sur du matériel possédé. Les services externes sont listés et justifiés : DNS et certificats (OVH, Let's Encrypt), VPN de téléchargement (ProtonVPN), relais de courriels (Resend). |
| Sobriété | Chaque brique ajoutée doit se justifier. Pas d'outil pour un usage rare : une procédure suffit. |
| Rien d'exposé sans protection | Tout accès public passe par Traefik et CrowdSec, avec une authentification. L'administration n'est joignable que depuis le LAN ou le tailnet. |
| État décrit et reproductible | La configuration des hôtes est décrite dans le dépôt [ims-ops](/procedures/administration-ims-ops) ; les choix sont consignés dans les [décisions d'architecture](/history/adr/index). |

## Architecture

```mermaid
%%{init: {'flowchart': {'defaultRenderer': 'elk'}}}%%
flowchart LR
  internet[Internet] --> bbox[Bbox<br/>80, 443, 2222]
  tailnet[Appareils du tailnet] --> traefik
  bbox --> traefik[Traefik et CrowdSec<br/>VM 104]
  traefik --> apps[Services Docker<br/>VM 104 et worker 105]
  subgraph pve[pve · MS-01]
    nas[(ims-nas<br/>LXC 100)]
    pbs[ims-pbs<br/>LXC 103]
    vm104[ims-coolify<br/>VM 104]
  end
  subgraph macmini[pve-macmini · Mac Mini]
    worker[worker Coolify<br/>LXC 105]
    haos[Home Assistant<br/>VM 106]
  end
  apps --- vm104
  apps --- worker
  vm104 -->|NFS| nas
  pbs -->|NFS| nas
  class internet,tailnet externe
  class bbox,traefik reseau
  class apps service
  class nas,pbs,vm104,worker,haos hote
  classDef hote fill:#e6eef6,stroke:#2f5d8a,color:#1b2129
  classDef service fill:#e8f3eb,stroke:#2e6b3f,color:#1b2129
  classDef reseau fill:#fdf2e3,stroke:#a35a00,color:#1b2129
  classDef externe fill:#f1f2f4,stroke:#6b7480,color:#1b2129,stroke-dasharray:4 3
```

## Composants

| Composant | Rôle | Page |
| --- | --- | --- |
| Labrax | Rack 10 pouces imprimé en 3D, switch, alimentation | [Labrax](/infrastructure/labrax) |
| `pve` (Minisforum MS-01) | Nœud principal du cluster `ims-cluster` | [MS-01](/infrastructure/proxmox-host) |
| `pve-macmini` (Mac Mini Late 2012) | Second nœud : worker Coolify et Home Assistant | [Mac Mini](/infrastructure/mac-mini) |
| Raspberry Pi 3B+ | Écrans du rack (éteint) | [Raspberry Pi](/infrastructure/rpi-monitor) |
| `ims-nas` (LXC 100) | Stockage : NFS pour les invités, SMB pour le LAN | [ims-nas](/infrastructure/ims-nas) |
| `ims-pbs` (LXC 103) | Proxmox Backup Server | [ims-pbs](/infrastructure/ims-pbs) |
| `ims-coolify` (VM 104) | Coolify, Traefik et la plupart des services | [VM 104](/infrastructure/vm-coolify) |
| `ims-coolify-worker` (LXC 105) | Nœud d'exécution Docker distant de Coolify | [Worker 105](/infrastructure/lxc-coolify-worker) |
| Home Assistant (VM 106) | Domotique | [Home Assistant](/infrastructure/vm-home-assistant) |

Les services sont classés par usage dans la navigation : identité et accès, supervision, médias, outils.

## Points d'entrée

<CardGroup cols={2}>
  <Card title="Dépanner" icon="wrench" href="/procedures/depannage-courant">
    Pièges déjà rencontrés et leur solution.
  </Card>
  <Card title="Accès de secours" icon="life-ring" href="/procedures/commandes-urgence">
    Tailnet ou Termix indisponible, commandes d'urgence.
  </Card>
  <Card title="Administrer le parc" icon="terminal" href="/procedures/administration-ims-ops">
    Configurer un hôte, créer une VM, gérer les sauvegardes.
  </Card>
  <Card title="Exposition des services" icon="shield-halved" href="/reseau/matrice-securite-exposition">
    Qui accède à quoi, et par quel chemin.
  </Card>
  <Card title="Plan de reprise" icon="rotate-left" href="/procedures/plan-reprise-activite-pra">
    Reconstruction après la perte d'un nœud ou d'un disque.
  </Card>
  <Card title="Feuille de route" icon="list-check" href="/roadmap">
    Chantiers ouverts et réalisés.
  </Card>
</CardGroup>
