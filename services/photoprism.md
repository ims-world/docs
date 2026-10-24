---
title: "PhotoPrism"
description: "Bibliothèque des photos haute résolution et des fichiers RAW, alimentée par WebDAV."
icon: "camera"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "260728 / MariaDB 11.8.8"
---

## Accès

<CardGroup cols={2}>
  <Card title="PhotoPrism" icon="camera" href="https://studio.ims-world.fr">
    Bibliothèque, recherche, albums.
  </Card>
  <Card title="Dépôt WebDAV" icon="upload">
    `https://studio.ims-world.fr/import/`
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
docker exec -it $(docker ps -qf "name=photoprism_photoprism") photoprism import   # traiter /import/
docker exec -it $(docker ps -qf "name=photoprism_photoprism") photoprism index    # réindexer
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `studio.ims-world.fr` |
| Accès | Public ; OIDC Authentik (client `photo-prism`, groupe `admins`) |
| Hôte | VM 104 |
| Coolify | UUID `yfotvbtkqj8cqw5alox6gfpr`, chemin `/data/coolify/services/yfotvbtkqj8cqw5alox6gfpr/` |
| Image | `photoprism/photoprism:260728`, `mariadb:11.8.8` |
| Données | Originaux et métadonnées sur le HDD du NAS (`/mnt/nas-storage/photoprism-data/`) ; base MariaDB dans un volume local (`photoprism-mariadb-data`) |
| Sauvegarde | Base avec la VM 104 ; **originaux non sauvegardés** |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 17/08/2026 |

## Fonctionnement

| Emplacement | Contenu |
| --- | --- |
| `originals/` | Photos et fichiers RAW |
| `storage/sidecar/` | Modifications manuelles (mots-clés, visages, lieux) au format YAML : à conserver avec les originaux |
| Volume `photoprism-mariadb-data` | Base MariaDB, sur le disque local de la VM : une base relationnelle sur NFS pose des problèmes de verrous et de performances |

## Dépôt de photos par WebDAV

PhotoPrism a son propre serveur WebDAV : un dépôt depuis n'importe quel ordinateur, sans partage réseau à configurer.

| Dossier | Usage |
| --- | --- |
| `/import/` | Dépôt : les fichiers sont convertis si besoin (HEIC vers JPEG), classés par date, puis déplacés dans la bibliothèque |
| `/originals/` | Bibliothèque déjà indexée, en lecture et écriture. Ne pas y déposer de nouvelles photos |

<Steps>
  <Step title="Obtenir un mot de passe WebDAV">
    Un compte créé par OIDC n'a pas de mot de passe local : créer un mot de passe d'application dans **Settings → Account → Apps and Devices**. À défaut, utiliser le compte administrateur local (`PHOTOPRISM_ADMIN_PASSWORD` dans les variables Coolify).
  </Step>
  <Step title="Se connecter">
    macOS : Finder, **Aller → Se connecter au serveur** (`Cmd+K`), `https://studio.ims-world.fr/import/`. Windows : **Connecter un lecteur réseau**, avec l'adresse donnée dans **Settings → Account → Connect via WebDAV**.
  </Step>
  <Step title="Lancer l'import">
    Après le dépôt, lancer `photoprism import` (commande plus haut) : les fichiers `._` créés par le Finder retardent l'import automatique de plus de 20 minutes.
  </Step>
</Steps>

## Transcodage

```yaml
devices:
  - /dev/dri:/dev/dri
environment:
  PHOTOPRISM_FFMPEG_ENCODER: intel
  PHOTOPRISM_INIT: 'intel tensorflow'
```

`PHOTOPRISM_INIT` doit contenir `intel` : sans lui, les paquets nécessaires à l'iGPU ne sont pas installés et FFmpeg utilise l'encodeur logiciel, sans message d'erreur. L'utilisation effective de l'iGPU n'a pas été confirmée (erreurs `MFX session` connues sur les processeurs Intel récents). La reconnaissance faciale reste sur le processeur.

## Données et sauvegarde

<Warning>
Les originaux et les métadonnées YAML sont sur le HDD du NAS, sans sauvegarde : seule la base est dans la sauvegarde de la VM 104. Voir la [politique de sauvegarde](/procedures/politique-sauvegardes#ce-qui-nest-pas-sauvegardé).
</Warning>

Export de la base :

```bash
docker exec $(docker ps -qf "name=photoprism_mariadb") mariadb-dump -u root -p photoprism > photoprism-$(date +%F).sql
```

## Exploitation

Mise à jour : changer le tag dans Coolify, redéployer, puis lancer `photoprism index` si les notes de version le demandent.

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| Pas de mot de passe pour WebDAV | Compte OIDC | Mot de passe d'application |
| Import qui tarde après un dépôt depuis un Mac | Fichiers `._` du Finder | Lancer `photoprism import` |
| Transcodage logiciel malgré l'iGPU | `PHOTOPRISM_INIT` sans `intel` | Ajouter `intel` et redéployer |

## Voir aussi

- [Immich](/services/immich)
- [ims-nas](/infrastructure/ims-nas)
