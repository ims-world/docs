---
title: "Authentik"
description: "Fournisseur d'identité du homelab : connexion unique (OIDC) pour les applications compatibles, forward-auth pour les autres, invitations et groupes."
icon: "key"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "2026.8.0"
---

import { domains } from "/snippets/variables.mdx";

## Accès

<CardGroup cols={2}>
  <Card title="Authentik" icon="shield-halved" href="https://auth.ims-world.fr">
    Portail de connexion et console d'administration.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104 : lien de récupération pour akadmin, valable 1 heure
docker exec -it authentik-worker-k5mxvc2r6c4zlb6j3d443h7b ak create_recovery_key 1 akadmin
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | {domains.sso} |
| Accès | Public ; double facteur (TOTP, WebAuthn) |
| Hôte | VM 104 |
| Coolify | UUID `k5mxvc2r6c4zlb6j3d443h7b`, chemin `/data/coolify/services/k5mxvc2r6c4zlb6j3d443h7b/` |
| Image | `ghcr.io/goauthentik/server:2026.8.0` ; outpost `ghcr.io/goauthentik/proxy:2026.8.0` |
| Données | PostgreSQL (volume Docker sur la VM 104) ; clé `SECRET_KEY` |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | Août 2026 |

## Fonctionnement

Deux modes d'intégration, selon l'application :

| Mode | Applications | Procédure |
| --- | --- | --- |
| OIDC : l'application dialogue avec Authentik | Grafana, Headplane, Headscale, Immich, Zipline, Forgejo, PhotoPrism, Termix, interface CrowdSec | [Connecter une application en OIDC](/procedures/integration-service-authentik-oidc) |
| Forward-auth : Traefik demande à l'outpost avant de transmettre | Dozzle, Uptime Kuma, IT-Tools, Stirling PDF | [Forward-auth](/procedures/securiser-application-authentik-forward-auth) ([ADR-002](/history/adr/adr-002-pattern-forward-auth-authentik)) |

L'outpost `ak-outpost-ims-outpost` (port 9000) est créé et géré par Authentik lui-même, par le socket Docker : il n'apparaît pas dans le compose de Coolify. S'il s'arrête, tous les services en forward-auth deviennent inaccessibles.

Les droits par groupe (`authentik Admins`, `admins`, `membres`, `invites`, `authentik Read-only`) sont dans la [matrice d'exposition](/reseau/matrice-securite-exposition#droits-par-groupe-authentik).

## Données et sauvegarde

La base PostgreSQL et la configuration sont sauvegardées avec la VM 104. La clé `SECRET_KEY` doit être restaurée avec la base : sans elle, sessions et jetons sont invalidés. Export ponctuel : voir la [politique de sauvegarde](/procedures/politique-sauvegardes).

## Exploitation

### Inviter un utilisateur

<Steps>
  <Step title="Créer l'invitation">
    **Répertoire → Invitations → New Invitation**, avec le flux d'inscription existant. Nom en minuscules (`jean-dupont`), **Usage unique** coché.
  </Step>
  <Step title="Transmettre le lien">
    Copier le lien généré, ou l'envoyer depuis l'interface.
  </Step>
  <Step title="Inscription">
    L'invité choisit son identifiant et son mot de passe, puis confirme son adresse (courriel envoyé par Resend, `no-reply@ims-world.fr`). Le compte rejoint le groupe `membres`.
  </Step>
</Steps>

### Mettre à jour

<Steps>
  <Step title="Changer le tag">
    Modifier le tag de `authentik-server` et `authentik-worker` dans Coolify, puis redéployer.
  </Step>
  <Step title="Aligner l'outpost">
    Le conteneur outpost n'est pas redéployé avec le reste. Si **Applications → Outposts** signale une version différente :

    ```bash
    docker stop ak-outpost-ims-outpost && docker rm ak-outpost-ims-outpost
    docker restart authentik-server-k5mxvc2r6c4zlb6j3d443h7b authentik-worker-k5mxvc2r6c4zlb6j3d443h7b
    ```
    Authentik recrée l'outpost à la bonne version. Le conteneur ne contient aucune donnée.
  </Step>
</Steps>

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| Outpost resté à l'ancienne version après une mise à jour | Il n'est pas géré par le compose de Coolify | Supprimer le conteneur et redémarrer serveur et worker |
| Labels forward-auth sans effet sur un service restreint au tailnet | Le routeur de `vpn-only.yaml` prend la priorité | Déclarer le middleware avec `@docker` dans `vpn-only.yaml` |
| Parseur CrowdSec Authentik en échec | Il attend du syslog | Voir [CrowdSec](/reseau/crowdsec) |

## Voir aussi

- [Matrice d'exposition](/reseau/matrice-securite-exposition)
- [Headscale](/reseau/headscale-headplane) (enrôlement par OIDC)
