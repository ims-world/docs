---
title: "Forgejo"
description: "Forge Git auto-hébergée : dépôts, tickets, demandes de fusion, et miroir de sécurité des dépôts GitHub."
icon: "code-branch"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "11"
---

## Accès

Interface publique avec connexion Authentik ; Git en SSH par le port 2222.

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
| Données | Dépôts, LFS et avatars sur le SSD du NAS (`/mnt/nas-hot/forgejo-data`) ; base PostgreSQL sur la VM 104 |
| Sauvegarde | Base avec la VM 104 ; **dépôts non sauvegardés** |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 14/08/2026 |

## Fonctionnement

Deux conteneurs : l'application et sa base.

<CardGroup cols={2}>
  <Card title="forgejo" icon="code-branch">Application, HTTP 3000 derrière Traefik, SSH 2222 publié directement.</Card>
  <Card title="forgejo-db" icon="database">PostgreSQL 16.</Card>
</CardGroup>

- Le port 2222 est redirigé par la Bbox et publié par Docker (`ports: '2222:2222'`), sans passer par Traefik ni Authentik. La sécurité repose sur les clés SSH des comptes Forgejo ([ADR-006](/history/adr/adr-006-exposition-port-ssh-forgejo-bbox)).
- `extra_hosts: auth.ims-world.fr:host-gateway` : le conteneur joint Authentik par l'hôte pour valider les jetons.
- Courriels sortants par Resend (`smtp.resend.com:587`, expéditeur `forgejo@ims-world.fr`).

## Miroirs GitHub

GitHub reste la plateforme principale ; Forgejo garde une copie à jour de plusieurs dépôts, synchronisée automatiquement (vers 04:00). Chaque miroir utilise un jeton d'accès personnel GitHub (portée `repo`), à renouveler à son expiration : **Paramètres du dépôt → Miroir**.

## Données et sauvegarde

La base est sauvegardée avec la VM 104. Les dépôts sont sur le SSD du NAS, qui n'est pas sauvegardé : les dépôts miroirs se reconstruisent depuis GitHub, les dépôts propres à Forgejo seraient perdus avec le disque. Voir la [politique de sauvegarde](/procedures/politique-sauvegardes#ce-qui-nest-pas-sauvegardé).

## Exploitation

Mise à jour : changer le tag dans Coolify, redéployer. Une version majeure peut migrer la base : sauvegarder la VM avant.

## Pièges connus

Chaque piège donne le symptôme observé, sa cause et la correction.

<AccordionGroup>
  <Accordion title="Miroirs arrêtés">
    **Cause** : jeton GitHub expiré.

    **Solution** : renouveler le jeton dans chaque dépôt miroir.
  </Accordion>
  <Accordion title="Connexion OIDC en échec">
    **Cause** : le conteneur ne joint pas `auth.ims-world.fr`.

    **Solution** : vérifier `extra_hosts`.
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card title="ADR-006" icon="scale-balanced" href="/history/adr/adr-006-exposition-port-ssh-forgejo-bbox">Pourquoi le port SSH est exposé.</Card>
  <Card title="Matrice d'exposition" icon="table-cells" href="/reseau/matrice-securite-exposition">Zone et authentification de chaque service.</Card>
</CardGroup>
