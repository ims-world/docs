---
title: "Mac Mini (pve-macmini)"
description: "Second nœud du cluster Proxmox ims-cluster : héberge le worker Coolify et Home Assistant."
icon: "apple"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

import { ips } from "/snippets/variables.mdx";

## Fiche

Second nœud du cluster, plus modeste : il porte le worker Coolify et Home Assistant, et apporte le second vote du quorum.

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

Deux invités, tous deux dans le pool de sauvegarde.

<CardGroup cols={2}>
  <Card color="#3B82F6" title="105 · ims-coolify-worker" icon="cubes" href="/infrastructure/lxc-coolify-worker">
    LXC non privilégié, Debian 13. Worker Coolify.
  </Card>
  <Card color="#3B82F6" title="106 · Home Assistant" icon="house-signal" href="/infrastructure/vm-home-assistant">
    VM, Home Assistant OS. Domotique.
  </Card>
</CardGroup>

## Limites du nœud

Ce que le Mac Mini ne peut pas faire, à garder en tête avant d'y placer un invité.

- 16 Go de RAM et pas d'iGPU partageable : le Mac Mini ne peut pas reprendre la VM 104 en cas de panne du MS-01.
- Ports USB 3.0 uniquement : une clé radio Zigbee doit passer par une rallonge USB 2.0 blindée (voir [Home Assistant](/infrastructure/vm-home-assistant)).
- Le cluster n'a que deux votes : la perte d'un nœud fait perdre le quorum. Un QDevice sur le Raspberry Pi est suivi dans la [feuille de route](/roadmap).

## Particularités

Ce qui distingue ce nœud des autres hôtes du parc.

- Écarts au [socle commun](/infrastructure/socle-commun) : root par clé (cluster), mises à jour manuelles.
- Démarrage EFI natif, sans patch de noyau. Carte réseau Broadcom `tg3` reconnue nativement.
- Dépôts `pve-enterprise.sources` et `ceph.sources` désactivés, `pve-no-subscription` activé.

## Sauvegarde et reprise

Les invités 105 et 106 sont sauvegardés chaque nuit dans le PBS (pool `backup`). Le nœud se réinstalle puis rejoint le cluster (`pvecm add`).

## Supervision

Agent Alloy avec le collecteur SMART du SSD interne (`/etc/cron.d/smartmon`, toutes les 5 minutes). Envoi vers `192.168.1.52`.

## Voir aussi

<CardGroup cols={2}>
  <Card color="#3B82F6" title="MS-01 (pve)" icon="server" href="/infrastructure/proxmox-host">Nœud principal du cluster.</Card>
  <Card color="#8B5CF6" title="Politique de sauvegarde" icon="shield" href="/procedures/politique-sauvegardes">Ce qui est sauvegardé, quand, où.</Card>
</CardGroup>
