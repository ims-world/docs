---
title: "ADR-009 — Bug du Middleware Traefik vpn-only lié au userland-proxy Docker"
description: "Masquage des IP sources par le proxy Docker utilisateur (10.0.1.1) et stratégies d'atténuation"
icon: "shield-exclamation"
iconType: "duotone"
---

## Statut
<Badge color="amber">🟡 En Analyse & Contournement Applicatif</Badge> *(2026-08-18)*

---

## Contexte

Le middleware Traefik `vpn-only` (`ipAllowList: 100.64.0.0/10`, défini dans `/data/coolify/proxy/dynamic/vpn-only.yaml`) vise à restreindre l'accès à certaines interfaces privées (`qbit`, `radarr`, `sonarr`, `prowlarr`, `monitoring`) aux seuls clients connectés au réseau VPN Overlay Tailscale (`100.64.0.0/10`).

Le 18 août 2026, un dysfonctionnement a été identifié : **le middleware bloque systématiquement 100% des requêtes entrantes (y compris les requêtes légitimes émanant du Tailnet)** en renvoyant une réponse **HTTP 403 Forbidden (9 octets)**.

---

## Cause Racine (Root Cause)

Le démon Docker sur l'hôte (VM Coolify) utilise par défaut le composant **`userland-proxy`** (`docker-proxy`) pour transférer le trafic des ports publiés (80/443). 

1. `docker-proxy` accepte la connexion TCP sur l'interface de l'hôte (ex: `tailscale0` ou `vmbr0`).
2. Au lieu de faire du NAT direct au niveau du noyau (iptables) qui préserverait l'IP d'origine (`100.64.0.x`), `docker-proxy` ouvre une **seconde connexion TCP séparée** vers le conteneur `coolify-proxy` (Traefik).
3. Cette connexion est sourcée avec l'adresse IP de la passerelle du bridge Docker (ex: **`10.0.1.1`** pour le réseau bridge Coolify `br-765ae81efc77`).
4. **Conséquence** : Dans `docker logs coolify-proxy`, Traefik évalue `ClientAddr: 10.0.1.1` pour l'intégralité des requêtes HTTP/HTTPS entrantes, quelle que soit leur provenance réelle (Tailnet, LAN ou WAN).

### Preuves Diagnostic
- `curl -v https://monitoring.ims-world.fr` → **HTTP 403 Forbidden** (corps de 9 octets, signature exacte de Traefik `ipAllowList`).
- `tcpdump -i tailscale0` → Le paquet réseau sous-jacent arrive bien avec l'IP source client `100.64.0.3`.
- `docker logs coolify-proxy` → Traefik enregistre `ClientAddr: 10.0.1.1` et rejette la requête car `10.0.1.1` n'est pas inclus dans la plage `100.64.0.0/10`.

---

## Décisions & Stratégie d'Atténuation

### 1. Retrait immédiat de `vpn-only` sur Grafana (`monitoring.ims-world.fr`)
Pour rétablir l'accès à Grafana sans compromettre la sécurité, le middleware `vpn-only` a été retiré de son routeur et l'accès a été sécurisé par **l'authentification SSO OIDC Authentik** (`auth.ims-world.fr`).

### 2. Maintien temporaire sur la Stack Media (`qbit`, `radarr`, `sonarr`, `prowlarr`)
Le middleware `vpn-only` est conservé pour le moment sur la stack media. Les services restent protégés par leurs authentifications applicatives natives (formulaire/mot de passe/clé API), mais le filtrage d'IP réseau par ipAllowList demeure inopérant tant que `userland-proxy` intercepte le trafic.

### 3. Report de la modification système `"userland-proxy": false`
La désactivation globale du proxy utilisateur dans `/etc/docker/daemon.json` (`"userland-proxy": false`) permettrait à iptables de préserver les IP sources d'origine. Cependant, cette modification a été **reportée à une fenêtre de maintenance dédiée** car elle exige :
- Un redémarrage complet du démon Docker de la VM Coolify (coupure momentanée de tous les conteneurs).
- Une vérification approfondie post-redémarrage de la connectivité inter-conteneurs sur la vingtaine de réseaux bridges Docker (`br-*`).

---

## Plan d'Action (Roadmap)

<Steps>
  <Step title="Vérification Formelle Stack Media">
    Tester l'impact exact du blocage sur les interfaces web de qBittorrent, Sonarr, Prowlarr et Radarr.
  </Step>
  <Step title="Fenêtre de Maintenance userland-proxy">
    Tester `"userland-proxy": false` dans `/etc/docker/daemon.json` sur la VM Coolify et contrôler les tables iptables + la connectivité inter-services.
  </Step>
  <Step title="Alternative SSO/Forward-Auth (Option de Secours)">
    Si la désactivation de `userland-proxy` provoque des régressions sur les bridges Docker, généraliser l'authentification Forward-Auth Authentik sur les 4 services d'administration media.
  </Step>
</Steps>
