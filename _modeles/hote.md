---
title: "<Nom de l'hôte>"
description: "<Ce qu'est l'hôte et son rôle, en une phrase.>"
icon: "server"
iconType: "duotone"
last_reviewed: "AAAA-MM-JJ"
---

import { ips } from "/snippets/variables.mdx";

## Rôle

<Facultative.>

## Fiche

| Propriété | Valeur |
| --- | --- |
| Type | <Machine physique, VM ou conteneur LXC (privilégié ou non)> |
| Nœud et VMID | <pve, 104> |
| Noms | Proxmox `<nom>` · système `<nom>` · inventaire `<nom>` |
| OS | <Distribution et version> |
| CPU et RAM | <cœurs, Go> |
| Disques | <Disques et stockage> |
| IP LAN | `192.168.1.<x>` |
| IP tailnet | `100.64.0.<x>` |
| Sauvegarde | <Pool `backup`, exclu, autre> |
| Statut | <En service, arrêté, en test> |

## Ce qui tourne dessus

<Invités, services, partages.>

## Particularités

<Écarts par rapport au socle commun des hôtes, pièges propres à cet hôte.>

## Sauvegarde et reprise

<Couverture, exclusions, renvoi vers le plan de reprise.>

## Supervision

<Agent Alloy, tableaux de bord, sondes Uptime Kuma.>

## Voir aussi

- [<Page liée>](/<chemin>)
