---
title: "Mac Mini (pve-macmini)"
description: "Second nœud du cluster Proxmox ims-cluster : héberge le worker Coolify et Home Assistant."
icon: "apple"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

import { ips } from "/snippets/variables.mdx";

## Fiche

| Propriété | Valeur |
| --- | --- |
| Type | Machine physique, Apple Mac Mini Late 2012 (Macmini6,1) |
| Nœud et VMID | Nœud `pve-macmini` du cluster `ims-cluster` |
| Noms | Proxmox `pve-macmini` · système `pve-macmini` · inventaire `pve-macmini` · tailnet `ims-pve-macmini` |
| OS | Proxmox VE 9.2.11 (Debian 13) |
| CPU et RAM | Intel Core i5 double cœur, 4 threads · 16 Go DDR3 |
| Disques | SSD SATA Apple `SM0256F`, 256 Go |
| IP LAN | `192.168.1.42` |
| IP tailnet | <code>{ips.macmini}</code> |
| Sauvegarde | Non sauvegardé : se réinstalle ; ses invités sont dans le pool `backup` |
| Statut | En service |

## Ce qui tourne dessus

| VMID | Nom | Type | Rôle |
| --- | --- | --- | --- |
| 105 | `ims-coolify-worker` | LXC non privilégié, Debian 13 | [Worker Coolify](/infrastructure/lxc-coolify-worker) |
| 106 | Home Assistant | VM, Home Assistant OS | [Domotique](/infrastructure/vm-home-assistant) |

## Limites du nœud

- 16 Go de RAM et pas d'iGPU partageable : le Mac Mini ne peut pas reprendre la VM 104 en cas de panne du MS-01.
- Ports USB 3.0 uniquement : une clé radio Zigbee doit passer par une rallonge USB 2.0 blindée (voir [Home Assistant](/infrastructure/vm-home-assistant)).
- Le cluster n'a que deux votes : la perte d'un nœud fait perdre le quorum. Un QDevice sur le Raspberry Pi est suivi dans la [feuille de route](/roadmap).

## Particularités

- Écarts au [socle commun](/infrastructure/socle-commun) : root par clé (cluster), mises à jour manuelles.
- Démarrage EFI natif, sans patch de noyau. Carte réseau Broadcom `tg3` reconnue nativement.
- Dépôts `pve-enterprise.sources` et `ceph.sources` désactivés, `pve-no-subscription` activé.

## Sauvegarde et reprise

Les invités 105 et 106 sont sauvegardés chaque nuit dans le PBS (pool `backup`). Le nœud se réinstalle puis rejoint le cluster (`pvecm add`).

## Supervision

Agent Alloy avec le collecteur SMART du SSD interne (`/etc/cron.d/smartmon`, toutes les 5 minutes). Envoi vers `192.168.1.52`.

## Voir aussi

- [MS-01 (pve)](/infrastructure/proxmox-host)
- [Politique de sauvegarde](/procedures/politique-sauvegardes)
