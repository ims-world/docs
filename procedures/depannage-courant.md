---
title: "Dépannage courant"
description: "Problèmes déjà rencontrés sur le homelab, classés par domaine : symptôme, cause, solution."
icon: "wrench"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : variable · Risque : selon la solution · Interruption de service : selon la solution

## Quand l'utiliser

Devant un symptôme inconnu, chercher ici avant de diagnostiquer de zéro. Chaque entrée vient d'un cas réel. Les pièges propres à un service sont dans la section **Pièges connus** de sa fiche ; les plus fréquents sont repris ici.

## Prérequis

- Accès SSH aux hôtes (tailnet, ou LAN avec `IMS_NETWORK=lan` pour ims-ops).
- Si rien ne répond, commencer par [Accès de secours](/procedures/commandes-urgence).

## Étapes

<Steps>
  <Step title="Tester depuis un autre appareil">
    Un test lancé depuis la machine qui héberge le service, vers son propre nom tailnet, donne des résultats trompeurs (503, délais). Tester depuis un autre appareil du tailnet, ou hors tailnet pour vérifier l'exposition.
  </Step>
  <Step title="Valider en ligne de commande">
    `curl -sv <url>` avant de conclure à une panne : le navigateur peut bloquer seul (voir Réseau).
  </Step>
  <Step title="Chercher le symptôme ci-dessous">
    Puis appliquer la solution et vérifier le résultat attendu.
  </Step>
</Steps>

### Réseau

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `ERR_ADDRESS_UNREACHABLE` dans Chrome, `curl` répond | Conflit du navigateur avec une route du tailnet (cause exacte non établie) | Autre navigateur (Safari), ou `thisisunsafe` sur la page d'erreur |
| `curl` renvoie `000` (aucune réponse) depuis Internet | Redirection de la Bbox vers la mauvaise machine | Vérifier que les règles 80, 443 et 2222 ciblent `192.168.1.52` |
| Service restreint refusé depuis le tailnet | Nom absent des `extra_records`, ou IP source masquée | Voir [Restreindre un service au tailnet](/procedures/securiser-service-vpn-only#dépannage) |
| Un sous-domaine ne résout plus | Enregistrement `_acme-challenge.<nom>` oublié chez OVH | [Incident du 24/08/2026](/history/incidents/2026-08-24-dns-wildcard-shadowing-share-ims-world-fr) |
| Certificats non renouvelés | Résolveur DNS OVH injoignable | `dig @213.251.128.1 google.com` ; garder `8.8.8.8` seul dans les résolveurs DNS-01 |
| Alerte « REMOTE HOST IDENTIFICATION HAS CHANGED » | Headscale a réattribué l'adresse d'une ancienne machine | Vérifier l'empreinte sur l'hôte, puis `ssh-keygen -R <adresse>` |

### Docker et Coolify

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `service error: port is missing` dans Traefik | Conteneur sur plusieurs réseaux Docker | Label `traefik.docker.network=coolify` |
| Fichier de configuration ignoré | Coolify a créé un dossier vide à la place du fichier ; le montage reste figé | `file <chemin>` doit répondre « text » ; sinon corriger, puis `docker rm -f <conteneur>` et redéployer (un simple redémarrage ne suffit pas) |
| Page blanche ou erreur CORS (Vaultwarden, qBittorrent) | Domaine figé dans un fichier de configuration, prioritaire sur les variables | `grep -ri "domain\|serverdomains\|hostheader" <fichier>`, corriger, vider le cache du navigateur |
| qBittorrent refuse les requêtes derrière Traefik | `HostHeaderValidation` actif, même avec `ServerDomains=*` | `WebUI\HostHeaderValidation=false` |
| Interface Coolify en 502 après une mise à jour | Proxy déconnecté de Coolify | `docker restart coolify-proxy` |
| Modification de `/data/coolify/proxy/docker-compose.yml` perdue | Fichier régénéré par Coolify | Modifier depuis l'interface Coolify |

### Stockage et NFS

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `Stale file handle` sur la VM 104 ou le PBS | Montage NFS périmé | VM 104 : `docker restart` des conteneurs concernés ; PBS : `umount -l /mnt/pbs-datastore && mount /mnt/pbs-datastore`, puis `systemctl restart proxmox-backup-proxy proxmox-backup` |
| Taille d'un dossier aberrante | `du` lancé séparément sur des dossiers qui partagent des liens physiques | Mesurer en une seule commande, ou `df -h` |
| `mount: Permission denied` (« access denied by server ») dans un LXC | Un LXC non privilégié ne peut pas monter de NFS, malgré `mount=nfs` | Conteneur privilégié |
| `smartd` ou `hd-idle` sans effet dans un LXC | Le conteneur voit le système de fichiers, pas le périphérique bloc | Les faire tourner sur le nœud ; vérifier avec `hdparm -C` |
| Disque absent du tableau « Gestion des disques » | Disque en veille : `smartmon.sh` ne le réveille pas | Normal ; il réapparaît à son réveil |

<Info>
Les `Stale file handle` répétés d'août à octobre 2026 venaient de MergerFS exporté en NFS. Le passage aux bind-mounts ext4, le 23/10/2026, a supprimé la cause ([incident](/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix)). Les contournements de l'époque (NFS v3 forcé, redémarrage du PBS après chaque redémarrage du NAS, sauvegarde du NAS désactivée) ne sont plus nécessaires en principe ; à confirmer lors du prochain redémarrage isolé du NAS.
</Info>

### Sauvegarde

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `namespace not found` | Datastore illisible (NFS périmé) ou namespace inexistant sur `pbs-coolify` | Remonter le datastore ; vider le champ **Namespace** du stockage |
| `wrong content type` | Sauvegarde vers `local-lvm` | Stockage `local` |
| Aucune alerte malgré un échec | Notifications non configurées sur l'outil | Voir la [politique de sauvegarde](/procedures/politique-sauvegardes#mise-en-place-du-job) |

### GPU (VM 104)

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `/dev/dri: no such file or directory` au démarrage de Jellyfin | Module `i915` absent pour le noyau démarré | `sudo apt install -y linux-modules-extra-$(uname -r) && sudo modprobe i915`, puis vérifier que `linux-modules-extra-generic` est installé ([incident du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri)) |

### Supervision

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `/metrics` d'Uptime Kuma en erreur | Étiquette créée pendant que le conteneur tourne | `docker restart uptime-kuma-il53bmpdybmss5q14sfy0umm` |
| Erreurs `hwmon` dans les journaux d'un LXC | Capteurs inaccessibles | Collecteur désactivé par ims-ops pour le groupe `lxc` |

### Administration (ims-ops, SSH, Termix)

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `Errno 38` sur une tâche Ansible longue dans un LXC | Mode `async` non supporté en conteneur | Tâche lancée sans `async` |
| Test d'une clé SSH faussé | `ssh -i` ajoute la clé à celles de `~/.ssh/config` | `ssh -F /dev/null -i <clé> -o IdentitiesOnly=yes` |
| Erreur d'encodage à la création d'une VM | Accents dans la description passée à `qm` | Description en ASCII |
| Aucun hôte dans Termix | Connexion avec le compte local au lieu du compte SSO | Se reconnecter avec le compte SSO |
| Boucle de redirection Termix vers Authentik | Connexion SSO automatique sans compte SSO existant | Voir [Termix](/services/termix) |

### Médiathèque

| Symptôme | Cause | Solution |
| --- | --- | --- |
| Espace non libéré après suppression dans Radarr ou Sonarr | Le fichier existe encore dans `downloads/` | Supprimer aussi le torrent avec ses fichiers dans qBittorrent |
| Import bloqué « Unable to parse file » | Qualité non déductible du nom | Import manuel, mode **Hardlink** |
| Téléchargements à la racine de `downloads/` | Torrent sans catégorie | Catégories qBittorrent avec dossier propre |

Détails dans [HomeFlix](/services/homeflix).

## Vérification

Le symptôme a disparu, et la vérification propre au service passe (sonde Uptime Kuma au vert, `just audit` sans écart pour un hôte).

## Retour arrière

Propre à chaque solution ; sauvegarder les fichiers touchés avant une modification manuelle.

## Dépannage

Un problème nouveau, une fois résolu, s'ajoute dans la section du domaine concerné, avec un lien vers l'incident s'il en a fait l'objet.
