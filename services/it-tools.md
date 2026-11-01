---
title: "IT-Tools"
description: "Boîte à outils en ligne : générateurs, convertisseurs, encodeurs, utilitaires réseau."
icon: "toolbox"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "latest"
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
| Image | `corentinth/it-tools:latest` |
| Données | Volume `it-tools-data` (préférences) |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 11/08/2026 |

## Fonctionnement

Application web statique : tous les calculs se font dans le navigateur. Sans authentification propre, elle est placée derrière le [forward-auth](/procedures/securiser-application-authentik-forward-auth) d'Authentik.

## Données et sauvegarde

Sans objet : aucune donnée utilisateur.

## Exploitation

L'image est en `latest`, contrairement à la règle des tags figés. L'épinglage sur un tag daté est suivi dans la [feuille de route](/roadmap).

## Pièges connus

Sans objet.

## Voir aussi

<CardGroup cols={2}>
  <Card color="#FF6B00" title="Authentik" icon="key" href="/services/authentik">Connexion unique des services.</Card>
</CardGroup>
