---
title: "Intégrer un Nouveau Service avec Authentik OIDC SSO"
description: "Guide opérationnel pas-à-pas pour connecter une nouvelle application à l'authentification unique Authentik OIDC (Méthode de capture de la Redirect URI, Providers, Applications)"
icon: "key"
iconType: "duotone"
---

<Badge color="green">🟢 Procédure Standard Validée</Badge>

## Objectif

Ce guide décrit la séquence exacte pour intégrer une **nouvelle application** du homelab au fournisseur d'identité centralisé **Authentik** via le protocole **OpenID Connect (OIDC / OAuth2)** (`https://auth.ims-world.fr`).

---

## 🎯 Méthode Infaillible : Capturer la Redirect URI (Callback URL) en 10 Secondes

<Info>
Lors de l'installation d'un nouveau service, déterminer la bonne **Redirect URI** à inscrire dans le Provider Authentik est la seule réelle difficulté. **Ne cherchez pas dans la doc de l'application : laissez Authentik vous donner l'URL exacte !**
</Info>

### ⚡ La Procédure de Capture d'URL en 4 Étapes

<Steps>
  <Step title="1. Saisir une Regex temporaire dans Authentik">
    Lors de la création du Provider dans Authentik Admin (`https://auth.ims-world.fr/if/admin/#/core/providers`), mettez cette Regex temporaire dans le champ **Redirect URIs** :
    ```text
    https://<nouveau-service>.ims-world.fr/.*
    ```
  </Step>

  <Step title="2. Configurer les variables OIDC sur la nouvelle App">
    Renseignez le `CLIENT_ID`, `CLIENT_SECRET` et l'Issuer URL (`https://auth.ims-world.fr/application/o/<slug>/`) sur votre nouveau conteneur.
  </Step>

  <Step title="3. Lancer la connexion et lire l'URL du navigateur">
    Ouvrez une fenêtre privée, accédez à votre nouvelle application et cliquez sur **Se connecter avec Authentik**.
    
    Regardez la barre d'adresse de votre navigateur sur la page de connexion Authentik : l'application envoie son adresse de retour exacte dans le paramètre GET `redirect_uri=...` !
  </Step>

  <Step title="4. Décoder et coller l'URL dans le Provider">
    Copiez la valeur du paramètre `redirect_uri` (décodée) et remplacez la Regex temporaire dans le Provider Authentik.
  </Step>
</Steps>

<Tip>
**Exemple Vécu (CrowdSec Dashboard / Shield)** :
L'URL affichée par le navigateur dans la barre d'adresse :
`https://auth.ims-world.fr/application/o/authorize/?client_id=...&redirect_uri=https%3A%2F%2Fshield.ims-world.fr%2Fapi%2Fauth%2Foidc%2Fcallback&response_type=code`

Le paramètre `redirect_uri` décodé donne immédiatement la bonne valeur :
`https://shield.ims-world.fr/api/auth/oidc/callback` ➔ **À coller directement dans le Provider Authentik !**
</Tip>

---

## 📚 Tableau des Motifs Standard par Stack Technique

Si vous préférez tenter la saisie directe selon la technologie de la nouvelle application :

| Framework / Stack de la nouvelle App | Format de Redirect URI / Callback à tester |
|---|---|
| **Next.js / Node.js (NextAuth / Remix)** | `https://<service>.ims-world.fr/api/auth/callback/authentik`<br/>`https://<service>.ims-world.fr/api/auth/oidc/callback` |
| **Go / Python / FastAPI / Flask** | `https://<service>.ims-world.fr/oauth2/callback`<br/>`https://<service>.ims-world.fr/auth/login` |
| **Java / Spring Boot** | `https://<service>.ims-world.fr/login/oauth2/code/authentik` |
| **PHP / Laravel / Symfony** | `https://<service>.ims-world.fr/auth/authentik/callback` |
| **Generic OAuth2 / Grafana / Caddy** | `https://<service>.ims-world.fr/login/generic_oauth` |
| **Apps Mobiles (Immich, Ente, etc.)** | `app.<nom-app>:///oauth-callback` *(ajouter cette 2ᵉ URI pour l'app smartphone)* |

---

## 🌐 Endpoints Universels Authentik OIDC (`auth.ims-world.fr`)

Lorsque la nouvelle application vous demande ses paramètres d'authentification OIDC, utilisez ces URLs canoniques :

| Paramètre OIDC réclamé par la nouvelle App | URL / Valeur à Renseigner |
|---|---|
| **OpenID Configuration (Well-Known / Auto-discovery)** | `https://auth.ims-world.fr/application/o/<slug>/.well-known/openid-configuration` |
| **Issuer URL** | `https://auth.ims-world.fr/application/o/<slug>/` |
| **Authorization Endpoint** | `https://auth.ims-world.fr/application/o/authorize/` |
| **Token Endpoint** | `https://auth.ims-world.fr/application/o/token/` |
| **Userinfo Endpoint** | `https://auth.ims-world.fr/application/o/userinfo/` |
| **JSON Web Key Set (JWKS)** | `https://auth.ims-world.fr/application/o/<slug>/jwks/` |

---

## Procédure Pas-à-Pas de Déploiement (5 Étapes)

<Steps>
  <Step title="Étape 1 — Créer le Provider OAuth2/OpenID dans Authentik">
    1. Se connecter à la console d'administration Authentik (`https://auth.ims-world.fr/if/admin/#/core/providers`).
    2. Cliquer sur **Applications ➔ Providers ➔ Create**.
    3. Choisir le type **OAuth2/OpenID Provider** et valider.
    4. Renseigner les champs :
       - **Name** : `<Nouveau-Service> Provider`
       - **Authorization flow** : `default-authorization-flow` (gère la connexion + 2FA TOTP).
       - **Client type** : `Confidential` (pour la majorité des apps serveur).
       - **Redirect URIs / Callbacks** : Saisir le motif déterminé via la méthode de capture ci-dessus.
       - **Signing Key** : Sélectionner `authentik Self-signed Certificate`.
    5. Enregistrer le Provider.
  </Step>

  <Step title="Étape 2 — Créer l'Application dans Authentik">
    1. Se rendre dans **Applications ➔ Applications ➔ Create**.
    2. Renseigner les champs :
       - **Name** : Nom affiché sur le portail (ex. `Nouveau Service`)
       - **Slug** : Identifiant unique de l'URL (ex. `nouveau-service`)
       - **Provider** : Sélectionner le Provider créé à l'Étape 1.
       - **Launch URL** : `https://<nouveau-service>.ims-world.fr`
    3. Enregistrer l'application.
  </Step>

  <Step title="Étape 3 — Récupérer le Client ID et Client Secret">
    1. Retourner dans **Applications ➔ Providers** et cliquer sur le Provider créé.
    2. Copier les deux jetons secrets :
       - **Client ID** (ex. `8f9a2b...`)
       - **Client Secret** (ex. `wX9zK2...`)
  </Step>

  <Step title="Étape 4 — Configurer la nouvelle Application (Env / UI)">
    Injecter les identifiants dans la configuration de la nouvelle application conteneurisée.

    ```env
    # Variables d'environnement OIDC usuelles
    OIDC_ENABLED=true
    OIDC_CLIENT_ID="<client-id-authentik>"
    OIDC_CLIENT_SECRET="<client-secret-authentik>"
    OIDC_ISSUER_URL="https://auth.ims-world.fr/application/o/<slug>/"
    OIDC_DISCOVERY_URL="https://auth.ims-world.fr/application/o/<slug>/.well-known/openid-configuration"
    OIDC_SCOPE="openid profile email"
    ```
  </Step>

  <Step title="Étape 5 — Tester et Valider la Connexion SSO">
    1. Ouvrir une **fenêtre de navigation privée**.
    2. Naviguer sur `https://<nouveau-service>.ims-world.fr`.
    3. Cliquer sur **Se connecter avec Authentik**.
    4. Saisir les identifiants utilisateur et valider le deuxième facteur **2FA TOTP**.
    5. Vérifier que la redirection retourne bien sur l'application avec le profil utilisateur connecté.
  </Step>
</Steps>

---

<CardGroup cols={2}>
  <Card title="Fiche Service Authentik" icon="shield-check" href="/services/authentik">
    Architecture globale d'Authentik, Outposts et gestion des utilisateurs.
  </Card>
  <Card title="Matrice de Sécurité & Exposition" icon="lock" href="/reseau/matrice-securite-exposition">
    Cartographie des modes d'authentification par service.
  </Card>
</CardGroup>
