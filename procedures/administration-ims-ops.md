---
title: "Administration avec ims-ops"
description: "Ce que fait le dépôt ims-ops, ses garde-fous, et les parcours courants : configurer un hôte, créer une VM, gérer les sauvegardes."
icon: "terminal"
iconType: "duotone"
last_reviewed: "2026-11-08"
---

Durée : quelques minutes par parcours · Risque : faible avec la simulation préalable · Interruption de service : aucune en général

## Quand l'utiliser

ims-ops est la boîte à outils d'administration du parc. Ansible décrit l'état attendu des machines ; toutes les commandes passent par `just`, lancé depuis le poste de l'administrateur. La liste complète des commandes et leur détail sont dans le [README du dépôt](https://github.com/ims-world/ims-ops).

| ims-ops gère | ims-ops ne gère pas |
| --- | --- |
| Comptes, clés SSH, sudo | Le contenu des services Docker (composes, configuration dans Coolify) |
| Socle système : SSH, fail2ban, paquets, mises à jour | Les mises à jour des nœuds Proxmox |
| Agent Alloy et enrôlement dans le tailnet | Home Assistant OS, Windows, le Raspberry Pi |
| Template de VM, création et destruction de VM | Les conteneurs LXC existants (créés à la main) |
| Pool de sauvegarde PBS | La rétention et les vérifications du PBS |
| Rapports : conformité, doublons de la médiathèque | La suppression de fichiers |
| Mise à jour de l'image d'un service Coolify | Les healthchecks et autres réglages des composes |

Le contenu exact du socle est décrit dans [Socle commun des hôtes](/infrastructure/socle-commun).

## Prérequis

- Le dépôt cloné sur le poste, `just` et `uv` installés, `.env` renseigné (`IMS_USER`), puis `just install`.
- Une clé SSH autorisée sur les hôtes pour le compte de l'administrateur.
- Le poste dans le tailnet, ou sur le LAN avec `IMS_NETWORK=lan`.

## Principes et garde-fous

| Principe | Effet |
| --- | --- |
| Une seule source par valeur | Toute valeur propre au site vit dans `inventory/`. Elle sert à la fois d'entrée aux rôles et d'état attendu pour l'audit |
| Simuler avant d'appliquer | `just check` affiche les écarts sans rien modifier ; `just setup` n'est lancé qu'après lecture du diff |
| Jamais root | Connexion avec le compte personnel et sudo |
| Aucun secret en clair | Secrets dans un coffre Ansible Vault, mot de passe dans le trousseau macOS |
| Détecter, pas corriger | `just audit`, `just health` et `just homeflix-doublons` produisent des rapports et ne modifient rien |
| Inventaire en deux fichiers | `hosts.yml` est tenu à la main ; `vms-created.yml` est écrit par `just vm`. Seules les VM de ce second fichier peuvent être détruites par la commande |
| Rebond SSH | Une VM sans IP tailnet est jointe en rebondissant par le nœud des VM (`pve`) |
| Arrêt au premier échec | Une opération sur plusieurs hôtes s'arrête dès qu'un hôte échoue ou ne répond pas |

**Cible** d'une commande : un hôte (`pve`), un groupe (`lxc`, `vms`, `proxmox_nodes`, `third_party`), `managed` pour tout le parc, ou une combinaison Ansible (`'managed:!lab-elo'`).

**Tags** : limitent `check` et `setup` à une partie de la configuration (`accounts`, `base`, `packages`, `system`, `sshd`, `fail2ban`, `updates`, `tailscale`, `alloy`). `just tags` les liste.

## Étapes

### Modifier la configuration d'un hôte

<Steps>
  <Step title="Modifier la valeur dans l'inventaire">
    `group_vars/all/main.yml` (tout le parc), `group_vars/<groupe>.yml` ou `host_vars/<hôte>.yml`.
  </Step>
  <Step title="Simuler">
    ```bash
    just check <cible> [tags]
    ```
    Chaque écart annoncé doit être compris avant d'appliquer.
  </Step>
  <Step title="Appliquer">
    ```bash
    just setup <cible> [tags]
    ```
  </Step>
  <Step title="Contrôler">
    ```bash
    just check <cible>   # doit annoncer changed=0
    just audit <cible>   # rapport dans reports/audit/latest.md
    ```
  </Step>
</Steps>

### Créer et détruire une VM

```bash
just vm web-01                  # Ubuntu, gabarit S, configuration complète et sauvegarde
just vm essai ubuntu M create   # VM brute, sans configuration ni sauvegarde
just vm-destroy essai           # demande de retaper le nom
```

| Gabarit | Cœurs | RAM | Disque |
| --- | --- | --- | --- |
| S | 2 | 2 Go | 20 Go |
| M | 4 | 4 Go | 40 Go |
| L | 8 | 8 Go | 80 Go |

`just vm` choisit le prochain VMID et la première IP libre entre `192.168.1.200` et `.249`, inscrit la VM dans `inventory/vms-created.yml`, puis, en mode `full`, la configure, l'enrôle dans le tailnet et l'ajoute au pool de sauvegarde. Relire le `git diff` de l'inventaire et le commiter.

`just vm-destroy` retire la VM du tailnet, la détruit dans Proxmox et la retire de l'inventaire. Ses sauvegardes existantes sont conservées.

### Gérer la sauvegarde d'une machine

```bash
just backup-list            # machines sauvegardées ou non
just backup-add <nom|vmid>  # ajoute au pool backup
just backup-remove <nom|vmid>
just backup-now <nom|vmid>  # sauvegarde immédiate, avant une intervention
```

`just backup-now` sauvegarde vers le PBS, sans arrêt de l'invité. Le NAS et le PBS, hors pool, sont sauvegardés en archive locale ; le NAS est arrêté le temps de la sienne, ce qui coupe le NFS.

Le fonctionnement de la sauvegarde est décrit dans la [politique de sauvegarde](/procedures/politique-sauvegardes).

### Faire tourner une clé SSH d'administrateur

<Steps>
  <Step title="Ajouter la nouvelle clé">
    Ajouter la clé publique en seconde ligne de `keys/admins/<compte>.pub`, puis `just check managed accounts` et `just setup managed accounts`.
  </Step>
  <Step title="Tester la nouvelle clé seule">
    ```bash
    ssh -F /dev/null -i <clé privée> -o IdentitiesOnly=yes <compte>@<hôte> hostname
    ```
    `-F /dev/null` est nécessaire : sans lui, `ssh -i` ajoute la clé à celles de `~/.ssh/config` au lieu de la tester seule.
  </Step>
  <Step title="Retirer l'ancienne clé">
    Supprimer l'ancienne ligne, puis `just setup managed accounts`.
  </Step>
</Steps>

Un hôte éteint pendant la rotation garde l'ancienne clé jusqu'à son prochain `just setup <hôte> accounts`.

### Retirer une machine du tailnet

```bash
just tailnet-remove <nom>
```

La commande refuse les hôtes déclarés dans `hosts.yml`.

### Contrôler l'état du parc

```bash
just health   # alertes à l'écran, rapport complet dans reports/health/latest.md
```

La commande ne modifie rien. Elle complète `just audit` : l'audit vérifie la conformité de la configuration, `health` vérifie que le parc fonctionne.

<CardGroup cols={2}>
  <Card title="Hôtes" icon="server" color="#3B82F6">Joignables, résolution DNS, redémarrage en attente, occupation des disques et des partages, unités systemd en échec, mises à jour en attente.</Card>
  <Card title="Cluster" icon="network-wired" color="#FF6B00">Quorum, état de chaque invité.</Card>
  <Card title="Sauvegardes" icon="box-archive" color="#8B5CF6">Âge de la dernière sauvegarde de chaque invité du pool, archives locales, occupation du datastore.</Card>
  <Card title="Disques" icon="hard-drive" color="#8B5CF6">Santé SMART relevée sur les nœuds, sans réveiller un disque en veille.</Card>
  <Card title="Ports à l'écoute" icon="ethernet" color="#FF6B00">Services joignables depuis le réseau sur chaque hôte, pour repérer un service oublié. Les ports publiés par Docker n'y figurent pas.</Card>
</CardGroup>

Les seuils sont dans `group_vars/all/main.yml` (`health_disk_warn_percent`, `health_backup_max_age_hours`) ; les partages contrôlés, dans `health_paths` de chaque hôte.

### Mettre à jour les paquets

```bash
just update ims-pbs         # un hôte
just update proxmox_nodes   # les nœuds, un à la fois
```

<Steps>
  <Step title="Lister">
    La commande rafraîchit le cache APT et affiche les paquets en attente de chaque hôte.
  </Step>
  <Step title="Confirmer">
    Rien n'est appliqué sans réponse `y`.
  </Step>
  <Step title="Appliquer">
    `apt dist-upgrade`, un hôte à la fois, arrêt au premier échec. Sur un nœud Proxmox, la commande refuse de continuer si le cluster n'est pas complet.
  </Step>
  <Step title="Redémarrer à la main">
    La commande ne redémarre jamais : elle signale les hôtes à redémarrer.
  </Step>
</Steps>

Les machines de tiers ne sont mises à jour que si elles sont nommées explicitement. Sur un hôte sans snapshot possible (VM 104, NAS), lancer d'abord `just backup-now <nom>`.

### Mettre à jour un service Coolify

```bash
just app-update <service> <version>       # ex. just app-update vaultwarden 1.37.6
just app-update <service> <version> yes   # avec sauvegarde PBS préalable de la VM 104 (une vingtaine de minutes)
```

`<service>` est le début du nom du conteneur (`<service>-<uuid Coolify>`), la version est le tag signalé par [WUD](/services/whatsupdocker). La commande affiche les versions publiées sur GitHub entre l'actuelle et la cible, demande confirmation, remplace le tag dans le compose par l'API de Coolify (compose d'origine conservé sur la VM 104), redéploie et attend que le conteneur soit sain. Les services d'une même image (application et worker) sont mis à jour ensemble ; toute la ressource Coolify redémarre.

- Sauvegarde préalable (`yes`) : pour une version majeure ou une migration de données sans retour ; sinon, le retour arrière repose sur l'ancien tag et la sauvegarde de la nuit.
- Échec : la commande affiche le dernier état du conteneur et la commande de retour arrière. Un conteneur `unhealthy` alors que le service répond vient souvent d'un healthcheck devenu incompatible avec la nouvelle image (cas de Loki et PhotoPrism).
- Prérequis dans Coolify : accès à l'API activé et limité à la VM 104 (*Settings*, *Allowed IPs* : `10.0.1.1`), jeton `read:sensitive`, `write`, `deploy`, rangé dans le coffre ims-ops.
- Une ressource dont plusieurs images partagent un même compose (supervision) se met à jour plus simplement à la main, en un seul redéploiement.

### Tester une restauration

```bash
just restore-test <nom|vmid>   # rapport dans reports/restore-test/latest.md
```

La commande restaure la dernière sauvegarde PBS dans une VM jetable (VMID 900), l'isole, la démarre, attend une réponse, puis la détruit. Le détail est dans le [plan de reprise](/procedures/plan-reprise-activite-pra#tester-les-sauvegardes).

### Chercher les doublons de la médiathèque

```bash
just homeflix-doublons   # rapport dans reports/duplicates/latest.md
```

Voir [ims-nas](/infrastructure/ims-nas).

## Vérification

- `just ping` : connexion SSH et sudo sur tout le parc.
- `just audit` : aucun écart entre l'état réel et l'inventaire.
- `just health` : aucune alerte.

## Retour arrière

- Configuration : remettre l'ancienne valeur dans l'inventaire, puis `just setup`. Sur un hôte sans snapshot possible (VM 104), sauvegarder les fichiers touchés avant d'appliquer.
- VM créée par erreur : `just vm-destroy <nom>`.

## Dépannage

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `Errno 38` sur une tâche longue dans un LXC | Le mode `async` d'Ansible ne fonctionne pas dans les conteneurs | Les tâches longues sont lancées sans `async` |
| Alerte « REMOTE HOST IDENTIFICATION HAS CHANGED » | Headscale a réattribué l'adresse d'une ancienne machine | Vérifier l'empreinte sur l'hôte, puis `ssh-keygen -R <adresse>` |
| Tailnet indisponible | — | Relancer la commande avec `IMS_NETWORK=lan` |
| Erreur d'encodage à la création d'une VM | Caractères accentués dans la description passée à `qm` | Description en ASCII |
