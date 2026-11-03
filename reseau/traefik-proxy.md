---
title: "Traefik"
description: "Reverse proxy unique du homelab, géré par Coolify : certificats Let's Encrypt par DNS-01, filtrage CrowdSec, routeurs restreints au tailnet et pages d'erreur."
icon: "traffic-light"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v3.7.13"
---

import { ips } from "/snippets/variables.mdx";

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | Tous les sous-domaines de `ims-world.fr` |
| Accès | Ports 80 et 443 redirigés par la Bbox ; tailnet |
| Hôte | VM 104, conteneur `coolify-proxy`, réseau Docker `coolify` |
| Coolify | Proxy du serveur (configuration dans `/data/coolify/proxy/`) |
| Image | `traefik:v3.7.13` |
| Données | `acme.json` (certificats), fichiers dynamiques `/data/coolify/proxy/dynamic/` |
| Sauvegarde | Avec la VM 104 |
| Supervision | Métriques Traefik dans Grafana (tableau « Traefik — Reverse Proxy ») |
| Mise en service | Août 2026 |

## Fonctionnement

Le schéma suit une requête, du point d'entrée au service.

```mermaid
flowchart LR
  client[Client] --> ep[Points d'entrée<br/>http, https]
  ep --> cs[Bouncer CrowdSec]
  cs --> routeurs{Routeur}
  routeurs -->|labels Coolify| public[Service public<br/>SSO ou forward-auth]
  routeurs -->|vpn-only.yaml| prive[Service d'administration<br/>tailnet ou LAN]
  routeurs -->|aucun| erreurs[Pages d'erreur]
  class client externe
  class ep,cs,routeurs reseau
  class public,prive,erreurs service
  classDef service fill:#e8f3eb,stroke:#2e6b3f,color:#1b2129
  classDef reseau fill:#fdf2e3,stroke:#a35a00,color:#1b2129
  classDef externe fill:#f1f2f4,stroke:#6b7480,color:#1b2129,stroke-dasharray:4 3
```

Chaque requête passe par le bouncer CrowdSec, appliqué à tous les points d'entrée, puis par un routeur :

| Type de service | Routeur | Exemple |
| --- | --- | --- |
| Public, avec SSO ou forward-auth | Généré par Coolify depuis le champ **Domains** | Authentik, Vaultwarden, Immich |
| Restreint au tailnet | Déclaré dans `vpn-only.yaml`, champ **Domains** vide dans Coolify | Grafana, qBittorrent, Coolify, Headplane |
| Inconnu | `catchall-error-pages` (priorité 1) | Sous-domaine inexistant |

L'exposition de chaque service est recensée dans la [matrice d'exposition](/reseau/matrice-securite-exposition).

## Certificats

Let's Encrypt par défi DNS-01, via l'API OVH. Traefik crée l'enregistrement `_acme-challenge`, attend sa propagation, puis stocke le certificat dans `acme.json`.

```yaml
command:
  - '--certificatesresolvers.letsencrypt.acme.dnschallenge=true'
  - '--certificatesresolvers.letsencrypt.acme.dnschallenge.provider=ovh'
  - '--certificatesresolvers.letsencrypt.acme.dnschallenge.resolvers=8.8.8.8:53'
  - '--certificatesresolvers.letsencrypt.acme.storage=/traefik/acme.json'
environment:
  - OVH_ENDPOINT_FILE=/run/secrets/ovh_endpoint
  - OVH_APPLICATION_KEY_FILE=/run/secrets/ovh_application_key
  - OVH_APPLICATION_SECRET_FILE=/run/secrets/ovh_application_secret
  - OVH_CONSUMER_KEY_FILE=/run/secrets/ovh_consumer_key
volumes:
  - /data/coolify/proxy/secrets:/run/secrets:ro
```

Les identifiants de l'API OVH ne figurent pas dans la configuration du proxy : chaque variable `_FILE` pointe vers un fichier de `/data/coolify/proxy/secrets/` sur la VM 104 (dossier `700`, fichiers `600`, propriétaire root, sans ACL). Ils n'apparaissent ni dans Coolify ni dans `docker inspect`. La clé OVH est limitée aux enregistrements de la zone : `GET`, `POST` et `DELETE` sur `/domain/zone/ims-world.fr/*`.

<AccordionGroup>
  <Accordion title="Renouveler la clé OVH" icon="key">
    1. Créer une clé sur `https://eu.api.ovh.com/createToken/` avec les trois droits ci-dessus.
    2. Remplacer la valeur dans chaque fichier, un par un : `sudo nano /data/coolify/proxy/secrets/ovh_<nom>`.
    3. Dans Coolify, **Servers → localhost → Proxy → Restart Proxy**, puis vérifier l'absence d'erreur : `docker logs coolify-proxy 2>&1 | grep -i ovh`.
    4. Supprimer l'ancienne clé dans la console de l'API OVH (`DELETE /me/api/application/{id}`).

    Le dossier n'est lisible que par root : un `*` tapé sans `sudo sh -c '…'` n'est pas développé.
  </Accordion>
</AccordionGroup>

Le résolveur conseillé par OVH (`213.251.128.1`) était injoignable lors de la mise en place : seul `8.8.8.8` est utilisé.

## Bouncer CrowdSec

Plugin `crowdsec-bouncer-traefik-plugin` v1.6.0, appliqué aux deux points d'entrée :

```text
--entrypoints.http.http.middlewares=crowdsec-bouncer@file,error-404@file,error-5xx@file
--entrypoints.https.http.middlewares=crowdsec-bouncer@file,error-404@file,error-5xx@file
```

- Mode `stream` : les décisions sont synchronisées toutes les 60 secondes, sans appel par requête.
- `updateMaxFailure: -1` : si CrowdSec s'arrête, le trafic passe (fail-open), validé par un test réel.

Détail dans [CrowdSec](/reseau/crowdsec).

## Services restreints au tailnet

Les routeurs des services d'administration sont centralisés dans `/data/coolify/proxy/dynamic/vpn-only.yaml`, relu à chaud par Traefik ([ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only)) :

```yaml
http:
  middlewares:
    vpn-only:
      ipAllowList:
        sourceRange:
          - "100.64.0.0/10"    # tailnet
          - "192.168.1.0/24"   # LAN
    admin-gzip:
      compress: {}
  routers:
    grafana-admin:
      rule: "Host(`monitoring.ims-world.fr`)"
      entryPoints: [https]
      service: grafana-admin
      middlewares: [vpn-only, admin-gzip]
      tls: {certResolver: letsencrypt}
  services:
    grafana-admin:
      loadBalancer:
        servers:
          - url: "http://grafana-rrw19kmye6gng961igtzqpgw:3000"
```

Routeurs déclarés : Grafana, qBittorrent, Sonarr, Radarr, Prowlarr, Coolify, Headplane, Dozzle, interface CrowdSec, WUD, Termix, Home Assistant. Le fichier fait foi.

Chaque service est désigné par le nom de son conteneur, `<service>-<UUID Coolify>`, seul nom connu de Traefik sur le réseau `coolify`. Ce nom ne change pas au redéploiement ni à la mise à jour, seulement si le service est supprimé puis recréé dans Coolify : le routeur répond alors 502 et la sonde Uptime Kuma le signale. Le fichier unique est conservé volontairement, même au prix de ces noms peu lisibles : il donne la liste complète des accès restreints au même endroit, et permet de vérifier chaque middleware d'un coup d'œil. Alias réseau et labels par conteneur ont été écartés (03/11/2026).

Le filtrage suppose que Traefik voie l'IP réelle du client : voir [VM 104](/infrastructure/vm-coolify#réseau-docker-et-ip-source).

## Pages d'erreur

Un conteneur Nginx (`nginx:1.27-alpine`, UUID Coolify `vciwi7dolcl0hw1mffvjcfha`) sert les pages d'erreur. Configuration dans `/data/coolify/proxy/dynamic/error-pages.yaml` :

```yaml
http:
  middlewares:
    error-404:
      errors:
        status: ["404", "403"]
        service: error-pages-svc
        query: "/404.html"
    error-5xx:
      errors:
        status: ["500-599"]
        service: error-pages-svc
        query: "/5xx.html?code={status}"
  routers:
    catchall-error-pages:
      rule: "PathPrefix(`/`)"
      priority: 1
      entryPoints: [http, https]
      service: error-pages-svc
  services:
    error-pages-svc:
      loadBalancer:
        servers:
          - url: "http://vciwi7dolcl0hw1mffvjcfha-nginx:80"
```

**Refus hors tailnet.** Un client hors tailnet qui demande un service restreint reçoit le code HTTP 403 de `vpn-only`, avec le contenu de la page 404 générique : la page n'indique pas que le service existe. Les tests en ligne de commande lisent donc un code 403 (`curl -I`), le navigateur affiche « Page non trouvée ».

## Exploitation

```bash
# Depuis la VM 104
docker logs --tail 100 -f coolify-proxy
docker restart coolify-proxy
# Après une modification du plugin CrowdSec
docker compose -f /data/coolify/proxy/docker-compose.yml up -d --force-recreate
```

## Pièges connus

Chaque piège donne le symptôme observé, sa cause et la correction.

<AccordionGroup>
  <Accordion title="Une modification de /data/coolify/proxy/docker-compose.yml disparaît">
    **Cause** : Coolify régénère ce fichier depuis sa base.

    **Solution** : modifier la commande dans l'interface Coolify (**Servers → Proxy**).
  </Accordion>
  <Accordion title="Un service restreint redevient public">
    **Cause** : domaine saisi dans le champ **Domains** de Coolify : un second routeur, sans `vpn-only`, est créé.

    **Solution** : vider le champ, garder le routeur dans `vpn-only.yaml`.
  </Accordion>
  <Accordion title="Les labels d'un conteneur (forward-auth) n'ont aucun effet">
    **Cause** : un routeur du fichier `vpn-only.yaml` porte le même nom d'hôte et prend la priorité.

    **Solution** : déclarer le middleware dans `vpn-only.yaml` avec le suffixe `@docker` ([incident du 24/08/2026](/history/incidents/2026-08-24-bypass-sso-dozzle-traefik-file-provider)).
  </Accordion>
  <Accordion title="Tout le monde est refusé par vpn-only">
    **Cause** : Traefik voit l'IP du pont Docker (`10.0.1.1`).

    **Solution** : vérifier `userland-proxy` et le SNAT Tailscale ([incident du 18/08/2026](/history/incidents/2026-08-18-blocage-traefik-vpn-only-docker-proxy)).
  </Accordion>
  <Accordion title="Interface Coolify en 502 après une mise à jour">
    **Cause** : le proxy a perdu la liaison avec Coolify.

    **Solution** : `docker restart coolify-proxy`.
  </Accordion>
  <Accordion title="Tester la page 5xx sans couper un service">
    **Cause** : `docker stop` retire le routeur (on tombe sur le 404).

    **Solution** : `docker pause <conteneur>`, puis `docker unpause`.
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card color="#FF6B00" title="Matrice d'exposition" icon="table-cells" href="/reseau/matrice-securite-exposition">Zone et authentification de chaque service.</Card>
  <Card color="#FF6B00" title="Restreindre un service au tailnet" icon="list-check" href="/procedures/securiser-service-vpn-only">Procédure de restriction au tailnet.</Card>
  <Card color="#FF6B00" title="Forward-auth Authentik" icon="list-check" href="/procedures/securiser-application-authentik-forward-auth">Procédure de protection par Authentik.</Card>
</CardGroup>
