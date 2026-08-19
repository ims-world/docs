---
title: "Incident — Rejet HTTP 403 du Middleware vpn-only (docker-proxy)"
description: "Substitution d'IP source en 10.0.1.1 par le userland-proxy Docker causant le blocage de tout le trafic HTTP/HTTPS"
icon: "shield-exclamation"
iconType: "duotone"
---

<Badge color="green">🟢 Résolu & Implémenté via CIS Benchmark</Badge> *(2026-08-19)*

---

## Symptômes

Lors des tests d'accès aux services privés d'administration applicative (`qbit`, `radarr`, `sonarr`, `prowlarr`, `monitoring`), l'intégralité des requêtes émanant des clients légitimes connectés au VPN Overlay Tailscale (`100.64.0.0/10`) a été rejetée avec l'erreur :

```text
HTTP/1.1 403 Forbidden
Content-Length: 9

Forbidden
```

---

## Cause Racine (Root Cause)

Le démon Docker sur l'hôte (VM IMS-Coolify) s'exécute avec le composant **`userland-proxy`** (`docker-proxy`) activé par défaut.

1. Lorsqu'une connexion TCP s'établit sur un port publié (80/443), `docker-proxy` intercepte le trafic au niveau applicatif plutôt qu'au niveau paquet iptables.
2. `docker-proxy` ouvre une nouvelle connexion TCP vers le conteneur Traefik (`coolify-proxy`), sourcée avec l'adresse IP de la passerelle Docker bridge (ex: **`10.0.1.1`** pour `br-765ae81efc77`).
3. Traefik reçoit donc `ClientAddr: 10.0.1.1` pour l'ensemble du trafic entrant (WAN, LAN ou Tailnet).
4. Le middleware `vpn-only` (`ipAllowList: 100.64.0.0/10`) évalue `10.0.1.1` hors de la plage autorisée et rejette systématiquement les requêtes.

---

## Preuves Diagnostic & Traces

```bash
# 1. Test Client depuis le Tailnet — Rejet 403 immédiat
curl -v https://monitoring.ims-world.fr
# < HTTP/1.1 403 Forbidden
# < Content-Length: 9
# Forbidden

# 2. Capture de Paquets sur l'interface Tailscale (VM Coolify)
tcpdump -i tailscale0 port 443
# Confirme : Le paquet IP d'origine arrive bien avec l'IP source 100.64.0.3

# 3. Inspecter les logs de Traefik en temps réel
docker logs --tail 20 coolify-proxy
# Confirme : Traefik journalise ClientAddr: 10.0.1.1 et applique la règle ipAllowList sur 10.0.1.1.
```

---

## Actions de Contournement Appliquées

1. **Grafana (`monitoring.ims-world.fr`)** : Le middleware `vpn-only` a été retiré de sa route et la sécurité a été confiée au **SSO OIDC Authentik** (`auth.ims-world.fr`).
2. **Stack Media (`qbit`, `radarr`, `sonarr`, `prowlarr`)** : Le middleware y est conservé, mais les applications s'appuient sur leurs authentifications natives (mots de passe / clés API).

---

## Plan de Fix Définitif (Roadmap)

Le fix système global consiste à inscrire `"userland-proxy": false` dans `/etc/docker/daemon.json` sur la VM Coolify. Cette modification a été inscrite à la [Roadmap](/procedures/roadmap#7--fenêtre-de-maintenance-userland-proxy-false--restauration-ip-source-adr-009) pour une fenêtre de maintenance dédiée afin de tester l'absence d'effets de bord sur les ~20 bridges Docker d'infrastructure.

Voir le document de référence d'architecture : **[ADR-009 — Bug userland-proxy](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only)**.
