---
title: "Authentik"
description: "SSO / OIDC — provider d'identité central"
icon: "key"
iconType: "duotone"
---

<Card title="Authentik SSO" icon="key" href="https://auth.ims-world.fr">
  Provider d'identité centralisé (OAuth2 / OIDC + WebAuthn 2FA). Accessible sur `auth.ims-world.fr`.
</Card>

## Fiche service

| Propriété | Valeur |
|---|---|
| **Domaine** | `auth.ims-world.fr` |
| **Rôle** | SSO / OIDC pour l'ensemble des services (Headscale, Headplane, Jellyfin prévu) |
| **Base de données** | PostgreSQL |
| **UUID Coolify** | `k5mxvc2r6c4zlb6j3d443h7b` |
| **Chemin** | `/data/coolify/services/k5mxvc2r6c4zlb6j3d443h7b/` |
| **Statut** | 🟢 Production |

## Séquence d'Authentification OIDC Centralisée

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 Utilisateur
    participant Client as 🚀 Service Applicatif (Headscale/Headplane/Jellyfin)
    participant Auth as 🔐 Authentik (auth.ims-world.fr)
    participant DB as 🐘 Postgres DB (authentik)

    User->>Client: Accès au service
    Client-->>User: Redirection OAuth2 / OIDC Authorization Code
    User->>Auth: Prompt Connexion SSO + WebAuthn 2FA
    Auth->>DB: Vérification identifiants & session
    DB-->>Auth: Valide (User Authenticated)
    Auth-->>User: Redirection vers le Service avec Authorization Code
    User->>Client: Transmet le Code
    Client->>Auth: Exchange Code against ID Token + Access Token
    Auth-->>Client: Token OIDC JWT signé
    Client-->>User: Accès accordé & Session établie
```

<Check>
Service d'identité centralisé actif. La clé `SECRET_KEY` doit rester strictement identique entre redéploiements pour préserver les sessions utilisateurs.
</Check>

## Sauvegarde & Restauration PostgreSQL

<Steps>
  <Step title="Sauvegarde de la base de données">
    ```bash
    docker exec postgresql-k5mxvc2r6c4zlb6j3d443h7b pg_dump -U <user> -d authentik -F c -f /tmp/dump.sql
    ```
  </Step>
  <Step title="Restauration de la base">
    ```bash
    # Arrêter les conteneurs applicatifs (authentik-server / worker)
    pg_restore -U <user> -d authentik --clean --if-exists /tmp/dump.sql
    ```
  </Step>
</Steps>

## Découvertes non prévues

<Warning>
**Branding par domaine** : le CSS custom et le logo n'apparaissaient pas sur la nouvelle instance malgré un restore DB réussi. Cause : la config de marque (Customization → Brands) est rattachée à un domaine précis dans Authentik, pas globale.
</Warning>

<Warning>
**Emplacement réel des médias** : le dossier `/media` du container est vide (volume interne éphémère). Les vrais fichiers (icônes, logos) vivent dans `/data/media/public/...`, à l'intérieur du bind-mount `/data`. Localisé via :
```bash
docker exec <container> find / -iname "<nom_fichier_connu>" 2>/dev/null
```
</Warning>

## WebAuthn (2FA)

<Info>
Les credentials WebAuthn sont cryptographiquement liés au nom de domaine exact — non testables sur un sous-domaine `-ng`, uniquement validables une fois le vrai domaine actif. Reste à valider explicitement post-cutover.
</Info>

Contournement utilisé pendant les tests (bypass total mot de passe + 2FA, mécanisme natif) :
```bash
docker exec -it authentik-worker-<uuid> ak create_recovery_key 1 <username>
```

## OIDC — providers configurés

| Client | Issuer |
|---|---|
| Headscale | `https://auth.ims-world.fr/application/o/headscale/` |
| Headplane | `https://auth.ims-world.fr/application/o/headplane/` |
| Jellyfin (prévu) | `https://auth.ims-world.fr/application/o/jellyfin/` |

<Warning>
Au moment du cutover, Headscale a temporairement dû désactiver `only_start_if_oidc_is_available` (mis à `false`) car le port-forward routait vers un Authentik pas encore accessible avec un certificat valide — dépendance circulaire. À repasser en `true` maintenant que la situation est stable. Voir [Headscale](/services/headscale-headplane).
</Warning>
