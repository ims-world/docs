---
title: "Glossaire"
description: "Termes, sigles et noms propres employés dans cette documentation, avec leur sens dans le homelab."
icon: "book-bookmark"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

## Virtualisation

| Terme | Définition |
| --- | --- |
| Cluster `ims-cluster` | Les deux nœuds Proxmox (`pve`, `pve-macmini`), qui partagent leur configuration (`/etc/pve`) et se votent mutuellement le quorum |
| Conteneur LXC | Système Linux isolé qui partage le noyau du nœud. Plus léger qu'une VM. Utilisé pour le NAS, le PBS et le worker Coolify |
| LXC privilégié | Conteneur dont root correspond au root du nœud. Nécessaire pour monter du NFS ou servir des exports (NAS, PBS) |
| VM | Machine virtuelle QEMU/KVM, avec son propre noyau (VM 104, Home Assistant, Windows) |
| VMID | Numéro d'un invité Proxmox (100, 104…). Les VM créées par ims-ops reçoivent le prochain numéro libre |
| Template | VM modèle, clonée pour créer des VM. Le template actuel est `ubuntu-2404-ims-ops` (9001) |
| Cloud-Init | Configuration appliquée au premier démarrage d'une VM clonée. Ici, seulement le nom, le compte et la clé SSH |
| Passthrough | Transmission d'un périphérique du nœud à un invité : disques du NAS (`mp0`, `mp1`), iGPU de la VM 104 (`hostpci0`) |
| Pool `backup` | Pool Proxmox qui désigne les machines sauvegardées chaque nuit |

## Stockage et sauvegarde

| Terme | Définition |
| --- | --- |
| `storage` | Partage du HDD de 3 To : médiathèque, photos RAW, documents, datastore PBS |
| `storage-hot` | Partage du SSD de 4 To : Immich, Forgejo |
| Bind-mount | Montage d'un dossier à un second emplacement. `/mnt/storage` est un bind-mount de `/mnt/disk1` |
| FUSE, MergerFS | Système de fichiers en espace utilisateur. MergerFS agrégeait des disques sous `/mnt/storage` ; retiré le 23/10/2026 car incompatible avec un export NFS ([ADR-012](/history/adr/adr-012-retrait-de-mergerfs)) |
| `ESTALE` | Erreur NFS « Stale file handle » : le client tient une référence que le serveur ne reconnaît plus |
| Lien physique (hardlink) | Deuxième nom pour un même fichier, sans copie. Radarr et Sonarr relient ainsi les téléchargements à la bibliothèque |
| PBS | Proxmox Backup Server : stocke les sauvegardes des invités par blocs dédupliqués |
| Datastore | Espace de stockage du PBS (`pve-backups`, sur le HDD du NAS) |
| `vzdump` | Outil de sauvegarde de Proxmox VE ; vers un stockage PBS, il envoie des blocs, pas une archive |
| Élagage (prune) | Suppression des sauvegardes au-delà de la rétention |

## Réseau et sécurité

| Terme | Définition |
| --- | --- |
| Tailnet | Réseau privé WireGuard entre les machines enrôlées, coordonné par Headscale (`100.64.0.0/10`) |
| Headscale, Headplane | Serveur de coordination du tailnet, et son interface d'administration |
| Utilisateur `infra` | Utilisateur Headscale de toutes les VM et de tous les conteneurs |
| Clé de pré-authentification | Jeton qui permet à une machine de rejoindre le tailnet sans connexion interactive |
| `extra_records` | Noms DNS servis aux clients du tailnet, pointés vers l'IP tailnet de Traefik |
| `vmbr0`, `vmbr1` | Ponts réseau de Proxmox : LAN, et réseau interne sans passerelle entre le NAS, le PBS et la VM 104 |
| Traefik | Reverse proxy unique du homelab, géré par Coolify |
| `vpn-only` | Middleware Traefik qui n'accepte que le tailnet et le LAN ; fichier `vpn-only.yaml` |
| Forward-auth | Traefik demande à l'outpost Authentik si l'utilisateur est connecté avant de transmettre la requête |
| OIDC | OpenID Connect : l'application délègue la connexion à Authentik |
| Défi DNS-01 | Preuve de propriété d'un domaine par un enregistrement TXT, pour obtenir un certificat Let's Encrypt |
| CrowdSec, bouncer, AppSec | Détection d'attaques à partir des journaux ; le bouncer applique les bannissements dans Traefik ; AppSec est son pare-feu applicatif |
| fail2ban | Bannissement local des adresses qui échouent en SSH sur les hôtes |
| Compte de service | Compte réservé à un outil, avec une clé acceptée seulement depuis ses adresses (`termix-usr`) |

## Administration

| Terme | Définition |
| --- | --- |
| ims-ops | Dépôt d'administration du parc : Ansible et `just` ([page](/procedures/administration-ims-ops)) |
| `just` | Lanceur de commandes du dépôt (`just check`, `just setup`, `just vm`…) |
| Socle commun | Configuration appliquée par ims-ops à tous les hôtes ([page](/infrastructure/socle-commun)) |
| Rebond SSH (ProxyJump) | Connexion à une machine en passant par une autre ; utilisé pour une VM pas encore dans le tailnet |
| Coolify, worker | Orchestrateur des services Docker (VM 104) et son nœud d'exécution distant (LXC 105) |
| Alloy | Agent de Grafana qui envoie métriques et journaux à Prometheus et Loki |
| ADR | Architecture Decision Record : trace d'une décision et de ses raisons ([index](/history/adr/index)) |

## Matériel

| Terme | Définition |
| --- | --- |
| Labrax | Rack 10 pouces imprimé en 3D qui accueille le matériel ([page](/infrastructure/labrax)) |
| MS-01 | Minisforum MS-01, nœud principal (`pve`) |
| PicoPSU-160-XT | Petite alimentation 12 V qui alimente les disques et le ventilateur du rack |
| ASM1166 | Contrôleur SATA du MS-01 ; ne gère pas le branchement à chaud |
