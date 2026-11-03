---
title: "WUD (veille des images)"
description: "What's Up Docker : détecte les nouvelles versions des images choisies sur la VM 104 et prévient sur Ntfy, sans rien modifier."
icon: "docker"
iconType: "duotone"
last_reviewed: "2026-11-01"
app_version: "v9.0.2"
---

## Accès

Restreint au tailnet, avec connexion Authentik.

<CardGroup cols={2}>
  <Card color="#16A34A" title="WUD" icon="docker" href="https://wud.ims-world.fr">
    Versions en service et versions disponibles.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
docker logs -f --tail=100 wud-qwe5jrqlqtwqneevgkf6mwr9
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `wud.ims-world.fr` |
| Accès | Tailnet ; OIDC Authentik, compte natif `cmolotkoff` en secours ; voir la [matrice](/reseau/matrice-securite-exposition) |
| Hôte | VM 104 |
| Coolify | UUID `qwe5jrqlqtwqneevgkf6mwr9`, chemin `/data/coolify/services/qwe5jrqlqtwqneevgkf6mwr9/` |
| Image | `getwud/wud:9.0.2` |
| Données | `store/` (état SQLite) ; socket Docker en lecture seule |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 14/09/2026 |

## Fonctionnement

WUD liste les tags publiés sur les registres (Docker Hub, GHCR, Quay, Codeberg) et les compare au tag de chaque conteneur surveillé. Il n'applique aucune mise à jour.

- Analyse une fois par jour à 08:00 (`0 8 * * *`), pour rester sous le quota anonyme de Docker Hub.
- Notifications groupées en un seul message par analyse (`WUD_TRIGGER_NTFY_IMS_MODE=batch`) sur le topic Ntfy `ims-alerts`, avec un jeton dédié (`WUD_TRIGGER_NTFY_IMS_AUTH_TOKEN`).
- Une image en `latest` n'a pas de version à comparer. Les services concernés sont listés dans la [feuille de route](/roadmap).

### Surveillance sur liste (opt-in)

Seuls les conteneurs portant le label `wud.watch=true` sont surveillés (`WUD_WATCHER_LOCAL_WATCHBYDEFAULT=false`). Les conteneurs arrêtés sont ignorés (`WUD_WATCHER_LOCAL_WATCHALL=false`).

Ce choix évite deux sources de bruit :

- les conteneurs gérés par un autre outil, dont les labels ne peuvent pas être modifiés durablement : conteneurs de Coolify (régénérés à chaque mise à jour de Coolify) et outposts Authentik (créés par Authentik) ;
- les comparaisons de tags sans rapport (`7-alpine` comparé à `32bit-stretch`, `v3.7.16` à une pré-version Windows), que WUD produit dès qu'un tag n'est pas une version sémantique stricte.

Règle : un label sur le conteneur applicatif principal de chaque stack, aucun sur ses dépendances.

| Conteneur | Surveillé | Raison |
| --- | --- | --- |
| Conteneur principal d'un service (`server` d'Authentik, par exemple) | Oui | Porte la version du service |
| Worker, outpost | Non | Même image et même tag que le conteneur principal |
| Base de données, cache (PostgreSQL, Redis) | Non | Version imposée par l'application ; une montée majeure demande un export et une restauration |
| `coolify`, `coolify-db`, `coolify-redis`, `coolify-sentinel` | Non | Mis à jour par Coolify |

### Labels

À placer dans le compose du service, sous le conteneur principal :

```yaml
labels:
  - wud.watch=true
  - 'wud.tag.include=^\d+\.\d+\.\d+$$'   # tags suivis, expression régulière
  - wud.display.name=Authentik            # nom affiché
  - wud.display.icon=sh-authentik         # icône selfh.st (sh-) ou dashboard-icons (hl-)
```

Expressions selon le format des tags publiés :

| Format | Exemple | `wud.tag.include` |
| --- | --- | --- |
| Version sémantique | `1.32.0` | `^\d+\.\d+\.\d+$$` |
| Préfixe `v` | `v1.8.2` | `^v\d+\.\d+\.\d+$$` |
| Version calendaire | `2026.8.5` | `^\d{4}\.\d+\.\d+$$` |

Le `$` final s'écrit `$$` dans un compose, sinon Docker Compose l'interprète comme une variable.

Autres labels utiles :

```yaml
labels:
  - "wud.watch.digest=true"    # suivre l'empreinte d'un tag sans version
  - "wud.watch.delay=24h"      # attendre 24 h après publication
```

### Connexion

Connexion OIDC par Authentik (application `wud`, fournisseur `IMS-WUD-Provider`, URL de redirection stricte `https://wud.ims-world.fr/auth/oidc/authentik/cb`). Le compte natif reste actif comme accès de secours si Authentik est indisponible.

Variables, déclarées dans le compose et renseignées dans l'onglet **Environment Variables** de Coolify :

```env
WUD_AUTH_OIDC_AUTHENTIK_DISCOVERY=https://auth.ims-world.fr/application/o/wud/.well-known/openid-configuration
WUD_AUTH_OIDC_AUTHENTIK_CLIENTID=<identifiant>
WUD_AUTH_OIDC_AUTHENTIK_CLIENTSECRET=<secret>
WUD_AUTH_OIDC_AUTHENTIK_REDIRECT=false   # page de connexion avec les deux méthodes
```

Le segment `AUTHENTIK` du nom des variables doit correspondre au segment `/authentik/` de l'URL de redirection. WUD ne gère pas de rôles : toute personne connectée a un accès complet, le filtrage se fait par la liaison de groupe dans Authentik.

<Warning>
WUD lit le socket Docker, qui expose les variables d'environnement de tous les conteneurs, donc leurs secrets. Le service reste restreint au tailnet en plus de l'OIDC.
</Warning>

Procédure générale : [Connecter une application à Authentik en OIDC](/procedures/integration-service-authentik-oidc).

## Données et sauvegarde

Sauvegardé avec la VM 104 ; l'état se reconstruit à la première analyse.

## Exploitation

Surveiller un nouveau service :

1. Ajouter les labels au conteneur principal, dans l'éditeur de compose de Coolify (service) ou le champ **Container Labels** (application, à la suite des labels existants). Pour `coolify-proxy` : **Servers → Proxy**.
2. Redéployer : un label ne s'applique qu'à la recréation du conteneur.
3. Vérifier le label reçu, qui doit se terminer par un seul `$` :

   ```bash
   docker inspect <conteneur> --format '{{ index .Config.Labels "wud.tag.include" }}'
   ```

4. Lancer **Watch Now** dans WUD et vérifier que le conteneur apparaît.

Après une alerte : lire les notes de version, changer le tag dans Coolify, redéployer, puis mettre à jour `app_version` de la fiche du service.

## Pièges connus

Chaque piège donne le symptôme observé, sa cause et la correction.

<AccordionGroup>
  <Accordion title="Fausse nouvelle version de qBittorrent (20.04.1)">
    **Cause** : tag de l'image de base pris pour une version.

    **Solution** : restreindre les tags suivis par label (`wud.tag.include`).
  </Accordion>
  <Accordion title="Mises à jour absurdes proposées (19rc1-trixie, 32bit-stretch, pré-version Windows)">
    **Cause** : surveillance de tous les conteneurs et tags non sémantiques (`15-alpine`, `7-alpine`) ; WUD compare le premier nombre sans tenir compte du suffixe.

    **Solution** : surveillance sur liste (`WATCHBYDEFAULT=false`) et `wud.tag.include` sur chaque conteneur surveillé.
  </Accordion>
  <Accordion title="Expression régulière sans effet">
    **Cause** : `$` final interprété par Docker Compose comme une variable.

    **Solution** : écrire `$$` dans le compose et vérifier avec `docker inspect`. Dans le champ **Container Labels** d'une application, vérifier aussi : si le conteneur reçoit `$$`, Coolify a déjà échappé, remettre un seul `$`.
  </Accordion>
  <Accordion title="Nom de conteneur préfixé (ff1132b3df68_wud-…) ou suffixé par l'UUID Coolify">
    **Cause** : WUD a enregistré le conteneur pendant que Docker Compose le renommait lors d'une recréation ; le nom technique Coolify porte toujours l'UUID.

    **Solution** : label `wud.display.name` sur chaque conteneur surveillé.
  </Accordion>
  <Accordion title="Erreur Authentik à la connexion OIDC">
    **Cause** : identifiant de client d'un autre fournisseur (fournisseur recréé), alors que l'URL de redirection était correcte.

    **Solution** : comparer le `client_id` de l'URL d'autorisation avec celui de `IMS-WUD-Provider`, reporter l'identifiant et le secret de ce fournisseur, redéployer. Contrôle sans afficher le secret :

    ```bash
    docker exec wud-qwe5jrqlqtwqneevgkf6mwr9 env | grep OIDC | sed 's/SECRET=.*/SECRET=***/'
    ```
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card color="#16A34A" title="Ntfy" icon="bell" href="/services/ntfy">Réception des alertes.</Card>
  <Card color="#16A34A" title="Authentik" icon="key" href="/services/authentik">Fournisseur d'identité.</Card>
</CardGroup>
