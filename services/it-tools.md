---
title: "IT-Tools"
description: "Boîte à outils en ligne : générateurs, convertisseurs, encodeurs, utilitaires réseau."
icon: "toolbox"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "2024.10.22-7ca5933"
---

## Accès

Public, derrière une connexion Authentik.

<CardGroup cols={2}>
  <Card color="#16A34A" title="IT-Tools" icon="toolbox" href="https://tools.ims-world.fr">
    Liste des outils, recherche.
  </Card>
</CardGroup>

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `tools.ims-world.fr` |
| Accès | Public ; forward-auth Authentik (administrateurs, `membres`, `invites`) |
| Hôte | VM 104 |
| Coolify | UUID `yefujwl3pxvum45edpsbsru7`, chemin `/data/coolify/services/yefujwl3pxvum45edpsbsru7/` |
| Image | `corentinth/it-tools:2024.10.22-7ca5933` |
| Données | Volume `it-tools-data` (préférences) |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 11/08/2026 |

## Fonctionnement

Application web statique : tous les calculs se font dans le navigateur. Sans authentification propre, elle est placée derrière le [forward-auth](/procedures/securiser-application-authentik-forward-auth) d'Authentik.

## Données et sauvegarde

Sans objet : aucune donnée utilisateur.

## Exploitation

Version épinglée sur le dernier tag publié, daté d'octobre 2024 : le projet n'a plus publié de version depuis. Le risque reste faible, l'application étant statique et tous les calculs faits dans le navigateur.

## Pièges connus

Sans objet.

## Voir aussi

<CardGroup cols={2}>
  <Card color="#FF6B00" title="Authentik" icon="key" href="/services/authentik">Connexion unique des services.</Card>
</CardGroup>
