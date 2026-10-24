---
title: "ADR-014 — Sauvegarde par pool unique"
description: "Un pool Proxmox désigne les machines sauvegardées, un seul job PBS sauvegarde tout le pool, et les échecs sont notifiés sur Ntfy."
icon: "scale-balanced"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

## Statut

Accepté — 24/10/2026

## Contexte

Jusqu'en octobre 2026, seule la VM 104 était sauvegardée dans le PBS, par un job qui la désignait nommément. Le worker 105 et Home Assistant (106), sur le second nœud, n'étaient pas couverts. Les échecs partaient vers la boîte locale `root`, que personne ne lisait : les sauvegardes de la VM 104 ont échoué cinq nuits de suite sans être remarquées ([incident du 23/10/2026](/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix)).

## Décision

- Un pool Proxmox `backup` liste les machines à sauvegarder.
- Un seul job, sans restriction de nœud, sauvegarde le pool chaque nuit à 02:00 vers le stockage `pbs-coolify`, en mode snapshot.
- Ajouter ou retirer une machine se fait par le pool : `just backup-add`, `just backup-remove` (ims-ops).
- La rétention reste portée par le PBS.
- Proxmox VE et le PBS notifient les avertissements et les échecs sur Ntfy, avec un utilisateur Ntfy dédié en écriture seule et un jeton par outil.
- Le NAS et le PBS restent hors du pool.

## Alternatives écartées

| Option | Raison du rejet |
| --- | --- |
| Un job par machine | Chaque nouvelle machine exige un job ; un oubli passe inaperçu |
| Un job par nœud | Une machine migrée d'un nœud à l'autre change de job, ou n'en a plus |
| Sélection « toutes les machines » | Inclurait le NAS et le PBS, et les VM de test |
| Notifications par courriel | Pas de serveur de messagerie ; Ntfy est déjà utilisé pour les autres alertes |

## Conséquences

- La couverture se lit en une commande : `just backup-list`.
- Une VM créée par `just vm` en mode `full` est ajoutée au pool automatiquement.
- Un échec de sauvegarde produit une alerte sur le téléphone le matin même.
- Les données stockées sur le NAS (photos, dépôts, médiathèque) restent hors de ce dispositif : leur protection est un chantier distinct.
