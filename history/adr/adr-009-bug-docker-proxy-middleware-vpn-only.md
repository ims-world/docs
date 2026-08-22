---
title: "ADR-009 — Isolation Réseau des Services d'Administration (vpn-only & userland-proxy)"
description: "Décision d'architecture pour le durcissement du réseau Docker (CIS Benchmark) et la centralisation des accès privés via Traefik File Provider"
icon: "shield-check"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Implémenté & Validé</Badge> *(Août 2026)*

---

## Contexte

L'architecture du homelab IMS-WORLD exige la possibilité de restreindre l'accès à un ensemble de services d'administration et d'infrastructures sensibles (`coolify`, `headplane`, `qbit`, `sonarr`, `radarr`, `prowlarr`, `monitoring`) aux seuls clients connectés au réseau privé Overlay Tailscale/Headscale (`100.64.0.0/10`), tout en conservant d'autres services (Immich, PhotoPrism, HomeFlix Jellyfin, Vaultwarden, Authentik) accessibles sur l'Internet public.

Initialement, un middleware Traefik `vpn-only` (`ipAllowList: 100.64.0.0/10`) avait été configuré, mais s'avérait **inopérant** : l'intégralité des requêtes HTTP/HTTPS arrivait au conteneur Traefik avec l'adresse IP de la passerelle Docker bridge (`10.0.1.1`), déclenchant un rejet HTTP 403 y compris pour les utilisateurs légitimes du VPN.

---

## Décision d'Architecture

### 1. Les Deux Piliers Réseau de Préservation de l'IP Source
La préservation de l'adresse IP source réelle du client jusqu'au conteneur Traefik repose sur deux configurations réseau complémentaires et indissociables :

#### Volet A — Durcissement du Démon Docker (`"userland-proxy": false`)
Nous avons désactivé le proxy applicatif userland de Docker dans `/etc/docker/daemon.json`. Conforme au **CIS Docker Benchmark (Section 2.12)**, Docker bascule l'intégralité du routage sur les mécanismes natifs du noyau Linux (iptables `DNAT` + `MASQUERADE` et paramètre `net.ipv4.route_localnet`), éliminant la substitution d'IP par le relais `docker-proxy` (`10.0.1.1`).

```json
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "default-address-pools": [{"base":"10.0.0.0/8","size":24}],
  "userland-proxy": false
}
```

#### Volet B — Désactivation du SNAT Tailscale (`--snat-subnet-routes=false`)
Par défaut, le démon `tailscaled` applique une règle `MASQUERADE` dans la chaîne iptables `ts-postrouting` basée sur le `fwmark 0x40000/0xff0000` dès qu'un paquet entrant sur `tailscale0` est réorienté (DNAT) vers le bridge réseau de Docker (`coolify`). Pour empêcher Tailscale de masquer l'IP du client avec l'IP du bridge lors du traversée de frontière réseau, la configuration suivante est appliquée :

```bash
sudo tailscale set --snat-subnet-routes=false
```

### 2. Routage Centralisé via Traefik File Provider (`vpn-only.yaml`)
Pour éliminer les concurrences de routeurs provoquées par l'orchestrateur Coolify (qui écrase les labels middlewares sur les ressources de type "Service"), l'intégralité des routeurs et des définitions de services `vpn-only` est centralisée dans un unique fichier dynamique du provider File de Traefik : `/data/coolify/proxy/dynamic/vpn-only.yaml`.

- Les sous-domaines d'administration sont **retirés du champ "Domains" de l'UI Coolify** pour éviter tout routeur généré automatiquement en parallèle.
- Le conteneur `coolify-proxy` conserve sa topologie en **mode bridge standard** sans modifier son attachement aux ~20 réseaux Docker isolés.
- Les routeurs Traefik pointent directement vers les conteneurs cibles via le DNS interne Docker (`http://<nom-conteneur>:<port>`).

```yaml
# /data/coolify/proxy/dynamic/vpn-only.yaml
http:
  middlewares:
    vpn-only:
      ipAllowList:
        sourceRange:
          - 100.64.0.0/10
          - 192.168.1.0/24
    admin-gzip:
      compress: {}

  routers:
    grafana-admin:
      rule: "Host(`monitoring.ims-world.fr`)"
      entryPoints: [https]
      service: grafana-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}

    qbit-admin:
      rule: "Host(`qbit.ims-world.fr`)"
      entryPoints: [https]
      service: qbit-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}

    sonarr-admin:
      rule: "Host(`sonarr.ims-world.fr`)"
      entryPoints: [https]
      service: sonarr-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}

    radarr-admin:
      rule: "Host(`radarr.ims-world.fr`)"
      entryPoints: [https]
      service: radarr-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}

    prowlarr-admin:
      rule: "Host(`prowlarr.ims-world.fr`)"
      entryPoints: [https]
      service: prowlarr-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}

    coolify-admin:
      rule: "Host(`coolify.ims-world.fr`)"
      entryPoints: [https]
      service: coolify-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}

    headplane-admin:
      rule: "Host(`admin.vpn.ims-world.fr`)"
      entryPoints: [https]
      service: headplane-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}

  services:
    grafana-admin:
      loadBalancer:
        servers:
          - url: "http://grafana-rrw19kmye6gng961igtzqpgw:3000"

    qbit-admin:
      loadBalancer:
        servers:
          - url: "http://gluetun-w39uebmcnse7yctsft8hzed8:8080"

    sonarr-admin:
      loadBalancer:
        servers:
          - url: "http://sonarr-w39uebmcnse7yctsft8hzed8:8989"

    radarr-admin:
      loadBalancer:
        servers:
          - url: "http://radarr-w39uebmcnse7yctsft8hzed8:7878"

    prowlarr-admin:
      loadBalancer:
        servers:
          - url: "http://prowlarr-w39uebmcnse7yctsft8hzed8:9696"

    coolify-admin:
      loadBalancer:
        servers:
          - url: "http://coolify:8080"

    headplane-admin:
      loadBalancer:
        servers:
          - url: "http://headplane-i136ix2bmrrbeovnyrh1o72w:3000"
```

### 3. Découplage Headscale / Headplane (2 URLs Distinctes)
Pour résoudre l'incompatibilité entre le serveur de coordination VPN et l'interface d'administration web :
- **`vpn.ims-world.fr`** (Public WAN) : Conservé publiquement sans middleware `vpn-only` pour permettre l'initialisation des connexions Tailscale des clients distants avant l'établissement du tunnel.
- **`https://admin.vpn.ims-world.fr/admin`** (Tailnet Only) : Interface d'administration Headplane restreinte au Tailnet.
  - Résolution split-horizon via l'enregistrement Headscale `dns.extra_records` (`admin.vpn.ims-world.fr` → `100.64.0.4`).
  - **Le suffixe de chemin `/admin` est OBLIGATOIRE** car Headplane est compilé avec le base path `/admin` et renvoie une erreur 404 sur la racine `/`.

---

## Alternatives Envisagées & Rejetées

### 1. Mode Réseau Host sur Traefik (`network_mode: host`)
- **Rejetée** : Augmente de manière inacceptable la surface d'attaque de Traefik en cas de compromission (accès direct à tous les services écoutant sur `127.0.0.1` de l'hôte). Exige de retirer Traefik du réseau `coolify`, risquant de rompre la résolution des 20 réseaux isolés.

### 2. Second Traefik Dédié en Sidecar Tailscale
- **Rejetée** : Entraîne le maintien de deux conteneurs supplémentaires, la duplication de la gestion des certificats ACME et du challenge DNS-01, pour une complexité injustifiée alors que la désactivation de `userland-proxy` couvre l'intégralité des besoins.

### 3. ForwardAuth Authentik Généralisé (sans fix réseau)
- **Rejetée** : Tailscale fournissant déjà une authentification forte basée sur l'identité matérielle (clés WireGuard), l'ajout de 5 Outposts/Providers Authentik sur la stack de téléchargement aurait apporté une complexité disproportionnée pour une protection redondante.

### 4. Labels Traefik directement dans les Compose managés par Coolify
- **Rejetée** : Coolify écrase silencieusement la clé `middlewares` des routeurs générés automatiquement pour les ressources de type "Service". L'utilisation de priorités explicites sur des routeurs concurrents produit un comportement non déterministe sous Traefik.

---

## Conséquences & Bénéfices

- **Sécurité en Profondeur (CIS Benchmark)** : Préservation absolue de l'IP source réelle sans dépendre d'un proxy userland fragile.
- **Zéro Concurrence de Routeurs** : La suppression du domaine dans l'UI Coolify garantit que Traefik applique exclusivement les règles centralisées du fichier `vpn-only.yaml`.
- **Hot-Reload Instantané** : Toute modification de `vpn-only.yaml` est prise en compte instantanément par Traefik sans aucun redémarrage du proxy.
- **Ressources Système Optimisées** : Suppression des processus `docker-proxy` en arrière-plan.
