---
title: "Immich"
description: "Photothèque et vidéothèque avec sauvegarde depuis les téléphones, reconnaissance des visages et recherche par description."
icon: "images"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v3.2.0"
---

## Accès

Public, avec connexion Authentik ou compte natif. Les applications mobiles se configurent sur ce serveur.

<CardGroup cols={2}>
  <Card color="#8B5CF6" title="Immich" icon="images" href="https://photos.ims-world.fr">
    Interface web ; applications iOS et Android sur ce serveur.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
cd /data/coolify/services/p3ujda5c7sc8nf4j9zzd8lck/ && docker compose logs -f --tail=100
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `photos.ims-world.fr` |
| Accès | Public ; OIDC Authentik ou comptes natifs |
| Hôte | VM 104 |
| Coolify | UUID `p3ujda5c7sc8nf4j9zzd8lck`, chemin `/data/coolify/services/p3ujda5c7sc8nf4j9zzd8lck/` |
| Image | `ghcr.io/immich-app/immich-server:v3.2.0`, `immich-machine-learning:v3.2.0`, `postgres:14-vectorchord0.3.0-pgvectors0.2.0`, `redis:7.4-alpine` |
| Données | Photos et vidéos sur le SSD du NAS (`/mnt/nas-hot/immich-data`) ; base PostgreSQL sur la VM 104 |
| Sauvegarde | Base avec la VM 104 ; **photos non sauvegardées** (voir plus bas) |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 11/08/2026 |

## Fonctionnement

Quatre conteneurs : le serveur, l'apprentissage automatique, la base et la file des tâches.

<CardGroup cols={2}>
  <Card color="#8B5CF6" title="immich-server" icon="images">API et interface web, port 2283.</Card>
  <Card color="#16A34A" title="immich-machine-learning" icon="brain">Reconnaissance des visages et recherche par description (CLIP), sur le processeur.</Card>
  <Card color="#8B5CF6" title="database" icon="database">PostgreSQL avec l'extension vectorielle VectorChord.</Card>
  <Card color="#16A34A" title="redis" icon="layer-group">File des tâches.</Card>
</CardGroup>

Dossiers dans `/mnt/nas-hot/immich-data` : `upload/` (originaux), `library/`, `thumbs/`, `encoded-video/`, `profile/`, `backups/`.

**Choix : apprentissage automatique sur le processeur.** L'accélération OpenVINO par l'iGPU réduirait une indexation complète de 7 heures à moins d'une heure, mais l'indexation initiale est faite ; pour le flux quotidien, le gain ne justifie pas la complexité.

## Données et sauvegarde

<Warning>
Les photos sont sur le SSD du NAS, qui n'est sauvegardé nulle part. Seule la base (albums, visages, métadonnées) est dans la sauvegarde de la VM 104. La protection des données du NAS est suivie dans la [feuille de route](/roadmap).
</Warning>

- `DB_STORAGE_TYPE=SSD` concerne la base locale de la VM, pas les photos.

## Exploitation

- Mise à jour : lire les notes de version (les versions majeures modifient la base), changer le tag des deux conteneurs Immich, redéployer.
- Les applications mobiles doivent être de la même version majeure que le serveur (v3 pour un serveur v3).

## Pièges connus

Chaque piège donne le symptôme observé, sa cause et la correction.

<AccordionGroup>
  <Accordion title="Redémarrages en boucle, erreur ENOENT">
    **Cause** : fichier marqueur `.immich` absent dans l'un des six dossiers.

    **Solution** : recréer les marqueurs.

    ```bash
    cd /mnt/nas-hot/immich-data
    for dir in upload library thumbs encoded-video profile backups; do
      mkdir -p "$dir" && touch "$dir/.immich"
    done
    ```
  </Accordion>
  <Accordion title="Pas de miniatures ni de visages">
    **Cause** : le dossier n'est monté que sur `immich-server` (modèle Coolify par défaut).

    **Solution** : monter `/mnt/nas-hot/immich-data:/usr/src/app/upload` sur les deux conteneurs Immich.
  </Accordion>
  <Accordion title="Application mobile qui ne synchronise plus">
    **Cause** : version majeure différente du serveur.

    **Solution** : mettre l'application à jour.
  </Accordion>
  <Accordion title="Échec du premier déploiement d'une base lancée à la main">
    **Cause** : réseau Docker du service absent.

    **Solution** : `docker network create p3ujda5c7sc8nf4j9zzd8lck`.
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card color="#8B5CF6" title="ims-nas" icon="hard-drive" href="/infrastructure/ims-nas">Partages NFS et SMB.</Card>
  <Card color="#16A34A" title="PhotoPrism" icon="camera" href="/services/photoprism">Bibliothèque des photos RAW.</Card>
</CardGroup>
