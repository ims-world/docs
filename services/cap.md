---
title: "Cap"
description: "Enregistrement et partage d'écran self-hosted"
icon: "video"
iconType: "duotone"
---

<Info>
**Statut: En attente de déploiement** — Stack technique et dumps préparés (base MySQL 202 Mo + données MinIO 28 Mo).
</Info>

## Ingestion & Restauration des Données

<Tabs>
  <Tab title="🐬 Dump & Restauration MySQL">
    <Warning>
    L'utilisateur applicatif n'a pas les privilèges `PROCESS` requis pour un dump complet — utiliser le compte **root** :
    </Warning>
    ```bash
    # Sauvegarde MySQL via compte root
    docker exec cap-db mysqldump -u root -p'<password>' --no-tablespaces planetscale > dump.sql

    # Restauration dans le nouvel environnement
    docker exec -i cap-db mysql -u root -p'<password>' planetscale < dump.sql
    ```
  </Tab>
  <Tab title="🪣 Backup Données Objets MinIO (S3)">
    MinIO stocke ses objets directement comme fichiers sur disque :
    ```bash
    docker stop cap-s3
    sudo tar czf minio-data.tar.gz -C /data/coolify/services/<uuid>/s3 .
    docker start cap-s3
    ```
  </Tab>
  <Tab title="🔑 Clefs de Chiffrement Critiques">
    <Warning>
    `DATABASE_ENCRYPTION_KEY` et `NEXTAUTH_SECRET` doivent être **repris à l'identique** — contrairement aux credentials MySQL/MinIO (régénérables sans risque), ces deux clés chiffrent/signent des données existantes dans le dump. Une nouvelle valeur casserait le déchiffrement des données.
    </Warning>
  </Tab>
</Tabs>

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine visé** | `cap.ims-world.fr` |
| **Composants** | `cap-web` (Next.js) + `cap-db` (MySQL 8.0) + `cap-s3` (MinIO) |
| **Statut** | ⏸️ Préparé (Pause) |

## Architecture Cap & État de la Migration

```mermaid
graph TD
    subgraph WAN ["🌐 Public Request"]
        REQ["cap.ims-world.fr"]
    end

    subgraph CAP_STACK ["🎥 Stack Cap (En Pause sur VM 104)"]
        WEB["cap-web (Next.js - SHA256 Pinned)"]
        DB["cap-db (MySQL 8.0 - Base PlanetScale 202 Mo)"]
        S3["cap-s3 (MinIO - Stockage Objets 28 Mo)"]
        KEYS["DATABASE_ENCRYPTION_KEY & NEXTAUTH_SECRET (A préserver!)"]
    end

    subgraph MIGRATION_STATE ["⏸️ Statut des Data Backup"]
        DUMP["dump.sql (MySQL Root Dump OK)"]
        MINIO_TAR["minio-data.tar.gz (MinIO Data Tar OK)"]
        RESTORE["Restore DB & MinIO en attente"]
    end

    REQ --> WEB
    WEB --> DB
    WEB --> S3
    WEB <--> KEYS

    DUMP -.-> RESTORE
    MINIO_TAR -.-> RESTORE
    RESTORE -.-> DB
    RESTORE -.-> S3

    classDef web fill:#F97316,stroke:#FB923C,color:#fff;
    classDef state fill:#2c3e50,stroke:#34495e,color:#fff;
    class WEB,DB,S3 web;
    class DUMP,MINIO_TAR,RESTORE state;
```

## Versions & Retours d'Expérience

<AccordionGroup>
  <Accordion title="Versions des images à figer">
    <Warning>
    `cap-web` n'expose aucun tag de version lisible — utiliser le digest exact plutôt que `latest` :
    ```yaml
    image: 'ghcr.io/capsoftware/cap-web@sha256:bd905035ea8fc4517a38add2cc74fde61cc8e332dd46d534a65692cbdd2b0b81'
    ```
    </Warning>

    | Composant | Référence figée |
    |---|---|
    | cap-web | digest `sha256:bd905035...` |
    | MySQL | `8.0` |
    | MinIO | `RELEASE.2025-09-07T16-13-09Z` |
  </Accordion>

  <Accordion title="Découverte : Présence de données réelles (202 Mo DB + 28 Mo S3)">
    <Warning>
    L'ancien plan classait Cap comme "recréation à neuf, aucune donnée" — **faux**, vérifié en session : 202 Mo de base MySQL + 28 Mo de données MinIO réelles (enregistrements). Toujours vérifier par soi-même plutôt que de faire confiance à une note qui peut dater.
    </Warning>
  </Accordion>

  <Accordion title="Checklist d'activation finale">
    <Steps>
      <Step title="Initialiser MySQL">
        Démarrer MySQL une première fois pour initialiser sa structure.
      </Step>
      <Step title="Restaurer la base">
        Restaurer le dump SQL par-dessus.
      </Step>
      <Step title="Vérifier les permissions MinIO">
        Vérifier la cohérence des permissions du dossier `s3/`.
      </Step>
      <Step title="Sécuriser les identifiants">
        Régénérer `MINIO_ROOT_PASSWORD` (exposé en clair pendant la préparation).
      </Step>
      <Step title="Validation fonctionnelle">
        Tester fonctionnellement avant bascule du domaine réel.
      </Step>
    </Steps>
  </Accordion>
</AccordionGroup>
