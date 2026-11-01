---
title: "Connecter une application à Authentik en OIDC"
description: "Créer le fournisseur et l'application dans Authentik, trouver l'URL de redirection attendue et configurer l'application."
icon: "key"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : 20 minutes · Risque : faible · Interruption de service : aucune

## Quand l'utiliser

Pour une application qui gère OpenID Connect (Immich, Grafana, Forgejo, Zipline, PhotoPrism, l'interface CrowdSec…). Si elle ne le gère pas, voir [forward-auth](/procedures/securiser-application-authentik-forward-auth).

## Prérequis

- Compte administrateur Authentik.
- Application déployée et joignable sur son domaine.

## Étapes

<Steps>
  <Step title="Créer le fournisseur OAuth2/OpenID">
    Dans Authentik : **Applications → Providers → Create**, type **OAuth2/OpenID Provider** :
    - **Name** : `<application> Provider` ;
    - **Authorization flow** : `default-authorization-flow` ;
    - **Client type** : `Confidential` ;
    - **Redirect URIs** : provisoirement `https://<sous-domaine>.ims-world.fr/.*` (expression régulière) ;
    - **Signing Key** : `authentik Self-signed Certificate`.
  </Step>
  <Step title="Créer l'application">
    **Applications → Applications → Create** : nom, slug (utilisé dans les URL), fournisseur, **Launch URL** `https://<sous-domaine>.ims-world.fr`. Lier les groupes autorisés.
  </Step>
  <Step title="Configurer l'application">
    Reporter l'identifiant et le secret du client, visibles dans le fournisseur. Variables usuelles :

    ```env
    OIDC_CLIENT_ID=<identifiant>
    OIDC_CLIENT_SECRET=<secret>
    OIDC_ISSUER_URL=https://auth.ims-world.fr/application/o/<slug>/
    OIDC_SCOPE=openid profile email
    ```

    Dans Coolify, déclarer le secret comme variable d'environnement avec l'option **Is Secret**.
  </Step>
  <Step title="Relever l'URL de redirection réelle">
    En navigation privée, cliquer sur la connexion par Authentik dans l'application. L'adresse de la page Authentik contient le paramètre `redirect_uri` : c'est l'URL exacte attendue par l'application. Exemple pour l'interface CrowdSec :

    ```text
    …/authorize/?client_id=…&redirect_uri=https%3A%2F%2Fshield.ims-world.fr%2Fapi%2Fauth%2Foidc%2Fcallback
    → https://shield.ims-world.fr/api/auth/oidc/callback
    ```
  </Step>
  <Step title="Remplacer l'expression provisoire">
    Dans le fournisseur, remplacer `https://<sous-domaine>.ims-world.fr/.*` par l'URL décodée. Une application mobile peut demander une seconde URL (par exemple `app.immich:///oauth-callback`).
  </Step>
</Steps>

## Références

| Paramètre | Valeur |
| --- | --- |
| Découverte | `https://auth.ims-world.fr/application/o/<slug>/.well-known/openid-configuration` |
| Émetteur (issuer) | `https://auth.ims-world.fr/application/o/<slug>/` |
| Autorisation | `https://auth.ims-world.fr/application/o/authorize/` |
| Jeton | `https://auth.ims-world.fr/application/o/token/` |
| Informations utilisateur | `https://auth.ims-world.fr/application/o/userinfo/` |
| Clés (JWKS) | `https://auth.ims-world.fr/application/o/<slug>/jwks/` |

| Type d'application | URL de redirection habituelle |
| --- | --- |
| Next.js, NextAuth | `/api/auth/callback/authentik` ou `/api/auth/oidc/callback` |
| Grafana, OAuth générique | `/login/generic_oauth` |
| Spring Boot | `/login/oauth2/code/authentik` |
| Laravel, Symfony | `/auth/authentik/callback` |

## Vérification

En navigation privée : connexion par Authentik, second facteur, retour sur l'application avec le bon profil. Un compte hors des groupes liés doit être refusé.

## Retour arrière

Désactiver OIDC dans l'application, puis supprimer l'application et le fournisseur dans Authentik.

## Dépannage

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `redirect_uri mismatch` | URL de redirection différente de celle déclarée | Relever `redirect_uri` comme à l'étape 4 |
| Connexion acceptée mais profil vide | Portée `email` ou `profile` absente | Ajouter les portées dans l'application et le fournisseur |
