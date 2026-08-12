---
title: "ADR-004 — Architecture Réseau en 3 Zones d'Exposition"
description: "Décision d'architecture concernant le cloisonnement et la matrice de sécurité du homelab"
icon: "network-wired"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-12)*

---

## Contexte

Exposer des services d'administration d'infrastructure (GUI Proxmox 8006, PBS 8007, qBittorrent, Radarr/Sonarr, Grafana) directement sur Internet présente un risque inacceptable de piratage ou d'attaque par déni de service. Il convenait d'établir une règle d'exposition claire et vérifiable pour chaque composant du homelab.

---

## Décision

Nous avons découpé l'ensemble du réseau en **3 zones de confiance étanches** :

1. **Zone 1 — Public WAN (Internet)** : Services ouverts sur le Web (`auth`, `vault`, `photos`, `homeflix`, `videoclub`, `status`, `tools`). Protégés par Traefik, certificat TLS Let's Encrypt DNS-01, et authentification systématique (SSO OIDC ou Forward-Auth Outpost).
2. **Zone 2 — Tailnet Overlay (VPN WireGuard `100.64.0.0/10`)** : Services d'administration applicative (`qbit`, `radarr`, `sonarr`, `prowlarr`, `monitoring`, `headplane`). Résolution DNS masquée sur OVH (`127.0.0.1`) et filtrage HTTP obligatoire par le middleware Traefik `vpn-only` (**HTTP 403 Forbidden** hors Tailnet).
3. **Zone 3 — Administration LAN & Bridge Isolé (`192.168.1.0/24` & `10.10.10.0/24`)** : Interfaces de gestion bas niveau des hyperviseurs (Proxmox 8006, PBS 8007, SSH 4242, SMB 445, NFS 2049). Accès strictement restreint au réseau physique local et au bridge virtuel NFS/Télémétrie `vmbr1`.

---

## Conséquences

### Positives
- **Surface d'attaque minimale** sur Internet (seuls les ports 80/443 de la Bbox sont transférés vers Traefik).
- **Protection en profondeur** : Même en cas de découverte d'un sous-domaine privé, le middleware Traefik `vpn-only` rejette la connexion HTTP avant d'atteindre l'application.
- **Résolution DNS étanche** : Aucun enregistrement DNS public ne révèle l'IP réelle des services privés Zone 2 (pointage sur `127.0.0.1`).

### Négatives / Contraintes
- Obligation d'être connecté au VPN Tailscale/Headscale pour administrer les arrs, la métrologie ou Headplane à distance.
