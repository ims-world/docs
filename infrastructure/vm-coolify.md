---
title: "ims-coolify (VM 104)"
description: "VM principale des services : Coolify, Traefik, CrowdSec, Authentik, supervision et la plupart des applications Docker."
icon: "box"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v4.3.14"
---

import { ips } from "/snippets/variables.mdx";

## Fiche

| Propriété | Valeur |
| --- | --- |
| Type | VM QEMU/KVM, chipset `q35`, processeur `host` ([ADR-005](/history/adr/adr-005-cpu-vm-coolify-mode-host)) |
| Nœud et VMID | `pve`, 104 |
| Noms | Proxmox `ims-coolify` · inventaire `ims-coolify` · tailnet `ims-pve-104-coolify` |
| OS | Ubuntu 24.04 LTS |
| CPU et RAM | 6 cœurs · 18 Go |
| Disques | 128 Go sur `local-lvm` ; partages NFS du NAS |
| IP LAN | `192.168.1.52` ; réseau interne `vmbr1` : `10.10.10.2` |
| IP tailnet | <code>{ips.coolify}</code> |
| Sauvegarde | Pool `backup`, chaque nuit vers le PBS |
| Statut | En service |

## Ce qui tourne dessus

- Coolify v4.3.14, maître : il orchestre cette VM et le [worker 105](/infrastructure/lxc-coolify-worker).
- [Traefik](/reseau/traefik-proxy) (conteneur `coolify-proxy`), seul point d'entrée HTTP et HTTPS du homelab, et [CrowdSec](/reseau/crowdsec).
- [Headscale et Headplane](/reseau/headscale-headplane).
- Les services de la navigation, sauf Home Assistant : identité et accès, supervision, médias, outils.

L'UUID Coolify et le chemin de chaque service figurent dans sa fiche.

## Stockage

```text
# /etc/fstab
10.10.10.1:/mnt/storage      /mnt/nas-storage  nfs  defaults,nofail,_netdev  0 0
10.10.10.1:/mnt/storage-hot  /mnt/nas-hot      nfs  defaults,nofail,_netdev  0 0
```

Les bases de données (PostgreSQL, MariaDB) restent sur le disque local de la VM, dans des volumes Docker nommés ; seuls les fichiers volumineux sont sur le NAS.

## Réseau Docker et IP source

Deux réglages permettent à Traefik de voir la vraie IP des clients du tailnet, condition du filtrage `vpn-only` ([ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only)) :

```json
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "default-address-pools": [{"base":"10.0.0.0/8","size":24}],
  "userland-proxy": false
}
```

```bash
sudo tailscale set --snat-subnet-routes=false
```

## GPU

L'iGPU du MS-01 est attribuée à la VM ([ADR-008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe)). Le pilote `i915` vient du métapaquet `linux-modules-extra-generic`, qui doit rester installé : sans lui, une mise à jour du noyau fait disparaître `/dev/dri` au redémarrage suivant ([incident du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri)).

## Particularités

- Écarts au [socle commun](/infrastructure/socle-commun) : root par clé (Coolify gère son hôte en SSH), envoi Alloy par `10.10.10.2`, collecteurs Docker et Traefik, `termix-usr` accepté aussi depuis `10.0.1.0/24`.
- Pas de snapshot Proxmox possible (périphérique PCI transmis) : sauvegarder les fichiers touchés avant une modification manuelle.
- Les domaines restreints au tailnet ne sont pas déclarés dans l'interface Coolify mais dans `/data/coolify/proxy/dynamic/vpn-only.yaml` (voir [Traefik](/reseau/traefik-proxy)).
- Accès aux fichiers de Coolify sans `sudo`, par ACL :

  ```bash
  sudo setfacl -m u:cmolotkoff:--x /data/coolify
  sudo setfacl -R -m u:cmolotkoff:rwX /data/coolify/services /data/coolify/proxy
  sudo setfacl -R -d -m u:cmolotkoff:rwX /data/coolify/services /data/coolify/proxy
  ```

  Le propriétaire n'est pas modifié : Coolify vérifie ses propres permissions (utilisateur `9999`).
- `cmolotkoff` est dans le groupe `docker`, ce qui équivaut à un accès root.

## Sauvegarde et reprise

Coolify ne fait aucune sauvegarde applicative. La VM entière, volumes Docker compris, est sauvegardée chaque nuit par le PBS. Les données sur le NAS (médiathèque, photos) ne sont pas dans cette sauvegarde. Voir la [politique de sauvegarde](/procedures/politique-sauvegardes).

## Supervision

Agent Alloy avec journaux Docker et métriques Traefik. Sondes Uptime Kuma sur les services exposés.

## Voir aussi

- [Worker Coolify (LXC 105)](/infrastructure/lxc-coolify-worker)
- [Déployer un service](/procedures/deploiement-service)
