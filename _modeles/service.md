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

<Une phrase : qui accède, depuis où.>

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

<Une phrase sur le flux.>

<Schéma Mermaid si plus de deux composants.>

<CardGroup cols={2}>
  <Card title="<Composant>" icon="<icône>"><Rôle du composant.></Card>
</CardGroup>

## Données et sauvegarde

<Où vivent les données, ce qui est sauvegardé, comment restaurer.>

## Exploitation

<Une phrase d'introduction.>

<AccordionGroup>
  <Accordion title="<Opération>" icon="<icône>">
    <Commandes et explications.>
  </Accordion>
</AccordionGroup>

## Pièges connus

<AccordionGroup>
  <Accordion title="<Symptôme>">
    **Cause** : <…>

    **Solution** : <…>
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card title="<Page liée>" icon="<icône>" href="/<chemin>"><Pourquoi la lire.></Card>
</CardGroup>
