---
title: "Politique de sauvegarde"
description: "Ce qui est sauvegardé, où, quand et avec quelle rétention ; ce qui ne l'est pas ; comment sauvegarder à la main avant une intervention."
icon: "shield-check"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

## Principes

- Les invités sont sauvegardés au niveau de l'hyperviseur par le [PBS](/infrastructure/ims-pbs) : système, configuration et volumes Docker compris. Aucune application ne gère sa propre sauvegarde.
- Un pool Proxmox, `backup`, désigne les machines sauvegardées. Un seul job, sans restriction de nœud, sauvegarde tout le pool. Ajouter une machine revient à l'ajouter au pool ([ADR-014](/history/adr/adr-014-sauvegarde-par-pool-unique)).
- Le NAS et le PBS ne sont pas dans le pool : le datastore est sur le NAS et géré par le PBS, ils ne peuvent pas se sauvegarder eux-mêmes.
- Les échecs sont notifiés sur Ntfy, par Proxmox VE et par le PBS.

## Couverture

| Machine | Sauvegarde | Restauration |
| --- | --- | --- |
| VM 104 `ims-coolify` | Pool `backup`, chaque nuit | Depuis le PBS |
| LXC 105 worker Coolify | Pool `backup`, chaque nuit | Depuis le PBS |
| VM 106 Home Assistant | Pool `backup`, chaque nuit | Depuis le PBS |
| VM 102 Windows | À ajouter au pool (`just backup-add 102`) | — |
| VM 107 `lab-elo` | À décider avec son utilisatrice | — |
| LXC 103 `ims-pbs` | Locale, sur le stockage `local` du nœud | Ou réinstallation et rattachement du datastore |
| LXC 100 `ims-nas` | Manuelle, avant intervention | Ou recréation à l'identique |
| Nœuds `pve` et `pve-macmini` | Aucune | Réinstallation |

`just backup-list` affiche l'état réel du pool.

### Ce qui n'est pas sauvegardé

| Données | Emplacement | Conséquence d'une perte |
| --- | --- | --- |
| Médiathèque HomeFlix | HDD du NAS | Se retélécharge |
| Photos Immich, dépôts Forgejo | SSD du NAS (`storage-hot`) | Perte définitive : aucune copie connue |
| Originaux PhotoPrism | HDD du NAS | Perte définitive : aucune copie connue |
| Sauvegardes PBS elles-mêmes | HDD du NAS | Plus aucune sauvegarde des invités |

Le datastore PBS et la médiathèque partagent le même disque. La copie hors site des sauvegardes et la protection des données du NAS sont suivies dans la [feuille de route](/roadmap).

## Planification

| Heure | Tâche | Où |
| --- | --- | --- |
| 02:00 chaque jour | Sauvegarde du pool `backup`, mode snapshot | Proxmox VE (**Datacenter → Backup**) |
| 03:00 chaque jour | Sauvegarde locale de LXC 103 | Proxmox VE, stockage `local` |
| Chaque jour | Élagage : 7 quotidiennes, 4 hebdomadaires, 3 mensuelles | PBS |
| Samedi 03:00 | Nettoyage des blocs inutilisés | PBS |
| Dimanche 04:00 | Vérification d'intégrité | PBS |

La rétention est portée par le PBS, pas par le job Proxmox.

## Mise en place du job

Faite une fois, le 24/10/2026, sur un nœud Proxmox :

```bash
pveum pool add backup --comment "Sauvegarde quotidienne vers le PBS (ims-ops)"
pveum pool modify backup --vms 104,105,106
pvesh create /cluster/backup --schedule 02:00 --storage pbs-coolify --mode snapshot \
  --pool backup --enabled 1 --notes-template '{{guestname}}' \
  --notification-mode notification-system --comment "Pool backup vers PBS (ims-ops)"
```

Notifications, dans l'interface de Proxmox VE puis du PBS : cible webhook `ntfy` (`POST https://ntfy.ims-world.fr/ims-alerts`, en-tête `Authorization: Bearer {{ secrets.token }}`, secret `token` égal au jeton Ntfy de l'outil), et règle `echecs-ntfy` sur les sévérités `error` et `warning`. Sur le PBS, le datastore est en mode de notification **Notification System**.

## Sauvegarder à la main avant une intervention

<Steps>
  <Step title="Sauvegarder la machine dans le PBS">
    Dans Proxmox VE : sélectionner la machine, **Backup → Backup now**, stockage `pbs-coolify`. En ligne de commande, depuis le nœud :

    ```bash
    vzdump <vmid> --storage pbs-coolify --mode snapshot
    ```
  </Step>
  <Step title="Pour le NAS : sauvegarde locale">
    ```bash
    vzdump 100 --mode stop --compress zstd --storage local
    ```
    Les disques de données sont exclus (`backup=0`).
  </Step>
  <Step title="Pour Coolify : exporter aussi sa base et sa clé">
    Depuis la VM 104 :

    ```bash
    mkdir -p ~/sauvegarde-coolify && cd ~/sauvegarde-coolify
    docker exec coolify-db pg_dump -U coolify -d coolify -F c > coolify-$(date +%F).dump
    sudo cp /data/coolify/source/.env coolify-env-$(date +%F)
    ```
    Le fichier `.env` contient la clé d'application : sans elle, une base restaurée ne peut pas déchiffrer les secrets de Coolify.
  </Step>
  <Step title="Pour une base applicative précise">
    ```bash
    # PostgreSQL (Authentik, Immich, Forgejo, Zipline…)
    docker exec -t <conteneur-db> pg_dump -U <utilisateur> -d <base> -F c > <base>-$(date +%F).dump
    # MariaDB (PhotoPrism)
    docker exec -t <conteneur-db> mariadb-dump -u <utilisateur> -p <base> > <base>-$(date +%F).sql
    ```
  </Step>
</Steps>

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `wrong content type` en sauvegardant vers `local-lvm` | Un stockage LVM-Thin n'accepte pas les archives | Utiliser `local` (`/var/lib/vz`) |
| `namespace not found` | Le datastore ne répond plus (NFS périmé), ou un namespace inexistant est configuré sur `pbs-coolify` | Vérifier le montage sur le PBS, puis le champ **Namespace** du stockage (vide) |
| `ESTALE: Stale file handle` | Montage NFS du datastore périmé | `umount -l /mnt/pbs-datastore && mount /mnt/pbs-datastore`, puis `systemctl restart proxmox-backup-proxy proxmox-backup` |

## Voir aussi

- [Plan de reprise](/procedures/plan-reprise-activite-pra)
- [ims-pbs](/infrastructure/ims-pbs)
