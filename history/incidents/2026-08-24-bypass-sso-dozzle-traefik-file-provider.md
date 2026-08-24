---
title: "Incident — Bypass de l'Authentification SSO Dozzle via vpn-only.yaml"
description: "Contournement silencieux du middleware Forward-Auth Authentik causé par la priorité du routeur Traefik Dynamic File Provider sur le provider Docker"
icon: "shield-exclamation"
iconType: "duotone"
---

<Badge color="green">🟢 Résolu — Middleware `authentik-dozzle@docker` Ajouté dans vpn-only.yaml</Badge> *(2026-08-24)*

*Découvert et corrigé le 24/08/2026.*

---

## Symptôme

`https://logs.ims-world.fr` (Dozzle) était accessible **sans passer par Authentik**, malgré des labels Traefik apparemment corrects définis sur le conteneur Docker :

```yaml
labels:
  - 'traefik.http.middlewares.authentik-dozzle.forwardauth.address=http://ak-outpost-ims-outpost:9000/outpost.goauthentik.io/auth/traefik'
  - traefik.http.middlewares.authentik-dozzle.forwardauth.trustForwardHeader=true
  - 'traefik.http.middlewares.authentik-dozzle.forwardauth.authResponseHeaders=X-authentik-username,X-authentik-groups,X-authentik-email,X-authentik-uid'
  - 'traefik.http.routers.https-0-e4gwwow0ogo8os0gwsksgo4c-dozzle.middlewares=gzip,authentik-dozzle@docker'
```

---

## Cause Racine Identifiée

Dozzle est référencé dans `/data/coolify/proxy/dynamic/vpn-only.yaml` (routeur `dozzle-admin`, provider **file**), au même titre que Grafana, Coolify, Headplane, Sonarr/Radarr/Prowlarr, qBittorrent et `crowdsec-web-ui`. Ce routeur définit sa **propre** liste de middlewares pour `logs.ims-world.fr` :

```yaml
dozzle-admin:
  rule: Host(`logs.ims-world.fr`)
  entryPoints:
    - https
  service: dozzle-admin
  middlewares:
    - vpn-only
    - admin-gzip
  tls:
    certResolver: letsencrypt
```

**Comportement Traefik** : Un routeur du provider **file** portant le même `Host(...)` prend systématiquement la priorité sur le routeur généré automatiquement par les labels Docker (provider **docker**). 

Résultat : les labels `authentik-dozzle` sur le conteneur n'avaient aucun effet réel — l'accès était filtré par IP/VPN (`vpn-only`), mais **jamais par SSO Authentik**. Ce défaut était pré-existant, sans lien avec une migration récente, et a été découvert lors d'un audit de routine.

---

## Fix Appliqué

Ajout explicite du middleware Authentik au routeur `dozzle-admin` dans `/data/coolify/proxy/dynamic/vpn-only.yaml`, avec le suffixe obligatoire **`@docker`** (nécessaire car le middleware est déclaré côté provider Docker mais consommé depuis un routeur du provider File) :

```yaml
dozzle-admin:
  rule: Host(`logs.ims-world.fr`)
  entryPoints:
    - https
  service: dozzle-admin
  middlewares:
    - vpn-only
    - admin-gzip
    - authentik-dozzle@docker
  tls:
    certResolver: letsencrypt
```

Un simple hot-reload du provider File a été suffisant (aucun redémarrage `--force-recreate` de Traefik requis).

**Validation post-fix** : `https://logs.ims-world.fr` exige désormais l'authentification SSO Authentik avant de donner accès à l'interface de logs Dozzle.

---

## ⚠️ Point de Vigilance & Règle d'Or Traefik

<Warning>
**Règle d'Or Traefik File Provider vs Docker Provider** : Tout service dont le routeur est défini dans `vpn-only.yaml` et qui nécessite à la fois l'isolation VPN et l'authentification SSO doit **explicitement déclarer le middleware SSO avec la syntaxe `nom-middleware@docker`** dans `vpn-only.yaml`. Les labels Docker seuls seront ignorés au profit du routeur File Provider.
</Warning>

### Priority Check :
Vérifier en priorité l'ensemble des routeurs déclarés dans `/data/coolify/proxy/dynamic/vpn-only.yaml` (`crowdsec-web-ui-admin`, `grafana-admin`, `coolify-admin`, `headplane-admin`, etc.) pour s'assurer que leur combinaison de middlewares est complète.

```bash
# Commande utile pour détecter si un domaine est surchargé par le provider File :
grep -rl "logs.ims-world.fr" /data/coolify/proxy/dynamic/
```
