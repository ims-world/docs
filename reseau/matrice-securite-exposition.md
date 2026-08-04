---
title: "Matrice de Sécurité & d'Exposition"
description: "Cartographie centralisée des zones de confiance, de l'exposition publique, du filtrage et de l'authentification"
---

## Philosophie de Sécurité (Sécurité en Profondeur)

<Info>
L'infrastructure IMS-WORLD applique le principe de **moindre privilège** et de **sécurité en profondeur** : aucun service n'est exposé publiquement s'il n'en a pas le besoin strict. Le trafic d'administration est systématiquement restreint au réseau privé Tailscale ou au LAN local.
</Info>

---

## 🛡️ Schéma des 3 Zones de Confiance Réseau

```mermaid
graph TB
    subgraph ZONE_WAN ["🌐 Zone 1 — Public WAN (Internet)"]
        PUBLIC_USERS["Utilisateurs Externes"]
        HTTPS_443["Ports 80/443 (Port-Forward Bbox -> Traefik)"]
    end

    subgraph ZONE_TAILNET ["🔐 Zone 2 — Tailnet Overlay (Headscale 100.64.0.0/10)"]
        VPN_USERS["Appareils Authentifiés Tailnet"]
        MIDDLEWARE_VPN["Traefik Middleware: vpn-only (ipAllowList)"]
    end

    subgraph ZONE_LAN ["🏠 Zone 3 — LAN Interne & Virtual Bridges (192.168.1.0/24 & 10.10.10.0/24)"]
        ADMIN_LAN["Admin SSH (Port 4242) / GUI Proxmox (8006)"]
        NFS_ISO["Bridge vmbr1 Isolé (Trafic NFS Interne Guests)"]
    end

    subgraph SERVICES ["🐳 Services Applicatifs Docker (VM 104)"]
        PUB_APPS["Authentik (auth) / Vaultwarden (vault) / Jellyfin (homeflix)"]
        PRIV_APPS["qBittorrent (qbit) / Radarr / Sonarr / Prowlarr"]
        ADMIN_NODES["Host PVE (41) / NAS LXC (50) / PBS LXC (51)"]
    end

    PUBLIC_USERS --> HTTPS_443
    HTTPS_443 --> PUB_APPS

    VPN_USERS --> MIDDLEWARE_VPN
    MIDDLEWARE_VPN --> PRIV_APPS

    ADMIN_LAN --> ADMIN_NODES
    ADMIN_NODES <--> NFS_ISO

    classDef wan fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef vpn fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef lan fill:#1a2b3c,stroke:#0F6E56,color:#fff;
    class PUBLIC_USERS,HTTPS_443 wan;
    class VPN_USERS,MIDDLEWARE_VPN vpn;
    class ADMIN_LAN,NFS_ISO lan;
    class PUB_APPS,PRIV_APPS,ADMIN_NODES services;
```

---

## 📊 Matrice d'Exposition et d'Authentification des Services

| Service | Domaine / Adresse | Exposition | Méthode d'Authentification | Protections & Middlewares |
|---|---|---|---|---|
| **Authentik** | `auth.ims-world.fr` | 🌐 Public WAN | SSO OIDC + WebAuthn 2FA | Let's Encrypt DNS-01, TLS 1.3 |
| **Vaultwarden** | `vault.ims-world.fr` | 🌐 Public WAN | SSO Authentik + Mdp fort | Config `email_verified: true`, TLS 1.3 |
| **Jellyfin** | `homeflix.ims-world.fr` | 🌐 Public WAN | Auth native Jellyfin | DNS-01 TLS, Transcodage isolé |
| **Jellyseerr** | `videoclub.ims-world.fr` | 🌐 Public WAN | SSO / Auth Jellyfin | DNS-01 TLS |
| **Headscale** | `vpn.ims-world.fr` | 🌐 Public WAN | OIDC Authentik SSO + Noise Key | Protocol Noise encryption |
| **Headplane** | `vpn.ims-world.fr/admin` | 🔐 Tailnet-only | OIDC Authentik SSO | Middleware `vpn-only` |
| **qBittorrent** | `qbit.ims-world.fr` | 🔐 Tailnet-only | Auth native (`HostHeaderValidation=false`) | Middleware `vpn-only`, Gluetun VPN Tunnel |
| **Prowlarr** | `prowlarr.ims-world.fr` | 🔐 Tailnet-only | Auth native / API Key | Middleware `vpn-only` |
| **Radarr** | `radarr.ims-world.fr` | 🔐 Tailnet-only | Auth native / Formulaire | Middleware `vpn-only` |
| **Sonarr** | `sonarr.ims-world.fr` | 🔐 Tailnet-only | Auth native / Formulaire | Middleware `vpn-only` |
| **IT-Tools** | `it.ims-world.fr` | 🔐 Tailnet-only | Aucune (Outil stateless) | Middleware `vpn-only` |
| **Cap** | `cap.ims-world.fr` | ⏸️ En Pause | NextAuth / MySQL Keys | Chiffrement `DATABASE_ENCRYPTION_KEY` |
| **Proxmox VE GUI** | `192.168.1.41:8006` | 🏠 LAN / Tailnet | PAM / Compte nominatif `cmolotkoff` | HTTPS Auto-signé, Port restreint |
| **PBS Web GUI** | `192.168.1.51:8007` | 🏠 LAN / Tailnet | Auth PBS `cmolotkoff@pbs` | HTTPS Auto-signé |
| **NAS SMB** | `192.168.1.50:445` | 🏠 LAN-only | Auth SMB `cmolotkoff` | Bloqué hors du LAN |
| **NAS NFS** | `10.10.10.1:2049` | 🔒 `vmbr1` Isolé | Restriction IP Subnet `10.10.10.0/24` | `no_root_squash` restreint aux guests |

---

## 🔒 Règles de Sécurité Impératives

<Warning>
**Port SSH 4242** : Le serveur SSH réel écoute sur le port **4242**. Le port 22 est occupé par **Endlessh** (tarpit piège à bots). Ne jamais ouvrir le port 22 ou 4242 vers le WAN public.
</Warning>

<Warning>
**Protection Iptables / Docker Bypass** : Docker contourne par défaut les règles UFW/iptables standards. La chaîne `DOCKER-USER` est configurée pour forcer le respect des restrictions IP.
</Warning>

<Tip>
Pour vérifier à tout moment qu'un service restreint n'est pas accessible depuis l'extérieur, exécuter un test d'accès WAN :
`curl -Iv https://qbit.ims-world.fr` (Doit retourner un HTTP **403 Forbidden** via le middleware `vpn-only`).
</Tip>
