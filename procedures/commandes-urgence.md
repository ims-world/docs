---
title: "Accès de secours et commandes d'urgence"
description: "Reprendre la main quand le tailnet, Termix ou la VM 104 sont indisponibles, et commandes de diagnostic à copier."
icon: "life-ring"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : quelques minutes · Risque : faible · Interruption de service : aucune

## Quand l'utiliser

Quand l'accès habituel ne fonctionne plus. Termix et Authentik tournent sur la VM 104, Headscale aussi : si elle est arrêtée, ces trois accès tombent ensemble.

| Panne | Accès de remplacement |
| --- | --- |
| Termix indisponible | SSH direct depuis le poste, par l'IP tailnet ou LAN des hôtes |
| Tailnet indisponible (Headscale, VM 104) | Depuis le LAN : interfaces Proxmox, SSH par IP LAN, ims-ops avec `IMS_NETWORK=lan` |
| Authentik indisponible | Comptes locaux : Proxmox (PAM), compte local de secours de Termix (TOTP, dans Vaultwarden) |
| Clé SSH du poste perdue | Console Proxmox (interface web, compte `cmolotkoff` ou `root`), puis rotation de la clé avec ims-ops |
| Interface Proxmox injoignable | SSH sur le nœud ; le MS-01 n'a pas de sortie vidéo (iGPU transmise à la VM 104) |

## Prérequis

- Être sur le LAN, physiquement ou par un appareil déjà dans le tailnet.
- Les mots de passe de secours dans Vaultwarden (Proxmox, Termix). Si Vaultwarden est lui-même sur la VM 104 arrêtée, l'application Bitwarden garde un coffre local consultable hors connexion.

## Étapes

<Steps>
  <Step title="Reprendre la main depuis le LAN">
    ```bash
    ssh cmolotkoff@192.168.1.41     # pve
    ssh cmolotkoff@192.168.1.42     # pve-macmini
    # Interface Proxmox : https://192.168.1.41:8006
    ```
    Depuis le poste d'administration, ims-ops fonctionne aussi par le LAN : `IMS_NETWORK=lan just ping`.
  </Step>
  <Step title="Remettre la VM 104 en service">
    ```bash
    qm status 104
    qm start 104          # ou qm reboot 104
    ```
    Si elle ne démarre pas, voir le [plan de reprise](/procedures/plan-reprise-activite-pra#restaurer-une-machine-depuis-le-pbs).
  </Step>
  <Step title="Récupérer l'accès à Authentik si besoin">
    ```bash
    # Depuis la VM 104 : lien de connexion pour akadmin, valable 1 heure
    docker exec -it authentik-worker-k5mxvc2r6c4zlb6j3d443h7b ak create_recovery_key 1 akadmin
    ```
  </Step>
</Steps>

## Commandes de diagnostic

<CodeGroup>
```bash Proxmox
# État de toutes les VM et de tous les conteneurs du cluster
pvesh get /cluster/resources --type vm
# Invités du nœud courant
pct list
qm list
# Dernières tâches du nœud (sauvegardes, démarrages)
pvesh get /nodes/$(hostname)/tasks --limit 10
# Mémoire, disque système et stockages
free -h
df -h /
pvesm status
# Quorum du cluster
pvecm status
```

```bash Docker (VM 104)
docker ps --format "table {{.Names}}\t{{.Status}}"
docker logs --tail 100 -f <conteneur>
docker restart coolify-proxy
# Vider les journaux Docker si le disque est plein
# (rarement utile : chaque journal est limité à 3 fichiers de 10 Mo)
sudo sh -c 'truncate -s 0 /var/lib/docker/containers/*/*-json.log'
```

```bash Tailnet
tailscale status
tailscale ping 100.64.0.4
# Depuis la VM 104
docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale nodes list
```

```bash Stockage
# Sur la VM 104 ou le PBS : remonter un partage NFS bloqué
sudo umount -l /mnt/nas-storage && sudo mount -a
# Santé du HDD, depuis pve
smartctl -H /dev/disk/by-id/ata-APPLE_HDD_ST3000DM001_Z1F3N0NZ
```

```bash Exposition
# Hors tailnet : la requête doit être refusée (403)
curl -sI https://qbit.ims-world.fr | head -1
```
</CodeGroup>

## Vérification

`just ping` passe sur tout le parc, Uptime Kuma est au vert, `tailscale status` liste les machines attendues.

## Retour arrière

Sans objet : ces opérations ne modifient pas la configuration.

## Dépannage

Voir [Dépannage courant](/procedures/depannage-courant).
