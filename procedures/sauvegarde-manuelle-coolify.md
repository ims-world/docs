---
title: "Sauvegarde Manuelle de Coolify & Résolution d'Erreurs"
description: "Guide pas-à-pas pour la sauvegarde manuelle à chaud de la VM Coolify (104), des bases de données et la résolution du piège PBS namespace not found"
icon: "cube"
iconType: "duotone"
---

<Badge color="green">🟢 Procédure Standard Validée</Badge>

## Objectif

Cette procédure décrit comment réaliser une **sauvegarde manuelle à chaud** de l'environnement d'orchestration **VM IMS-Coolify (VM 104)** avant toute intervention majeure ou mise à niveau.

Elle couvre la sauvegarde complète au niveau hyperviseur, la sauvegarde ciblée des fichiers de configuration CLI, les exports de bases de données applicatives, et détaille le correctif de l'erreur Proxmox `command error: namespace not found`.

---

## Méthode 1 — Sauvegarde Globale à Chaud par Hyperviseur Proxmox (Recommandée)

Cette méthode est la plus sûre car elle capture en une commande l'intégralité de la VM 104 (système d'exploitation, démon Docker, arborescence `/data/coolify/` et tous les **volumes Docker nommés** dans `/var/lib/docker/volumes/`).

### Options de Stockage Cible

<Tabs>
  <Tab title="💾 Stockage Local NVMe (Inratable & Rapide)">
    Exécuter la commande depuis l'hôte **Proxmox MS-01** (SSH ou console web) :

    ```bash
    vzdump 104 --storage local --mode snapshot --compress zstd
    ```

    - **Mode** : `snapshot` (sauvegarde à chaud sans aucune interruption de service).
    - **Cible** : Stockage local NVMe (`/var/lib/vz/dump/`).
    - **Durée** : ~10 à 15 secondes pour générer l'archive `.vma.zstd`.
  </Tab>

  <Tab title="🛡️ Stockage Proxmox Backup Server (PBS)">
    Exécuter la commande vers le datastore PBS :

    ```bash
    vzdump 104 --storage pbs-coolify --mode snapshot
    ```

    <Warning>
    **Dépannage de l'erreur `command error: namespace not found`** :
    
    Si Proxmox retourne l'erreur suivante :
    ```text
    ERROR: VM 104 qmp command 'backup' failed - backup connect failed: command error: namespace not found
    ```
    **Cause** : Le paramètre **Namespace** configuré dans *Datacenter → Storage → pbs-coolify* sur Proxmox VE pointe vers un Namespace qui n'existe pas dans le datastore du serveur PBS (LXC 103).

    **Résolution** :
    1. Dans l'IHM Proxmox VE : Aller dans **Datacenter → Storage → pbs-coolify → Edit**.
    2. Vider le champ **Namespace** (ou saisir un Namespace existant créé dans le datastore PBS).
    3. Relancer la commande `vzdump 104 --storage pbs-coolify --mode snapshot`.
    </Warning>
  </Tab>
</Tabs>

---

## Méthode 2 — Sauvegarde Manuelle CLI Fichiers & Configs (depuis la VM 104)

Pour effectuer une sauvegarde légère de la configuration Coolify et des fichiers Compose sans sauvegarder l'intégralité de la VM :

```bash
# 1. Se connecter en SSH à la VM Coolify
ssh cmolotkoff@192.168.1.52

# 2. Créer le dossier d'export
mkdir -p ~/backups_coolify && cd ~/backups_coolify

# 3. Exporter à chaud la BDD SQLite interne de Coolify
docker exec coolify sqlite3 /var/lib/coolify/database.db ".backup '/var/lib/coolify/database_dump.db'"

# 4. Archiver l'arborescence des services /data/coolify/
sudo tar -czvf coolify-data-$(date +%Y%m%d_%H%M%S).tar.gz /data/coolify/
```

---

## Méthode 3 — Sauvegarde Ciblée des Bases de Données Applicatives

Pour extraire un dump SQL d'une base de données applicative spécifique avant une maintenance :

<CodeGroup>
  ```bash PostgreSQL (Authentik, Immich, Forgejo, Zipline, Patrimo)
  # Dump d'une base PostgreSQL conteneurisée
  docker exec -t <nom-conteneur-db> pg_dump -U <user> -d <nom-base> -F c > backup_<base>_$(date +%Y%m%d).dump

  # Exemple Authentik :
  docker exec -t postgresql-k5mxvc2r6c4zlb6j3d443h7b pg_dump -U authentik -d authentik -F c > backup_authentik.dump
  ```

  ```bash MariaDB (PhotoPrism)
  # Dump de la base MariaDB PhotoPrism
  docker exec -t mariadb-yfotvbtkqj8cqw5alox6gfpr mariadb-dump -u photoprism -p<password> photoprism > backup_photoprism.sql
  ```
</CodeGroup>

---

## Restauration d'Urgence (Break-Glass)

### Restauration d'un Snapshot Proxmox (VM 104)

1. En cas d'incident majeur sur la VM 104, aller dans l'IHM Proxmox VE :
   **VM 104 (ims-pve-104-coolify) → Backup → Sélectionner le snapshot → Restore**.
2. Valider le démarrage de la VM. Le reverse proxy Traefik et les conteneurs Docker reprennent leur état exact au moment du snapshot.
