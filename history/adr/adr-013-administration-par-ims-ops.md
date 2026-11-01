---
title: "ADR-013 — Administration du parc par ims-ops"
description: "La configuration des hôtes, la création de VM et le pool de sauvegarde passent par le dépôt ims-ops (Ansible et just), qui remplace IMS-INFRA."
icon: "scale-balanced"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

## Statut

Accepté — 23/10/2026, complété le 24/10/2026. Remplace l'[ADR-011](/history/adr/adr-011-organisation-ims-infra-par-lots).

## Contexte

Le dépôt IMS-INFRA, organisé en lots avec un Makefile, couvrait mal le parc réel : la production avait été configurée à la main, le template 9000 diffusait une clé Headscale en clair par Cloud-Init, et rien ne permettait de vérifier qu'un hôte était conforme à l'état attendu.

## Décision

| Sujet | Décision |
| --- | --- |
| Outil | Ansible pour l'état des machines, `just` comme point d'entrée unique |
| Source des valeurs | L'inventaire seul ; les mêmes valeurs servent d'entrée aux rôles et d'état attendu pour `just audit` |
| Inventaire | `hosts.yml` (tenu à la main) et `vms-created.yml` (écrit par `just vm`) ; la destruction est limitée au second |
| Connexion | Compte personnel et sudo, jamais root ; IP tailnet par défaut, LAN en secours ; rebond par le nœud pour une VM pas encore dans le tailnet |
| Secrets | Ansible Vault, mot de passe dans le trousseau macOS ; un jeton par outil |
| Périmètre | Comptes, socle système, Alloy, Tailscale, template et VM, pool de sauvegarde, rapports en lecture seule |
| Cloud-Init | Minimal (nom, compte, clé) ; plus de clé dans un snippet |
| Contrôle | `just check` avant tout `just setup`, arrêt au premier hôte en échec |
| Documentation | Le dépôt donne les commandes ; cette documentation explique l'état et les choix |

## Alternatives écartées

| Option | Raison du rejet |
| --- | --- |
| Terraform ou OpenTofu | Second outil et fichier d'état à protéger, pour quelques VM |
| Packer | Un seul template : un playbook suffit |
| AWX, Semaphore | Trop lourds pour la taille du parc ; Semaphore reporté |
| Molecule | La simulation sur le parc réel suffit |
| Agent CrowdSec sur les hôtes | Doublon avec CrowdSec devant Traefik ; les hôtes ne sont pas exposés |

## Conséquences

- Le socle des hôtes est décrit une fois ([socle commun](/infrastructure/socle-commun)) et vérifiable par `just audit`.
- Une nouvelle VM est configurée, enrôlée et sauvegardée en une commande.
- Les mises à jour des nœuds Proxmox, les services Docker et les appliances restent hors du dépôt.
- Voir [Administration avec ims-ops](/procedures/administration-ims-ops).
