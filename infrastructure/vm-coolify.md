---
title: "VM IMS-Coolify (VM 104)"
description: "Orchestration Docker — héberge tous les services applicatifs"
icon: "cube"
iconType: "duotone"
---

<Badge color="green">🟢 Production Active (VM 104)</Badge>

<Note>
🖥️ **Type d'Instance** : **Machine Virtuelle QEMU/KVM 104** (Ubuntu 24.04 LTS) — Isolation complète du noyau avec hyperviseur dédié et kernel propre.
</Note>

## Rôle

Héberge Coolify et l'ensemble de la stack applicative (Authentik, Vaultwarden, HomeFlix, Headscale, etc). C'est le point d'entrée de production principal de l'infrastructure.

## Fiche technique

| Propriété | Valeur |
|---|---|
| **VMID** | 104 |
| **OS** | Ubuntu 24.04 LTS (clonée depuis template `9000`) |
| **CPU / RAM** | 6 cores / 18 Go RAM |
| **Disque** | 128 Go NVMe |
| **Réseau** | `vmbr0` (192.168.1.52/24) + `vmbr1` (10.10.10.2/24) + client Tailscale dédié |
| **Tailscale** | `100.64.0.4`, hostname `ims-pve-104-coolify` |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

## Coolify & Architecture Docker

```mermaid
graph TD
    subgraph VM ["🚀 VM 104 (IMS-Coolify — 6 vCPU / 18 Go RAM)"]
        subgraph FS ["Filesystem Local & ACL"]
            DATA["/data/coolify/services (ACL cmolotkoff:rwX)"]
            PROXY_CFG["/data/coolify/proxy (Traefik Dynamic Config)"]
        end

        subgraph DOCKER ["🐳 Moteur Docker & Services"]
            TRAEFIK["coolify-proxy (Traefik v3.7)"]
            AUTH["Authentik (SSO)"]
            VAULT["Vaultwarden"]
            HOMEFLIX["HomeFlix Stack (9 containers)"]
            HEADSCALE["Headscale + Headplane"]
        end

        subgraph NFS_MNT ["📁 Points de Montage NFS (via 10.10.10.1)"]
            MNT_STOR["/mnt/nas-storage (NFS -> /mnt/storage)"]
            MNT_HOT["/mnt/nas-hot (NFS -> /mnt/storage-hot)"]
        end
    end

    TRAEFIK --> AUTH
    TRAEFIK --> VAULT
    TRAEFIK --> HOMEFLIX
    TRAEFIK --> HEADSCALE

    HOMEFLIX --> MNT_STOR
    AUTH --> MNT_STOR
    HEADSCALE --> DATA

    classDef vm fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef docker fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef nfs fill:#1a2b3c,stroke:#0F6E56,color:#fff;
    class DATA,PROXY_CFG vm;
    class TRAEFIK,AUTH,VAULT,HOMEFLIX,HEADSCALE docker;
    class MNT_STOR,MNT_HOT nfs;
```

| Propriété | Valeur |
|---|---|
| **Version** | 4.1.2 |
| **URL** | `https://coolify.ims-world.fr` |
| **Accès legacy Mac Mini** | `http://coolify-old.ims-world.fr:8000` (temporaire, période de validation) |

## Cartographie des services Coolify (UUIDs & Chemins)

| Service | UUID Coolify | Chemin d'accès sur la VM | Statut |
|---|---|---|---|
| **Authentik** | `k5mxvc2r6c4zlb6j3d443h7b` | `/data/coolify/services/k5mxvc2r6c4zlb6j3d443h7b/` | <Badge color="green">🟢 Production</Badge> |
| **Vaultwarden** | `i5ae953riutbot9afjcboptb` | `/data/coolify/services/i5ae953riutbot9afjcboptb/` | <Badge color="green">🟢 Production</Badge> |
| **HomeFlix** (Jellyfin/Sonarr/Radarr/Prowlarr/qBit/Gluetun) | `w39uebmcnse7yctsft8hzed8` | `/data/coolify/services/w39uebmcnse7yctsft8hzed8/` | <Badge color="green">🟢 Production</Badge> |
| **Headscale + Headplane** | `i136ix2bmrrbeovnyrh1o72w` | `/data/coolify/services/i136ix2bmrrbeovnyrh1o72w/` | <Badge color="green">🟢 Production</Badge> |
| **Monitoring LGTM** (Grafana/Loki/Prometheus) | `rrw19kmye6gng961igtzqpgw` | `/data/coolify/services/rrw19kmye6gng961igtzqpgw/` | <Badge color="green">🟢 Production</Badge> |
| **Ntfy** (Push Notifications) | `j5akn2e9pr6g7c2pjvdj78w0` | `/data/coolify/services/j5akn2e9pr6g7c2pjvdj78w0/` | <Badge color="green">🟢 Production</Badge> |
| **Dozzle** (Logs Docker Live) | `ejdn7jiuwiyixrmp8nffjkcj` | `/data/coolify/services/ejdn7jiuwiyixrmp8nffjkcj/` | <Badge color="green">🟢 Production</Badge> |
| **Immich** (Médiathèque Photo/Vidéo IA) | `p3ujda5c7sc8nf4j9zzd8lck` | `/data/coolify/services/p3ujda5c7sc8nf4j9zzd8lck/` | <Badge color="green">🟢 Production</Badge> |
| **IT-Tools** (smoke test) | `yefujwl3pxvum45edpsbsru7` | `/data/coolify/services/yefujwl3pxvum45edpsbsru7/` | <Badge color="gray">⚪ Actif</Badge> |

## Stockage NFS

```bash
10.10.10.1:/mnt/storage      /mnt/nas-storage  nfs  defaults,nofail,_netdev  0 0
10.10.10.1:/mnt/storage-hot  /mnt/nas-hot      nfs  defaults,nofail,_netdev  0 0
```

<Check>
Contrairement à PBS (LXC unprivileged), le montage NFS sur une VM fonctionne sans restriction — accès kernel complet, aucun des blocages user-namespace rencontrés ailleurs.
</Check>

## Accès filesystem Coolify — ACL

Pour éviter `sudo` à chaque commande sur `/data/coolify/services/` et `/data/coolify/proxy/` :

```bash
sudo setfacl -R -m u:cmolotkoff:rwX /data/coolify/services
sudo setfacl -R -d -m u:cmolotkoff:rwX /data/coolify/services
sudo setfacl -m u:cmolotkoff:--x /data/coolify        # traversée du parent uniquement
sudo setfacl -R -m u:cmolotkoff:rwX /data/coolify/proxy
sudo setfacl -R -d -m u:cmolotkoff:rwX /data/coolify/proxy
```

<Tip>
`chown` évité volontairement — casserait potentiellement des vérifications de permissions internes à Coolify (user `9999`).
</Tip>

## Docker sans sudo

```bash
sudo usermod -aG docker cmolotkoff
# reconnexion nécessaire, ou `newgrp docker`
```

<Warning>
Être dans le groupe `docker` équivaut en pratique à un accès root sur la machine. Acceptable ici (seul admin), à garder en tête si un autre utilisateur devait accéder à cette VM.
</Warning>

---

## ⚠️ Règle d'Or : Routage Traefik Coolify vs Labels Manuels

<Warning>
**Règle de Routage Traefik & Coolify** : 

L'astuce d'utilisation du champ *Domains* dans l'UI Coolify (*"laisser Coolify gérer le routeur Traefik"*) est **exclusivement réservée aux services standards sans middleware sur-mesure** (sans `vpn-only`, sans forward-auth).

Dès qu'un service nécessite un middleware Traefik spécifique (ex: filtrage d'accès IP `vpn-only` pour les services Tailscale-only), **il faut impérativement ré-expliciter les labels Traefik manuellement dans le fichier Compose (`docker-compose.yml`) et laisser le champ Domains vide dans l'UI Coolify**. Sinon, Coolify génère un routeur automatique parallèle sans middleware, ce qui ré-expose publiquement le service.
</Warning>

