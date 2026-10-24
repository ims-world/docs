---
title: "ims-pbs (LXC 103)"
description: "Proxmox Backup Server du homelab : sauvegardes dédupliquées des invités du pool backup, stockées sur le HDD du NAS."
icon: "shield-halved"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

import { ips } from "/snippets/variables.mdx";

## Fiche

| Propriété | Valeur |
| --- | --- |
| Type | Conteneur LXC privilégié, fonctionnalité `mount=nfs` (montage NFS direct) |
| Nœud et VMID | `pve`, 103 |
| Noms | Proxmox `ims-pbs` · inventaire `ims-pbs` · tailnet `ims-pve-103-pbs` |
| OS | Debian, Proxmox Backup Server |
| CPU et RAM | 2 cœurs · 1 Go |
| Disques | Système local ; datastore sur NFS |
| IP LAN | <code>{ips.pbsLan}</code> ; réseau interne `vmbr1` : `10.10.10.3` |
| IP tailnet | <code>{ips.pbs}</code> |
| Sauvegarde | Hors pool `backup` : se réinstalle et se rattache au datastore |
| Statut | En service |

Interface web : <code>https://{ips.pbsLan}:8007</code>.

## Datastore

| Propriété | Valeur |
| --- | --- |
| Nom | `pve-backups` |
| Chemin | `/mnt/pbs-datastore` |
| Source | `10.10.10.1:/mnt/storage/backups` (HDD du NAS) |
| Stockage côté Proxmox VE | `pbs-coolify` |

```text
# /etc/fstab
10.10.10.1:/mnt/storage/backups  /mnt/pbs-datastore  nfs  defaults,nofail,_netdev,vers=3  0 0
```

Le montage reste en NFS v3 : ce choix contournait un problème de MergerFS, retiré depuis. Le retour en v4.2 est possible mais non nécessaire.

## Tâches planifiées

| Tâche | Où | Planification | Détail |
| --- | --- | --- | --- |
| Sauvegarde du pool `backup` | Proxmox VE (Datacenter → Backup) | Tous les jours à 02:00 | Voir la [politique de sauvegarde](/procedures/politique-sauvegardes) |
| Élagage (prune) | PBS | Quotidien | 7 quotidiennes, 4 hebdomadaires, 3 mensuelles |
| Nettoyage (garbage collection) | PBS | Samedi 03:00 | Libère les blocs non référencés |
| Vérification (verify) | PBS | Dimanche 04:00 | Détecte une corruption silencieuse |

Les échecs sont notifiés sur Ntfy, depuis Proxmox VE et depuis le PBS (mode de notification du datastore : **Notification System**).

## Particularités

- Le PBS ne sauvegarde ni le NAS ni lui-même : il en dépend pour fonctionner.
- Le stockage `pbs-coolify` se connecte encore avec `root@pam`. Le remplacement par un utilisateur PBS limité à la sauvegarde est suivi dans la [feuille de route](/roadmap).
- En cas d'erreur `namespace not found` ou `ESTALE` à la sauvegarde, le datastore ne répond plus : remonter le partage et redémarrer les services PBS (voir [Dépannage](/procedures/depannage-courant)).
- Écarts au [socle commun](/infrastructure/socle-commun) : envoi Alloy par `10.10.10.2`.

## Sauvegarde et reprise

En cas de perte du conteneur, réinstaller un PBS, remonter le partage NFS et rattacher le datastore existant : les sauvegardes sont sur le NAS, pas dans le conteneur. Voir le [plan de reprise](/procedures/plan-reprise-activite-pra).

## Supervision

Agent Alloy, collecteur `hwmon` désactivé. Notifications PBS sur Ntfy.

## Voir aussi

- [ims-nas](/infrastructure/ims-nas)
- [Incident du 23/10/2026](/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix)
