---
title: "Headscale + Headplane"
description: "Control plane Tailscale self-hosted, avec interface d'administration"
icon: "network-wired"
iconType: "duotone"
---

## Fiche service

| Propriété | Valeur |
|---|---|
| **Domaine** | `vpn.ims-world.fr` |
| **Versions** | `headscale/headscale:v0.28.0` + `ghcr.io/tale/headplane:0.6.2` |
| **Base de données** | SQLite |
| **UUID Coolify** | `i136ix2bmrrbeovnyrh1o72w` |
| **Chemin** | `/data/coolify/services/i136ix2bmrrbeovnyrh1o72w/` |
| **Statut** | 🟢 Production |

## Architecture Headscale & Mesh VPN (WireGuard Overlay)

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
        PBS["PBS Storage (100.64.0.8)"]
        COOL["Coolify VM (100.64.0.10)"]
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
    HEADSCALE -.->|MagicDNS & Coordination WireGuard| MOBILE

    MAC <==>|Tunnels Directs Peer-to-Peer WireGuard| COOL
    PVE <==>|Tunnels Directs Peer-to-Peer WireGuard| COOL
    MOBILE <==>|Tunnels Directs Peer-to-Peer WireGuard| PBS

    classDef srv fill:#0F6E56,stroke:#16A085,color:#fff;
    classDef key fill:#2c3e50,stroke:#34495e,color:#fff;
    classDef node fill:#1a2b3c,stroke:#0F6E56,color:#fff;
    class HEADSCALE,HEADPLANE,AUTH_SRV srv;
    class NOISE_KEY,DB_SQLITE key;
    class MAC,PVE,PBS,COOL,MOBILE node;
```

<Warning>
Headscale est un control plane qui coordonne des connexions WireGuard peer-to-peer (Mesh VPN), et non un VPN centralisé classique.
</Warning>

## Identité Cryptographique & Clés Critiques

<Warning>
Le `server_url` (`vpn.ims-world.fr`) est **l'identité même du serveur**, gravée dans le protocole Noise et attendue par tous les clients du tailnet.
</Warning>

## Éléments critiques à préserver à l'identique

| Fichier | Rôle | Conséquence si changé |
|---|---|---|
| `noise_private.key` | Identité cryptographique du serveur | Tous les clients verraient un "nouveau serveur" non reconnu |
| `db.sqlite` | Nœuds, utilisateurs, clés | Perte de tous les appareils enregistrés |

<Info>
La clé API Headplane (communication interne Headplane↔Headscale) peut être régénérée si nécessaire :
```bash
docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale apikeys create --expiration 999d
```
</Info>

## Piège récurrent — dossier fantôme (3ème occurrence)

<Warning>
Coolify pré-crée un dossier vide à l'emplacement attendu d'un fichier de config avant toute copie. Ici plus retors qu'avec Authentik/Vaultwarden : `cp` vers un dossier existant a copié le fichier **dedans** au lieu de remplacer (erreur silencieuse). Pire — une fois qu'un **container a démarré** avec le mauvais mapping dossier↔fichier, un simple `docker restart` ne suffit PAS à corriger : le montage reste figé. Il faut une vraie recréation du container.

```bash
# Vérification systématique AVANT tout premier démarrage
file <chemin_config_attendu>   # doit dire "ASCII text", pas "directory"
```
</Warning>

## Cascade de blocages lors du cutover (02/08/2026)

<Steps>
  <Step title="502 Bad Gateway">
    Label `traefik.docker.network=coolify` manquant sur Headscale — même piège que sur Gluetun (HomeFlix), corrigé.
  </Step>
  <Step title="Port-forward pointait vers le mauvais hôte">
    La règle Bbox ciblait l'IP du **host Proxmox** (192.168.1.41, qui n'écoute que sur 8006) au lieu de la VM Coolify (192.168.1.52). Jamais remarqué avant puisque rien n'était encore basculé publiquement.
  </Step>
  <Step title="Crash-loop perpétuel">
    `only_start_if_oidc_is_available: true` bloque le démarrage tant qu'Authentik n'est pas joignable AVEC un certificat valide. Cascade visible : timeout d'abord, puis erreur de certificat TLS (`certificate valid for *.traefik.default, not auth.ims-world.fr`) — le port-forward corrigé route tout le trafic public vers le MS-01, y compris pour des domaines sans router configuré, qui reçoivent le certificat par défaut de Traefik.

    ```yaml
    only_start_if_oidc_is_available: false
    ```
  </Step>
</Steps>

<Warning>
`only_start_if_oidc_is_available` reste actuellement à `false` — **à repasser en `true`** maintenant que `auth.ims-world.fr` est stable sur le MS-01, pour retrouver le comportement de sécurité d'origine.
</Warning>

## Accès de secours au Mac Mini pendant la validation

```yaml
extra_records:
  - name: "coolify-old.ims-world.fr"
    value: "100.64.0.7"
```

À retirer une fois la période de validation terminée et le Mac Mini décommissionné.
