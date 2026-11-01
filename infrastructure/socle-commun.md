---
title: "Socle commun des hôtes"
description: "Configuration appliquée par ims-ops à tous les hôtes Linux gérés : comptes, SSH, fail2ban, paquets, mises à jour, supervision et tailnet."
icon: "layer-group"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

## Rôle

Les hôtes Linux du homelab partagent une configuration commune, décrite dans l'inventaire du dépôt ims-ops et appliquée par ses rôles Ansible. Cette page la résume une fois pour toutes : les fiches d'hôtes ne décrivent que leurs écarts.

La valeur de référence reste l'inventaire (`inventory/group_vars/`, `inventory/host_vars/`). Une divergence entre l'hôte et l'inventaire se détecte avec `just audit <hôte>`.

## Hôtes concernés

| Hôte | Nature | Particularités |
| --- | --- | --- |
| `pve`, `pve-macmini` | Nœuds Proxmox | Root par clé, mises à jour manuelles, collecteur SMART |
| `ims-nas`, `ims-pbs`, `ims-coolify-worker` | Conteneurs LXC | Capteurs matériels absents (collecteur `hwmon` désactivé) |
| `ims-coolify` | VM | Root par clé (Coolify), agent QEMU |
| `lab-elo` | VM d'un tiers | Pas de compte Termix |

Hors gestion : Home Assistant OS (VM 106, appliance), Windows (VM 102), Raspberry Pi (éteint).

## Comptes

| Compte | Hôtes | Accès |
| --- | --- | --- |
| `cmolotkoff` | Tous | Clé Ed25519 seule, sudo sans mot de passe, chaque commande journalisée (`log_subcmds`) |
| `termix-usr` | Tous sauf `lab-elo` | Compte de service de [Termix](/services/termix). Clé acceptée seulement depuis la VM 104 (`192.168.1.52`), et depuis le réseau Docker `10.0.1.0/24` sur `ims-coolify`. Sudo journalisé, groupe `adm` |
| `root` | Nœuds Proxmox, `ims-coolify`, worker | Par clé seulement : cluster Proxmox, Coolify vers ses hôtes |

Un compte retiré de la liste des administrateurs est désactivé, jamais supprimé, pour que ses fichiers gardent leur propriétaire.

## SSH

Fichier `/etc/ssh/sshd_config.d/00-ims-ops.conf`, lu avant toute autre configuration :

| Réglage | Valeur |
| --- | --- |
| Authentification par mot de passe | Désactivée |
| Redirection X11 | Désactivée |
| Connexion root | `prohibit-password` par défaut ; `no` sur les VM, sauf `ims-coolify` |
| Port | 22 ([ADR-010](/history/adr/adr-010-maintien-port-ssh-22-et-hostname-cluster)) |

Aucun port SSH d'hôte n'est ouvert sur Internet. L'accès se fait depuis le LAN ou le tailnet.

## fail2ban

| Réglage | Valeur |
| --- | --- |
| Prisons | `sshd`, `recidive` |
| Bannissement | 1 heure, doublé à chaque récidive, plafond d'une semaine |
| Seuil | 5 échecs en 10 minutes |
| Alertes | Topic Ntfy `ims-alerts`, jeton dédié à fail2ban ; Ntfy est joint par `192.168.1.52` |

```bash
sudo fail2ban-client status sshd
sudo fail2ban-client set sshd unbanip <ip>
```

## Système

| Domaine | Configuration |
| --- | --- |
| Fuseau horaire | `Europe/Paris` |
| Journal systemd | Persistant (`/var/log/journal`) |
| Paquets | `curl`, `git`, `htop`, `jq`, `ncdu`, `rsync`, `bind9-dnsutils`, `bash-completion`, `acl`, `tmux` ; `qemu-guest-agent` sur les VM |
| Mises à jour | Correctifs de sécurité automatiques, sans redémarrage automatique. Nœuds Proxmox mis à jour à la main, nœud par nœud |

## Supervision

Chaque hôte porte un agent Grafana Alloy qui envoie métriques et journaux à Prometheus et Loki sur la VM 104.

| Réglage | Valeur |
| --- | --- |
| Destination | `10.10.10.2` pour les hôtes du réseau interne `vmbr1` (`ims-nas`, `ims-pbs`, `ims-coolify`), `192.168.1.52` pour les autres |
| Étiquette `host` | Nom d'inventaire, sauf `pve` (`ms01-pve`, conservé pour l'historique des tableaux de bord) |
| Collecteurs propres | Docker sur `ims-coolify` et le worker ; Traefik sur `ims-coolify` ; SMART (fichier texte) sur les nœuds |

Voir [Supervision](/services/monitoring).

## Tailnet

Les hôtes rejoignent le tailnet Headscale sous l'utilisateur `infra`, avec la clé de pré-authentification conservée dans le coffre d'ims-ops. Le DNS du tailnet n'est pas accepté (`--accept-dns=false`). Voir [Ajouter une machine au tailnet](/procedures/ajout-machine-headscale).

Les conteneurs LXC ont besoin de l'accès à `/dev/net/tun`, déclaré dans leur configuration Proxmox :

```ini
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

## Voir aussi

- [Administration avec ims-ops](/procedures/administration-ims-ops)
- [Matrice d'exposition](/reseau/matrice-securite-exposition)
