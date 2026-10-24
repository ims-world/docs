---
title: "Administration avec ims-ops"
description: "Ce que fait le dépôt ims-ops, ses garde-fous, et les parcours courants : configurer un hôte, créer une VM, gérer les sauvegardes."
icon: "terminal"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : quelques minutes par parcours · Risque : faible avec la simulation préalable · Interruption de service : aucune en général

## Quand l'utiliser

ims-ops est la boîte à outils d'administration du parc. Ansible décrit l'état attendu des machines ; toutes les commandes passent par `just`, lancé depuis le poste de l'administrateur. La liste complète des commandes et leur détail sont dans le [README du dépôt](https://github.com/ims-world/ims-ops).

| ims-ops gère | ims-ops ne gère pas |
| --- | --- |
| Comptes, clés SSH, sudo | Les services Docker (Coolify) |
| Socle système : SSH, fail2ban, paquets, mises à jour | Les mises à jour des nœuds Proxmox |
| Agent Alloy et enrôlement dans le tailnet | Home Assistant OS, Windows, le Raspberry Pi |
| Template de VM, création et destruction de VM | Les conteneurs LXC existants (créés à la main) |
| Pool de sauvegarde PBS | La rétention et les vérifications du PBS |
| Rapports : conformité, doublons de la médiathèque | La suppression de fichiers |

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
| Détecter, pas corriger | `just audit` et `just duplicates` produisent des rapports et ne modifient rien |
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
```

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

### Chercher les doublons de la médiathèque

```bash
just duplicates   # rapport dans reports/duplicates/latest.md
```

Voir [ims-nas](/infrastructure/ims-nas).

## Vérification

- `just ping` : connexion SSH et sudo sur tout le parc.
- `just audit` : aucun écart entre l'état réel et l'inventaire.

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
