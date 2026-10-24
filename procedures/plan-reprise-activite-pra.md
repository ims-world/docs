---
title: "Plan de reprise"
description: "Restaurer une machine depuis le PBS, reconstruire un nœud, faire face à la perte d'un disque ou du tailnet, et tester les sauvegardes."
icon: "rotate-left"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

import { ips } from "/snippets/variables.mdx";

Durée : de 15 minutes (une machine) à une demi-journée (un nœud) · Risque : élevé · Interruption de service : selon le scénario

<Warning>
Ce plan n'a pas encore été validé par une restauration réelle. Le premier test est décrit en fin de page ; sa date sera inscrite ici une fois réalisé.
</Warning>

## Quand l'utiliser

| Scénario | Section |
| --- | --- |
| Une VM ou un conteneur du pool `backup` est cassé ou supprimé | [Restaurer une machine](#restaurer-une-machine-depuis-le-pbs) |
| Le MS-01 ou son disque système est perdu | [Reconstruire le nœud pve](#reconstruire-le-nœud-pve) |
| Le Mac Mini est perdu | [Reconstruire le nœud pve-macmini](#reconstruire-le-nœud-pve-macmini) |
| Le HDD ou le SSD du NAS est perdu | [Perte d'un disque du NAS](#perte-dun-disque-du-nas) |
| Headscale ou le tailnet est indisponible | [Perte du tailnet](#perte-du-tailnet) |

## Prérequis

- Accès au LAN (le tailnet dépend de la VM 104).
- Une clé USB d'installation Proxmox VE 9.
- Le poste d'administration avec le dépôt ims-ops et le mot de passe du coffre (trousseau macOS).
- L'empreinte du certificat du PBS, visible sur son tableau de bord (**Show Fingerprint**), si le stockage `pbs-coolify` doit être recréé.

## Étapes

### Restaurer une machine depuis le PBS

<Steps>
  <Step title="Choisir la sauvegarde">
    Dans Proxmox VE : **Datacenter → Storage → pbs-coolify → Backups**, filtrer sur le VMID. En ligne de commande :

    ```bash
    pvesm list pbs-coolify --vmid <vmid>
    ```
  </Step>
  <Step title="Restaurer">
    Depuis l'interface : sélectionner la sauvegarde, **Restore**, stockage `local-lvm`. En ligne de commande :

    ```bash
    # VM
    qmrestore pbs-coolify:backup/vm/<vmid>/<horodatage> <vmid> --storage local-lvm --force
    # Conteneur
    pct restore <vmid> pbs-coolify:backup/ct/<vmid>/<horodatage> --storage local-lvm --force
    ```
    `--force` écrase la machine existante de même VMID.
  </Step>
  <Step title="Démarrer et vérifier">
    ```bash
    qm start <vmid>     # ou pct start <vmid>
    ```
  </Step>
</Steps>

### Reconstruire le nœud pve

Les disques du NAS (HDD et SSD) ne sont pas touchés : les sauvegardes et les données y sont toujours. Les configurations des invités de `pve` restent aussi lisibles depuis `pve-macmini`, dans le système de fichiers du cluster (`/etc/pve/nodes/pve/`).

<Steps>
  <Step title="Mettre de côté les configurations des invités">
    Sur `pve-macmini` :
    ```bash
    mkdir -p /root/pve-configs
    cp -r /etc/pve/nodes/pve/qemu-server /etc/pve/nodes/pve/lxc /root/pve-configs/
    ```
  </Step>
  <Step title="Retirer l'ancien nœud du cluster">
    Sur `pve-macmini`, qui a perdu le quorum :
    ```bash
    pvecm expected 1
    pvecm delnode pve
    ```
  </Step>
  <Step title="Installer Proxmox VE">
    Nom d'hôte `pve` (ne pas en changer, [ADR-010](/history/adr/adr-010-maintien-port-ssh-22-et-hostname-cluster)), IP <code>{ips.pveLan}/24</code>, passerelle `192.168.1.254`. Recréer le pont `vmbr1` (`10.10.10.0/24`, sans port physique ni passerelle). Désactiver le dépôt entreprise et activer `pve-no-subscription`.
  </Step>
  <Step title="Rejoindre le cluster">
    Sur le nouveau `pve` :
    ```bash
    pvecm add 192.168.1.42
    ```
  </Step>
  <Step title="Recréer le NAS (LXC 100)">
    Recréer le conteneur à partir de sa configuration mise de côté : conteneur privilégié, disques transmis avec `backup=0`. Puis, dans le conteneur : bind-mounts de `/etc/fstab`, exports NFS, partage Samba avec le compte `cmolotkoff` (UID 1000, à recréer avec cet UID exact). Voir [ims-nas](/infrastructure/ims-nas). Appliquer ensuite le socle : `just setup ims-nas`.
  </Step>
  <Step title="Réinstaller le PBS (LXC 103) et rattacher le datastore">
    Recréer le conteneur (privilégié, `mount=nfs`, accès `/dev/net/tun`), installer `proxmox-backup-server`, monter `10.10.10.1:/mnt/storage/backups` sur `/mnt/pbs-datastore` (voir [ims-pbs](/infrastructure/ims-pbs)). Dans l'interface du PBS, ajouter le datastore `pve-backups` sur ce chemin en cochant la réutilisation d'un datastore existant. Les sauvegardes réapparaissent.
  </Step>
  <Step title="Recréer le stockage pbs-coolify">
    S'il n'est plus défini dans le cluster :
    ```bash
    pvesm add pbs pbs-coolify --server 192.168.1.51 --datastore pve-backups \
      --username <utilisateur>@pbs --password --fingerprint <empreinte>
    ```
  </Step>
  <Step title="Restaurer la VM 104 et les autres invités">
    Suivre [Restaurer une machine](#restaurer-une-machine-depuis-le-pbs) pour la VM 104, puis pour les autres invités sauvegardés. Si le matériel a changé, vérifier l'adresse PCI de l'iGPU avant de démarrer la VM 104 (`hostpci0`, [ADR-008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe)).
  </Step>
  <Step title="Rétablir l'ordre de démarrage et le socle">
    Ordre de démarrage : voir [MS-01](/infrastructure/proxmox-host#ordre-de-démarrage). Puis, depuis le poste : `just setup pve`, `just audit`.
  </Step>
</Steps>

### Reconstruire le nœud pve-macmini

Même démarche, en inversant les rôles : retirer `pve-macmini` du cluster depuis `pve`, réinstaller Proxmox VE avec le même nom et la même IP (`192.168.1.42`), rejoindre le cluster, puis restaurer les invités 105 et 106 depuis le PBS. En attendant, ces invités peuvent être restaurés temporairement sur `pve`.

### Perte d'un disque du NAS

| Disque perdu | Effet | Reprise |
| --- | --- | --- |
| HDD 3 To | Médiathèque, originaux PhotoPrism et **toutes les sauvegardes PBS** perdus. Les invités continuent de tourner | Remplacer le disque, recréer `/mnt/storage/backups`, créer un nouveau datastore, lancer immédiatement une sauvegarde du pool. Retélécharger la médiathèque |
| SSD 4 To | Photos Immich et dépôts Forgejo perdus, sans copie connue | Remplacer le disque, recréer `/mnt/storage-hot`. Les dépôts miroirs de GitHub se resynchronisent ; le reste est perdu |

Les deux procédures d'ajout et de remplacement de disque sont dans [Ajouter un disque](/procedures/ajout-nouveau-disque).

### Perte du tailnet

Headscale tourne sur la VM 104 : si elle est arrêtée, le tailnet ne coordonne plus de nouvelles connexions.

1. Passer par le LAN : interfaces Proxmox (`https://192.168.1.41:8006`), SSH par IP LAN, ims-ops avec `IMS_NETWORK=lan`.
2. Restaurer ou redémarrer la VM 104. L'état de Headscale (base, clé `noise_private.key`) est dans sa sauvegarde.

### Clés à vérifier après une restauration de la VM 104

| Composant | Élément | Risque s'il est perdu |
| --- | --- | --- |
| Headscale | `noise_private.key` dans le dossier du service | Toutes les machines du tailnet sont à réenrôler |
| Authentik | `SECRET_KEY` | Sessions et jetons invalidés |
| Vaultwarden | `data/rsa_key.pem` | Pièces jointes du coffre illisibles |
| Coolify | `/data/coolify/source/.env` (clé d'application) | Secrets de Coolify illisibles |

## Vérification

- Les services publics répondent et Uptime Kuma est au vert.
- `tailscale status` liste les machines attendues.
- `just ping` et `just audit` passent sur tout le parc.
- Une sauvegarde manuelle du pool réussit (`vzdump <vmid> --storage pbs-coolify`).

## Retour arrière

Une restauration avec `--force` écrase la machine existante. En cas de doute, restaurer d'abord sur un VMID libre (voir le test ci-dessous), comparer, puis remplacer.

## Tester les sauvegardes

À faire au moins une fois par an, et après tout changement de la chaîne de sauvegarde.

<Steps>
  <Step title="Restaurer la VM 104 sur un VMID libre">
    ```bash
    qmrestore pbs-coolify:backup/vm/104/<horodatage> 900 --storage local-lvm --unique 1
    ```
  </Step>
  <Step title="Isoler la copie avant de la démarrer">
    La copie a la même IP et la même identité tailnet que la production, et réclame l'iGPU déjà attribuée à la VM 104 :
    ```bash
    qm set 900 --delete hostpci0
    qm set 900 --net0 "$(qm config 900 | awk -F': ' '/^net0/ {print $2}'),link_down=1"
    qm set 900 --onboot 0
    qm start 900
    ```
  </Step>
  <Step title="Contrôler depuis la console">
    Ouvrir la console de la VM 900 dans Proxmox, vérifier que le système démarre, que Docker liste les conteneurs (`docker ps -a`) et que les clés du tableau ci-dessus sont présentes.
  </Step>
  <Step title="Supprimer la copie et dater le test">
    ```bash
    qm stop 900 && qm destroy 900 --purge
    ```
    Inscrire la date et le résultat en tête de cette page.
  </Step>
</Steps>
