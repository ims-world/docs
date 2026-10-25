---
title: "WUD (veille des images)"
description: "What's Up Docker : détecte les nouvelles versions des images utilisées sur la VM 104 et prévient sur Ntfy, sans rien modifier."
icon: "docker"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v9.0.2"
---

## Accès

Restreint au tailnet, avec compte natif.

<CardGroup cols={2}>
  <Card title="WUD" icon="docker" href="https://wud.ims-world.fr">
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
| Accès | Tailnet ; compte natif `cmolotkoff` |
| Hôte | VM 104 |
| Coolify | UUID `qwe5jrqlqtwqneevgkf6mwr9`, chemin `/data/coolify/services/qwe5jrqlqtwqneevgkf6mwr9/` |
| Image | `getwud/wud:9.0.2` |
| Données | `store/` (état SQLite) ; socket Docker en lecture seule |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 14/09/2026 |

## Fonctionnement

WUD liste les tags publiés sur les registres (Docker Hub, GHCR, Quay) et les compare au tag de chaque conteneur. Avec un tag figé (`1.32.0`), il signale une version corrective, mineure ou majeure. Il n'applique aucune mise à jour.

- Analyse une fois par jour à 08:00 (`0 8 * * *`), pour rester sous le quota anonyme de Docker Hub.
- Notifications sur le topic Ntfy `ims-alerts`, avec un jeton dédié (`WUD_TRIGGER_NTFY_DEFAULT_AUTH_TOKEN`).
- Une image en `latest` n'a pas de version à comparer : WUD ne peut suivre que son empreinte. Les services concernés sont listés dans la [feuille de route](/roadmap).

Réglages par conteneur, par labels :

```yaml
labels:
  - "wud.watch=false"          # ne pas surveiller
  - "wud.watch.digest=true"    # suivre l'empreinte d'un tag sans version
  - "wud.watch.delay=24h"      # attendre 24 h après publication
```

## Données et sauvegarde

Sauvegardé avec la VM 104 ; l'état se reconstruit à la première analyse.

## Exploitation

Après une alerte : lire les notes de version, changer le tag dans Coolify, redéployer, puis mettre à jour `app_version` de la fiche du service.

## Pièges connus

Chaque piège donne le symptôme observé, sa cause et la correction.

<AccordionGroup>
  <Accordion title="Fausse nouvelle version de qBittorrent (20.04.1)">
    **Cause** : tag de l'image de base pris pour une version.

    **Solution** : restreindre les tags suivis par label (`wud.tag.include`).
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card title="Ntfy" icon="bell" href="/services/ntfy">Réception des alertes.</Card>
</CardGroup>
