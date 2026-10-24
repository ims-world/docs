---
title: "Forgejo"
description: "Forge Git auto-hébergée : dépôts, tickets, demandes de fusion, et miroir de sécurité des dépôts GitHub."
icon: "code-branch"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "11"
---

## Accès

<CardGroup cols={2}>
  <Card title="Forgejo" icon="code-branch" href="https://forge.ims-world.fr">
    Interface web.
  </Card>
  <Card title="Git en SSH" icon="terminal">
    `ssh://git@forge.ims-world.fr:2222/<compte>/<dépôt>.git`
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
cd /data/coolify/services/culcigf0vwg0fbdvegbkzoan/ && docker compose logs -f --tail=100
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `forge.ims-world.fr` ; SSH sur le port 2222 |
| Accès | Public ; OIDC Authentik, inscriptions fermées (`DISABLE_REGISTRATION=true`) ; clés SSH pour Git |
| Hôte | VM 104 |
| Coolify | UUID `culcigf0vwg0fbdvegbkzoan`, chemin `/data/coolify/services/culcigf0vwg0fbdvegbkzoan/` |
| Image | `codeberg.org/forgejo/forgejo:11`, `postgres:16-alpine` |
| Données | `./data` (dépôts, LFS, avatars) et `./postgres` dans le dossier du service |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 14/08/2026 |

<Info>
Un dossier `forgejo-data` existe aussi sur le SSD du NAS (`/mnt/storage-hot`). L'emplacement réellement utilisé par le conteneur est à confirmer (`docker inspect`) : s'il s'agit du NAS, les dépôts ne sont pas dans la sauvegarde de la VM 104.
</Info>

## Fonctionnement

| Conteneur | Rôle |
| --- | --- |
| `forgejo` | Application, HTTP 3000 derrière Traefik, SSH 2222 publié directement |
| `forgejo-db` | PostgreSQL 16 |

- Le port 2222 est redirigé par la Bbox et publié par Docker (`ports: '2222:2222'`), sans passer par Traefik ni Authentik. La sécurité repose sur les clés SSH des comptes Forgejo ([ADR-006](/history/adr/adr-006-exposition-port-ssh-forgejo-bbox)).
- `extra_hosts: auth.ims-world.fr:host-gateway` : le conteneur joint Authentik par l'hôte pour valider les jetons.
- Courriels sortants par Resend (`smtp.resend.com:587`, expéditeur `forgejo@ims-world.fr`).

## Miroirs GitHub

GitHub reste la plateforme principale ; Forgejo garde une copie à jour de plusieurs dépôts, synchronisée automatiquement (vers 04:00). Chaque miroir utilise un jeton d'accès personnel GitHub (portée `repo`), à renouveler à son expiration : **Paramètres du dépôt → Miroir**.

## Données et sauvegarde

Base et dépôts sauvegardés avec la VM 104 (sous réserve de l'emplacement signalé plus haut). Les dépôts miroirs se reconstruisent depuis GitHub.

## Exploitation

Mise à jour : changer le tag dans Coolify, redéployer. Une version majeure peut migrer la base : sauvegarder la VM avant.

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| Miroirs arrêtés | Jeton GitHub expiré | Renouveler le jeton dans chaque dépôt miroir |
| Connexion OIDC en échec | Le conteneur ne joint pas `auth.ims-world.fr` | Vérifier `extra_hosts` |

## Voir aussi

- [ADR-006](/history/adr/adr-006-exposition-port-ssh-forgejo-bbox)
- [Matrice d'exposition](/reseau/matrice-securite-exposition)
