---
title: "ims-coolify (VM 104)"
description: "VM principale des services : Coolify, Traefik, CrowdSec, Authentik, supervision et la plupart des applications Docker."
icon: "box"
iconType: "duotone"
last_reviewed: "2026-11-08"
app_version: "v4.3.14"
---

import { ips } from "/snippets/variables.mdx";

## Fiche

La VM 104 porte presque tous les services du homelab, ainsi que Traefik, seul point d'entrée web.

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

L'orchestrateur, la couche réseau et les services.

<CardGroup cols={2}>
  <Card color="#3B82F6" title="Coolify v4.3.14" icon="cubes">
    Maître : il orchestre cette VM et le [worker 105](/infrastructure/lxc-coolify-worker).
  </Card>
  <Card color="#FF6B00" title="Traefik et CrowdSec" icon="route" href="/reseau/traefik-proxy">
    Conteneur `coolify-proxy`, seul point d'entrée HTTP et HTTPS du homelab, protégé par CrowdSec.
  </Card>
  <Card color="#FF6B00" title="Headscale et Headplane" icon="network-wired" href="/reseau/headscale-headplane">
    Coordination du tailnet.
  </Card>
  <Card color="#3B82F6" title="Services" icon="grip">
    Tous ceux de la navigation, sauf Home Assistant : identité et accès, supervision, médias, outils.
  </Card>
</CardGroup>

L'UUID Coolify et le chemin de chaque service figurent dans sa fiche.

## Stockage

Les partages du NAS passent par le réseau interne `vmbr1`.

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

L'iGPU du MS-01 est attribuée à la VM ([ADR-008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe)), pour le transcodage de Jellyfin et PhotoPrism.

<Warning>
Le pilote `i915` vient du métapaquet `linux-modules-extra-generic`, qui doit rester installé : sans lui, une mise à jour du noyau fait disparaître `/dev/dri` au redémarrage suivant ([incident du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri)).
</Warning>

## Particularités

Ce qui distingue cette VM des autres hôtes du parc.

- Écarts au [socle commun](/infrastructure/socle-commun) : root par clé (Coolify gère son hôte en SSH), envoi Alloy par `10.10.10.2`, collecteurs Docker et Traefik, `termix-usr` accepté aussi depuis `10.0.1.0/24`.
- Les domaines restreints au tailnet ne sont pas déclarés dans l'interface Coolify mais dans `/data/coolify/proxy/dynamic/vpn-only.yaml` (voir [Traefik](/reseau/traefik-proxy)).
- Accès aux fichiers de Coolify sans `sudo`, par ACL :

  ```bash
  sudo setfacl -m u:cmolotkoff:--x /data/coolify
  sudo setfacl -R -m u:cmolotkoff:rwX /data/coolify/services /data/coolify/proxy
  sudo setfacl -R -d -m u:cmolotkoff:rwX /data/coolify/services /data/coolify/proxy
  ```

  Le propriétaire n'est pas modifié : Coolify vérifie ses propres permissions (utilisateur `9999`).
- `cmolotkoff` est dans le groupe `docker`, ce qui équivaut à un accès root.
- Tailscale sans le DNS du tailnet, comme le reste du parc (corrigé le 10/11/2026, [incident](/history/incidents/2026-11-10-dns-worker-105)) : les conteneurs résolvent les domaines publics par le DNS du LAN.
- Métriques et analyses intégrées de Coolify (Sentinel) désactivées : la supervision passe par [Grafana](/services/monitoring). Sentinel réécrivait sa base en continu.
- Dossier de données d'Alloy en mémoire (tmpfs, `alloy_data_tmpfs` dans ims-ops) : son tampon de métriques n'écrit plus sur le disque. Il est vidé au redémarrage de la VM.

## Sauvegarde et reprise

<Note>
Pas de snapshot Proxmox possible (périphérique PCI transmis) : sauvegarder les fichiers touchés avant une modification manuelle.
</Note>

Coolify ne fait aucune sauvegarde applicative. La VM entière, volumes Docker compris, est sauvegardée chaque nuit par le PBS. Les données sur le NAS (médiathèque, photos) ne sont pas dans cette sauvegarde. Voir la [politique de sauvegarde](/procedures/politique-sauvegardes).

La sauvegarde est incrémentale : seuls les blocs écrits depuis la nuit précédente sont relus et envoyés. Sa durée dépend donc du volume écrit sur le disque dans la journée. Mesure du 08/11/2026 : environ 108 Go écrits par jour, 46 Gio de blocs modifiés, 23 minutes de sauvegarde. Principales sources :

| Source | Écritures par jour | Traitement |
| --- | --- | --- |
| Redis de Coolify (`coolify-redis`) | ~28 Go : sauvegarde complète toutes les 20 s (`--save 20 1`) | Conservé : réglage de l'installation de Coolify |
| Alloy (tampon des métriques) | ~16 Go | Dossier de données en mémoire |
| Sentinel (métriques de Coolify) | ~14 Go | Désactivé |

Écritures par conteneur depuis son démarrage :

```bash
for c in $(docker ps -q); do
  id=$(docker inspect -f '{{.Id}}' $c); name=$(docker inspect -f '{{.Name}}' $c | tr -d /)
  w=$(awk '{for(i=1;i<=NF;i++) if($i ~ /^wbytes=/){split($i,a,"="); s+=a[2]}} END{print s+0}' /sys/fs/cgroup/system.slice/docker-$id.scope/io.stat 2>/dev/null)
  echo "$w $name"
done | sort -n | tail -10 | numfmt --to=iec --field=1
```

## Supervision

Agent Alloy avec journaux Docker et métriques Traefik. Sondes Uptime Kuma sur les services exposés. Battement de cœur vers Healthchecks.io toutes les 5 minutes ([ADR-015](/history/adr/adr-015-alerte-externe-battement-de-coeur)).

## Voir aussi

<CardGroup cols={2}>
  <Card color="#3B82F6" title="Worker Coolify (LXC 105)" icon="server" href="/infrastructure/lxc-coolify-worker">Nœud d'exécution secondaire.</Card>
  <Card color="#64748B" title="Déployer un service" icon="list-check" href="/procedures/deploiement-service">Étapes d'un nouveau service.</Card>
</CardGroup>
