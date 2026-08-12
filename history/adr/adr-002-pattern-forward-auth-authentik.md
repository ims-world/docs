---
title: "ADR-002 — Sécurisation par Forward-Auth Authentik Outpost"
description: "Décision d'architecture pour l'authentification unifiée des applications web sans SSO natif"
icon: "key"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-12)*

---

## Contexte

Plusieurs applications web self-hosted indispensables au homelab (ex: Dozzle pour les logs Docker live, Uptime Kuma pour la statuspage, IT-Tools pour les utilitaires IT) ne possèdent pas de système d'authentification natif ni de support du protocole OIDC (OpenID Connect).

Exposer ces applications directement sur le Public WAN sans protection représentait une vulnérabilité critique.

---

## Décision

Nous avons décidé de généraliser l'utilisation du **Proxy Outpost Authentik (Forward-Auth Traefik)** (`ak-outpost-ims-outpost:9000`) pour toutes les applications hébergées n'ayant pas d'authentification native.

- **Principe** : Traefik intercepte chaque requête HTTPS entrante sur le sous-domaine visé (`logs.ims-world.fr`, `status.ims-world.fr`, `tools.ims-world.fr`).
- **Interrogation en amont** : Traefik transmet les en-têtes à l'outpost Authentik local via le middleware `forwardAuth`.
- **Contrôle d'accès** : Si l'utilisateur possède un cookie de session Authentik valide, la requête est transmise au conteneur cible. Sinon, il est immédiatement redirigé vers le portail SSO `auth.ims-world.fr`.

---

## Conséquences

### Positives
- **Sécurité zéro-confiance** : Aucune application non authentifiée n'est accessible sur le Web.
- **Expérience SSO unifiée** : Une seule connexion sur `auth.ims-world.fr` déverrouille l'ensemble des outils d'administration.
- **Zéro modification de code** : S'applique en simple configuration de labels Traefik dans le `docker-compose.yml` du service.

### Négatives / Contraintes
- Dépendance critique envers le conteneur `ak-outpost-ims-outpost` (s'il s'arrête, l'accès aux services Forward-Auth est bloqué).
- Impossibilité pour les applications mobiles tierces non navigables d'utiliser des APIs si elles ne gèrent pas le cookie de session Forward-Auth.
