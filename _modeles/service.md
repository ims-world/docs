---
title: "<Nom du service>"
description: "<Ce qu'est le service et à quoi il sert, en une phrase.>"
icon: "<icône>"
iconType: "duotone"
last_reviewed: "AAAA-MM-JJ"
app_version: "<tag déployé>"
---

import { ips, domains } from "/snippets/variables.mdx";

## Rôle

<Facultative : à garder seulement si la description ne suffit pas.>

## Accès

<CardGroup cols={2}>
  <Card title="<Interface>" icon="globe" href="https://<domaine>">
    <Ce que l'on y fait.>
  </Card>
</CardGroup>

```bash
# Commandes courantes, depuis la VM 104
docker logs --tail 100 -f <conteneur>
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `<domaine>` |
| Accès | <Zone (public, tailnet)> ; <authentification> ; voir la [matrice](/reseau/matrice-securite-exposition) |
| Hôte | <VM 104 ou worker 105> |
| Coolify | UUID `<uuid>`, chemin `/data/coolify/services/<uuid>/` |
| Image | `<image>:<tag>` |
| Données | <Volumes et emplacements> |
| Sauvegarde | <Couverture, ex. : sauvegarde de la VM 104 (pool `backup`)> |
| Supervision | <Sonde Uptime Kuma, tableau de bord> |
| Mise en service | JJ/MM/AAAA |

## Fonctionnement

<Composants et flux. Schéma Mermaid si plus de deux composants.>

## Données et sauvegarde

<Où vivent les données, ce qui est sauvegardé, comment restaurer.>

## Exploitation

<Opérations courantes.>

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| <…> | <…> | <…> |

## Voir aussi

- [<Page liée>](/<chemin>)
