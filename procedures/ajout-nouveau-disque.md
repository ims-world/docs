---
title: "Ajouter ou remplacer un disque"
description: "Ajouter un disque de capacité au NAS, ou remplacer le SSD de storage-hot, sans système de fichiers FUSE."
icon: "hard-drive"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : 1 à 3 heures (copie comprise) · Risque : moyen · Interruption de service : arrêt du MS-01 pour brancher le disque ; quelques minutes pour Immich et Forgejo en cas de remplacement du SSD

## Quand l'utiliser

| Cas | Option |
| --- | --- |
| Manque de place pour la médiathèque ou les archives | [A — Ajouter un disque de capacité](#option-a-ajouter-un-disque-de-capacité) |
| Premier SSD ou remplacement du SSD de `storage-hot` | [B — Ajouter ou remplacer le SSD](#option-b-ajouter-ou-remplacer-le-ssd-de-storage-hot) |

<Warning>
Le contrôleur SATA ASM1166 du MS-01 ne gère pas le branchement à chaud : toujours éteindre le nœud avant de brancher ou débrancher un disque.
</Warning>

<Warning>
Ne pas réintroduire MergerFS ni aucun système de fichiers FUSE sous un export NFS : c'est la cause de l'[incident du 23/10/2026](/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix). Chaque disque reste un système de fichiers ext4 indépendant.
</Warning>

## Prérequis

- Sauvegarde récente des invités (pool `backup`) et du système du NAS (`vzdump 100 --mode stop --storage local`).
- Accès SSH à `pve`, au NAS (`pct enter 100`) et à la VM 104.

## Étapes

### Identifier le disque

Depuis `pve`, après avoir branché le disque et redémarré :

```bash
lsblk
ls -la /dev/disk/by-id/ | grep -v part
```

Relever l'identifiant stable `ata-<modèle>_<numéro de série>`, jamais `/dev/sdX`, qui peut changer d'un démarrage à l'autre. Un disque sorti d'un boîtier USB peut avoir montré un WWN générique ; se fier à l'identifiant vu en SATA.

### Option A : ajouter un disque de capacité

Non encore pratiquée sur ce parc.

<Steps>
  <Step title="Formater depuis le nœud">
    ```bash
    sudo wipefs -a /dev/disk/by-id/<id>
    sudo mkfs.ext4 /dev/disk/by-id/<id>
    ```
    Le formatage précède la transmission au conteneur : `pct set … mp=` monte immédiatement le disque.
  </Step>
  <Step title="Transmettre le disque au NAS">
    ```bash
    pct set 100 --mp2 /dev/disk/by-id/<id>,mp=/mnt/disk2,backup=0
    ```
    `mp0` et `mp1` sont déjà pris. `backup=0` est obligatoire : sans lui, une sauvegarde du NAS copierait tout le disque.
  </Step>
  <Step title="Créer le point d'export">
    Dans le NAS, ajouter à `/etc/fstab` un bind-mount (`/mnt/disk2 /mnt/storage2 none bind 0 0`), puis à `/etc/exports` une ligne calquée sur celle de `/mnt/storage`, et appliquer :
    ```bash
    sudo systemctl daemon-reload && sudo mount -a
    sudo exportfs -r && exportfs -v
    ```
  </Step>
  <Step title="Monter le partage sur la VM 104">
    Ajouter à `/etc/fstab` : `10.10.10.1:/mnt/storage2 /mnt/nas-storage2 nfs defaults,nofail,_netdev 0 0`, puis `sudo mount -a`.
  </Step>
  <Step title="Déclarer le nouvel emplacement dans les applications">
    Monter `/mnt/nas-storage2` dans les conteneurs concernés, puis ajouter un dossier racine dans Radarr ou Sonarr et un dossier de bibliothèque dans Jellyfin. Un lien physique ne traverse pas deux disques : prévoir un dossier de téléchargement sur le nouveau disque et une catégorie qBittorrent qui y écrit.
  </Step>
</Steps>

### Option B : ajouter ou remplacer le SSD de storage-hot

Pratiquée le 18/08/2026 pour la bascule vers le SSD Samsung 870 EVO de 4 To.

<Steps>
  <Step title="Formater et transmettre">
    ```bash
    sudo wipefs -a /dev/disk/by-id/<id>
    sudo mkfs.ext4 /dev/disk/by-id/<id>
    pct set 100 --mp1 /dev/disk/by-id/<id>,mp=/mnt/ssd-hot-raw,backup=0
    ```
  </Step>
  <Step title="Copier à chaud (remplacement)">
    Dans le NAS, dans `tmux` :
    ```bash
    rsync -aH --info=progress2 /mnt/storage-hot/ /mnt/ssd-hot-raw/
    ```
  </Step>
  <Step title="Arrêter les services concernés">
    Arrêter Immich et Forgejo (et leurs bases) depuis Coolify.
  </Step>
  <Step title="Copier l'écart">
    Relancer la même commande `rsync`.
  </Step>
  <Step title="Retirer l'export NFS">
    ```bash
    exportfs -v | grep storage-hot
    sudo exportfs -u <réseau>:/mnt/storage-hot
    ```
    Sans cette étape, `umount` échoue (`target is busy`).
  </Step>
  <Step title="Basculer le montage">
    ```bash
    sudo umount /mnt/storage-hot
    sudo mount --bind /mnt/ssd-hot-raw /mnt/storage-hot
    sudo exportfs -r
    ```
  </Step>
  <Step title="Rendre la bascule permanente">
    Dans `/etc/fstab` du NAS, source de `/mnt/storage-hot` : `/mnt/ssd-hot-raw`. Puis :
    ```bash
    sudo systemctl daemon-reload
    systemctl status 'mnt-storage\x2dhot.mount'
    ```
    Sans `daemon-reload`, systemd garde l'ancienne version et un redémarrage reviendrait à l'ancienne source.
  </Step>
  <Step title="Remonter côté VM 104 et redémarrer les services">
    ```bash
    sudo umount -l /mnt/nas-hot && sudo mount /mnt/nas-hot
    ```
    Redémarrer Immich et Forgejo depuis Coolify.
  </Step>
</Steps>

## Vérification

```bash
df -h /mnt/storage-hot /mnt/storage2   # sur le NAS : bonne source, bonne taille
exportfs -v                             # exports présents
ls /mnt/nas-hot /mnt/nas-storage2       # sur la VM 104
```

<Check>
Les photos s'affichent dans Immich, les dépôts dans Forgejo, et les journaux ne montrent pas d'erreur d'accès.
</Check>

## Retour arrière

Tant que l'ancien support n'est pas effacé : remettre l'ancienne source dans `/etc/fstab` du NAS, `systemctl daemon-reload`, `mount -a`, `exportfs -r`, puis redémarrer les services. N'effacer l'ancien contenu qu'après un redémarrage du NAS validé.

## Dépannage

| Symptôme | Cause | Solution |
| --- | --- | --- |
| Disque non détecté | Branché à chaud | Éteindre le nœud, rebrancher, redémarrer |
| `target is busy` au démontage | Export NFS encore actif | `exportfs -u` avant `umount` |
| Retour à l'ancienne source après redémarrage | `daemon-reload` oublié | Corriger `/etc/fstab`, `systemctl daemon-reload` |
| Espace doublé après ajout d'un disque | Téléchargements et bibliothèque sur deux disques : copie au lieu de lien physique | Dossier de téléchargement sur le même disque que la bibliothèque |
