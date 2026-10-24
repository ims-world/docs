---
title: "MS-01 (pve)"
description: "Nœud principal du cluster Proxmox ims-cluster : Minisforum MS-01, hôte du NAS, du PBS et de la VM Coolify."
icon: "server"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

import { ips, hardware } from "/snippets/variables.mdx";

## Fiche

| Propriété | Valeur |
| --- | --- |
| Type | Machine physique, Minisforum MS-01 |
| Nœud et VMID | Nœud `pve` du cluster `ims-cluster` (2 votes sur 2) |
| Noms | Proxmox `pve` · système `pve` · inventaire `pve` · tailnet `ims-pve-ms01` |
| OS | Proxmox VE 9.2.11 |
| CPU et RAM | {hardware.ms01Cpu}, iGPU Iris Xe · {hardware.ms01Ram} |
| Disques | {hardware.ms01Storage} (LVM-Thin `local-lvm`) ; HDD 3 To et SSD 4 To SATA sur {hardware.sataCard}, confiés au NAS |
| IP LAN | <code>{ips.pveLan}</code> |
| IP tailnet | <code>{ips.ms01}</code> |
| Sauvegarde | Non sauvegardé : se réinstalle (voir [plan de reprise](/procedures/plan-reprise-activite-pra)) |
| Statut | En service |

Interface web : <code>https://{ips.pveLan}:8006</code> (LAN) ou <code>https://{ips.ms01}:8006</code> (tailnet).

## Ce qui tourne dessus

| VMID | Nom | Type | Ressources | Rôle |
| --- | --- | --- | --- | --- |
| 100 | `ims-nas` | LXC privilégié | 2 cœurs, 1 Go | [Stockage NFS et SMB](/infrastructure/ims-nas) |
| 102 | Windows | VM | — | Poste Windows, démarré à la demande |
| 103 | `ims-pbs` | LXC privilégié | 2 cœurs, 1 Go | [Proxmox Backup Server](/infrastructure/ims-pbs) |
| 104 | `ims-coolify` | VM | 6 cœurs, 18 Go, 128 Go | [Coolify et services](/infrastructure/vm-coolify) |
| 107 | `lab-elo` | VM | — | VM d'un tiers, souvent éteinte |
| 9001 | `ubuntu-2404-ims-ops` | Template | — | Base des VM créées par `just vm` |

## Ordre de démarrage

Le NAS démarre en premier, car le PBS et la VM 104 montent ses partages NFS.

| Ordre | Invité | Délai avant le suivant |
| --- | --- | --- |
| 1 | LXC 100 `ims-nas` | 15 s |
| 2 | LXC 103 `ims-pbs` | 10 s |
| 3 | VM 104 `ims-coolify` | 20 s |

```bash
pct set 100 -onboot 1 -startup order=1,up=15
pct set 103 -onboot 1 -startup order=2,up=10
qm set 104 --onboot 1 --startup order=3,up=20
```

## Template de VM et création de VM

Les VM Linux sont créées depuis le template `ubuntu-2404-ims-ops` (VMID `9001`), construit et utilisé par ims-ops.

| Élément | Valeur |
| --- | --- |
| Construction | `just template`, depuis l'image cloud officielle Ubuntu 24.04, somme SHA256 vérifiée |
| Processeur | `x86-64-v2-AES`, pour rester migrable vers le second nœud |
| Disque | `virtio-scsi-single` avec `iothread`, sur `local-lvm` |
| Cloud-Init | Minimal : nom, compte `cmolotkoff`, clé SSH. Le reste est appliqué par Ansible |
| Création | `just vm <nom> [os] [taille] [mode]` : VMID et IP choisis par la commande, dans la plage `192.168.1.200-249` |
| Destruction | `just vm-destroy <nom>`, limitée aux VM créées par `just vm` |

L'enrôlement dans le tailnet passe par le rôle Ansible `tailscale`, jamais par Cloud-Init.

<Warning>
L'ancien template (VMID 9000) et son snippet `vendor-data.yaml` ont été supprimés le 24/10/2026 : le snippet contenait une clé Headscale en clair. Ne pas recréer ce mécanisme.
</Warning>

## Passthrough de l'iGPU

L'iGPU Iris Xe est attribuée à la VM 104 (`hostpci0`, chipset `q35`) pour le transcodage matériel de Jellyfin. Le détail est dans l'[ADR-008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe).

Conséquence : le nœud n'a plus de sortie vidéo. En cas de besoin d'une console physique, retirer temporairement le périphérique (`qm set 104 --delete hostpci0`) et redémarrer le nœud.

## Disques du NAS : veille et SMART

Les outils qui parlent directement au disque (`hd-idle`, `smartd`) tournent sur le nœud, jamais dans le LXC du NAS : le montage transmis au conteneur ne donne pas accès au périphérique bloc.

| Outil | Réglage |
| --- | --- |
| `hd-idle` | Mise en veille du HDD après 30 minutes d'inactivité (`/etc/default/hd-idle`, option `-i 1800`) |
| `smartd` et `smartmon.sh` | Santé SMART, relevée toutes les 5 minutes pour Alloy |

```bash
systemctl status hd-idle smartd --no-pager
hdparm -C /dev/disk/by-id/ata-APPLE_HDD_ST3000DM001_Z1F3N0NZ
smartctl -H /dev/disk/by-id/ata-APPLE_HDD_ST3000DM001_Z1F3N0NZ
```

## Particularités

- Écarts au [socle commun](/infrastructure/socle-commun) : root par clé (cluster), mises à jour manuelles, étiquette Alloy `ms01-pve`.
- Dépôts APT au format `deb822` : `pve-enterprise.sources` désactivé, `proxmox.sources` sur `pve-no-subscription`.
- Le nom d'hôte `pve` est conservé : renommer un nœud de cluster casse Corosync ([ADR-010](/history/adr/adr-010-maintien-port-ssh-22-et-hostname-cluster)).
- Le pare-feu Proxmox n'est pas activé. Le chantier est suivi dans la [feuille de route](/roadmap).

## Sauvegarde et reprise

Le nœud lui-même n'est pas sauvegardé : il se réinstalle, puis les invités sont restaurés depuis le PBS. Voir le [plan de reprise](/procedures/plan-reprise-activite-pra).

## Supervision

Agent Alloy (étiquette `ms01-pve`) avec le collecteur SMART. Tableaux de bord dans [Supervision](/services/monitoring).

## Voir aussi

- [Mac Mini (pve-macmini)](/infrastructure/mac-mini)
- [Socle commun des hôtes](/infrastructure/socle-commun)
- [Politique de sauvegarde](/procedures/politique-sauvegardes)
