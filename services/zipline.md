---
title: "Zipline"
description: "Partage de fichiers et de captures d'écran (compatible ShareX) et raccourcisseur de liens."
icon: "share-nodes"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "latest"
---

## Accès

<CardGroup cols={2}>
  <Card title="Zipline" icon="share-nodes" href="https://share.ims-world.fr">
    Fichiers, liens courts, jetons d'API.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
cd /data/coolify/services/kbcknnnkswmcnlgmupxoyheh/ && docker compose logs -f --tail=100
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `share.ims-world.fr` |
| Accès | Public ; OIDC Authentik ; jetons d'API pour les envois automatisés |
| Hôte | VM 104 |
| Coolify | UUID `kbcknnnkswmcnlgmupxoyheh`, chemin `/data/coolify/services/kbcknnnkswmcnlgmupxoyheh/` |
| Image | `ghcr.io/diced/zipline:latest`, `postgres:16-alpine` |
| Données | `./uploads` (fichiers), `./postgres` (base) dans le dossier du service |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 14/08/2026 |

## Fonctionnement

Zipline reçoit des fichiers par l'interface ou par son API (ShareX sous Windows, scripts), et sert des liens publics. La connexion passe par Authentik en OIDC ; le conteneur joint `auth.ims-world.fr` par l'hôte (`host-gateway`).

## Données et sauvegarde

Fichiers et base dans le dossier du service, sauvegardés avec la VM 104.

## Exploitation

L'image est en `latest`. Zipline publie pourtant des tags versionnés : l'épinglage est suivi dans la [feuille de route](/roadmap).

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `share.ims-world.fr` en 503 alors que le conteneur répond | Enregistrements `_acme-challenge.share` oubliés : le générique DNS ne s'applique plus | Les supprimer dans la zone OVH ([incident du 24/08/2026](/history/incidents/2026-08-24-dns-wildcard-shadowing-share-ims-world-fr)) |

## Voir aussi

- [Connecter une application en OIDC](/procedures/integration-service-authentik-oidc)
