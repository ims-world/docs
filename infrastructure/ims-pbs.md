---
title: "ims-pbs (LXC 103)"
description: "Proxmox Backup Server du homelab : sauvegardes dédupliquées des invités du pool backup, stockées sur le HDD du NAS."
icon: "shield-halved"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

import { ips } from "/snippets/variables.mdx";

## Fiche

Le PBS reçoit chaque nuit les sauvegardes des invités du pool `backup`. Son datastore est sur le HDD du NAS, pas dans le conteneur.

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
| Sauvegarde | Hors pool `backup` ; archive locale chaque nuit à 03:00 sur le stockage `local` de `pve` (4 conservées) |
| Statut | En service |

<CardGroup cols={2}>
  <Card title="Interface web (tailnet)" icon="globe" href={`https://${ips.pbs}:8007`}>
    <code>https://{ips.pbs}:8007</code>
  </Card>
  <Card title="Interface web (LAN)" icon="house" href={`https://${ips.pbsLan}:8007`}>
    <code>https://{ips.pbsLan}:8007</code>
  </Card>
</CardGroup>

## Datastore

Un seul datastore, monté en NFS depuis le NAS.

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

<Note>
Le montage reste en NFS v3 : ce choix contournait un problème de MergerFS, retiré depuis. Le retour en v4.2 est possible mais non nécessaire.
</Note>

## Tâches planifiées

La sauvegarde est lancée par Proxmox VE ; l'entretien du datastore est fait par le PBS.

<Steps>
  <Step title="Sauvegarde du pool backup, chaque jour à 02:00">
    Lancée par Proxmox VE (**Datacenter → Backup**). Voir la [politique de sauvegarde](/procedures/politique-sauvegardes).
  </Step>
  <Step title="Élagage, chaque jour">
    Conserve 7 sauvegardes quotidiennes, 4 hebdomadaires et 3 mensuelles.
  </Step>
  <Step title="Nettoyage, le samedi à 03:00">
    Libère les blocs qui ne sont plus référencés (garbage collection).
  </Step>
  <Step title="Vérification, le dimanche à 04:00">
    Relit les blocs pour détecter une corruption silencieuse.
  </Step>
</Steps>

Les échecs sont notifiés sur Ntfy, depuis Proxmox VE et depuis le PBS (mode de notification du datastore : **Notification System**).

## Particularités

Ce qui distingue ce conteneur des autres hôtes du parc.

- Le PBS ne sauvegarde ni le NAS ni lui-même : il en dépend pour fonctionner.
- Le stockage `pbs-coolify` se connecte encore avec `root@pam`. Le remplacement par un utilisateur PBS limité à la sauvegarde est suivi dans la [feuille de route](/roadmap).
- En cas d'erreur `namespace not found` ou `ESTALE` à la sauvegarde, le datastore ne répond plus : remonter le partage et redémarrer les services PBS (voir [Dépannage](/procedures/depannage-courant)).
- Écarts au [socle commun](/infrastructure/socle-commun) : envoi Alloy par `10.10.10.2`.

## Sauvegarde et reprise

En cas de perte du conteneur, réinstaller un PBS, remonter le partage NFS et rattacher le datastore existant : les sauvegardes sont sur le NAS, pas dans le conteneur. Voir le [plan de reprise](/procedures/plan-reprise-activite-pra).

## Supervision

Agent Alloy, collecteur `hwmon` désactivé. Notifications PBS sur Ntfy.

## Voir aussi

<CardGroup cols={2}>
  <Card title="ims-nas" icon="hard-drive" href="/infrastructure/ims-nas">Partages NFS et SMB.</Card>
  <Card title="Incident du 23/10/2026" icon="triangle-exclamation" href="/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix">Sauvegardes en échec par ESTALE.</Card>
</CardGroup>
