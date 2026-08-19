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

1. **Zone 1 — Public WAN (Internet)** : Services ouverts sur le Web (`auth`, `vault`, `photos`, `homeflix`, `videoclub`, `status`, `tools`). Protégés par Traefik, certificat TLS Let's Encrypt DNS-01, et authentification système (SSO OIDC ou Forward-Auth Outpost).
2. **Zone 2 — Tailnet Overlay (VPN WireGuard `100.64.0.0/10`)** : Services d'administration applicative (`qbit`, `radarr`, `sonarr`, `prowlarr`, `monitoring`, `headplane`). Filtrage HTTP obligatoire par le provider file Traefik `vpn-only.yaml` (**HTTP 403 Forbidden** hors Tailnet) avec préservation d'IP source via `"userland-proxy": false` (CIS Docker Benchmark). Voir l'[ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only).
3. **Zone 3 — Administration LAN & Tailnet Direct (`192.168.1.0/24` & `100.64.0.0/10`)** : Interfaces de gestion bas niveau des hyperviseurs et accès SSH système (Proxmox GUI 8006, PBS GUI 8007, SSH 4242/22, SMB 445, NFS 2049 sur `10.10.10.0/24`). Accès SSH direct possible depuis le LAN et le Tailnet, avec **0 exposition sur le WAN Bbox**.

---

## Conséquences

### Positives
- **Surface d'attaque minimale** sur Internet (seuls les ports 80/443 de la Bbox sont transférés vers Traefik, ainsi que le port 2222 pour Forgejo Git SSH).
- **Protection en profondeur** : Même en cas de découverte d'un sous-domaine privé, le provider file Traefik `vpn-only.yaml` rejette la connexion HTTP avec un **403 Forbidden** avant d'atteindre l'application.
- **Accès Admin Sécurisé** : L'administration SSH et l'accès aux interfaces GUI Proxmox/PBS restent réservés au LAN local et au réseau virtuel Tailscale.

### Négatives / Contraintes
- Obligation d'être connecté au VPN Tailscale/Headscale pour administrer les arrs, la métrologie, Headplane ou accéder en SSH à distance.
