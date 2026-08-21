---
title: "Catalogue des Procédures Opérationnelles"
description: "Index centralisé de toutes les procédures d'administration, d'exploitation, de secours et de PRA"
icon: "book-bookmark"
iconType: "duotone"
---

import { ips, domains } from "/snippets/variables.mdx";

<Info>
Ce catalogue répertorie **l'ensemble des procédures opératoires du homelab IMS-WORLD**. Les procédures d'exploitation applicatives sont maintenues directement au cœur de leurs fiches respectives pour garantir une source de vérité unique, et sont ici indexées par liens profonds.
</Info>

---

## 🚨 1. Urgence, Secours & Reprise d'Activité (PRA)

<CardGroup cols={2}>
  <Card title="Plan de Reprise d'Activité (PRA / DRP)" icon="shield-virus" href="/procedures/plan-reprise-activite-pra">
    Stratégie globale de résilience, matrice RTO/RPO et procédures de basculement de secours.
  </Card>
  <Card title="Simulation Crash NVMe & Restauration DRP" icon="skull-crossbones" href="/procedures/simulation-crash-restauration">
    Runbook théorique de restauration à froid d'urgence de la VM Coolify depuis PBS suite à un crash NVMe (avec avertissement).
  </Card>
  <Card title="Commandes d'Urgence Break-Glass" icon="bolt" href="/procedures/commandes-urgence">
    Aide-mémoire des commandes de secours SSH, déblocage des accès et redémarrage des composants critiques.
  </Card>
  <Card title="Clé de Récupération Authentik" icon="key" href="/services/authentik#acces-rapides--administration">
    Générer une clé administrateur d'urgence à usage unique en cas de perte du WebAuthn 2FA ou du mot de passe admin.
  </Card>
  <Card title="Backup & Restauration Vaultwarden" icon="shield-halved" href="/services/vaultwarden#acces-rapides--administration">
    Procédure d'arrêt à chaud et d'archivage sécurisé des fichiers WAL du coffre-fort SQLite Vaultwarden.
  </Card>
  <Card title="Politique de Sauvegarde & Tâches Planifiées" icon="shield-check" href="/infrastructure/politique-sauvegardes">
    Vue d'ensemble de la protection des données, chronologie nocturne, règle d'anti-circularité et backups vzdump local.
  </Card>
</CardGroup>

---

## ⚙️ 2. Infrastructure, Stockage & Déploiement

<CardGroup cols={2}>
  <Card title="Ajout d'un Nouveau Disque (LVM / NFS)" icon="hard-drive" href="/procedures/ajout-nouveau-disque">
    Guide pas-à-pas pour formater, monter et étendre un volume de stockage sur Proxmox VE, NAS ou VM Coolify.
  </Card>
  <Card title="Déploiement d'un Nouveau Service Coolify" icon="cube" href="/procedures/deploiement-service">
    Check-list de déploiement Docker Compose, attribution d'UUID, configuration du réseau et masquage DNS.
  </Card>
  <Card title="Sécuriser un Service avec vpn-only" icon="shield-check" href="/procedures/securiser-service-vpn-only">
    Procédure pas-à-pas pour isoler un sous-domaine d'administration sur le réseau privé Tailscale via vpn-only.yaml.
  </Card>
  <Card title="Sécuriser une App avec Authentik Outpost" icon="shield-keyhole" href="/procedures/securiser-application-authentik-forward-auth">
    Procédure complète pour protéger une application web sans SSO natif via Traefik et l'Outpost Proxy Authentik.
  </Card>
  <Card title="Sauvegarde Manuelle de Coolify" icon="cube" href="/procedures/sauvegarde-manuelle-coolify">
    Guide de sauvegarde à chaud via Proxmox NVMe/PBS, dumps SQL applicatifs et résolution du piège PBS namespace not found.
  </Card>
  <Card title="Dépannage Courant & Pièges Vécus" icon="wrench" href="/procedures/depannage-courant">
    Base de connaissances des pannes résolues (droits Linux ACL, sockets containerd, double routeur Traefik).
  </Card>
</CardGroup>

---

## 🔑 3. Gestion des Utilisateurs & Authentification

<CardGroup cols={2}>
  <Card title="Inviter un Utilisateur (Authentik)" icon="user-plus" href="/services/authentik#exploitation--procedures-inviter-un-utilisateur">
    Générer un lien d'invitation à usage unique dans Authentik et affecter les rôles RBAC automatiquement.
  </Card>
  <Card title="Gestion des Utilisateurs & Tokens Ntfy" icon="bell" href="/services/ntfy#exploitation--procedures-cli">
    Commandes CLI Docker pour créer des comptes Ntfy, accorder des ACLs sur le topic `ims-alerts` et générer des tokens API.
  </Card>
  <Card title="Vérification Forward-Auth Dozzle" icon="list" href="/services/dozzle#securite-authentik-forward-auth">
    Procédure de contrôle du middleware d'authentification Traefik Outpost sur l'interface de logs Dozzle.
  </Card>
</CardGroup>

---

## 📊 4. Métrologie, Monitoring & Télémétrie

<CardGroup cols={2}>
  <Card title="Ajouter un Nouvel Hôte à Surveiller (Alloy)" icon="plus" href="/services/monitoring#exploitation--procedures">
    Installation d'Alloy en binaire systemd, configuration du push Remote-Write (`10.10.10.2`/`192.168.1.52`) et validation.
  </Card>
  <Card title="Créer une Règle d'Alerte Grafana → Ntfy" icon="bell" href="/services/monitoring#exploitation--procedures">
    Création de requêtes PromQL instantanées, réglage du *No data state* à Normal et association du Contact Point Ntfy.
  </Card>
</CardGroup>

---

## 🎬 5. Exploitation Stack Média (HomeFlix)

<CardGroup cols={2}>
  <Card title="Port Forwarding ProtonVPN (Gluetun)" icon="network-wired" href="/services/homeflix#exploitation--procedures-protonvpn--hardlinks">
    Extraire le port attribué dynamiquement par ProtonVPN (`/tmp/gluetun/forwarded_port`) et le mettre à jour dans qBittorrent.
  </Card>
  <Card title="Audit & Gestion des Hardlinks (`rdfind`)" icon="link" href="/services/homeflix#exploitation--procedures-protonvpn--hardlinks">
    Détecter les vrais doublons de fichiers sur le NAS NFS et valider l'occupation disque réelle avec `df -h`.
  </Card>
  <Card title="Scan & Nettoyage des Orphelins (Hardlinks)" icon="broom" href="/services/homeflix#detection--nettoyage-des-orphelins-downloads-audit--script-inodes">
    Script Bash de comparaison d'inodes réels sur `/mnt/disk1` pour éliminer les doublons de téléchargements.
  </Card>
  <Card title="Résolution Import Manuel (Unable to parse file)" icon="file-circle-exclamation" href="/services/homeflix#resolution-des-imports-manuels-bloques-unable-to-parse-file">
    Procédure d'attribution manuelle de qualité dans Radarr/Sonarr pour débloquer l'importation de releases génériques.
  </Card>
</CardGroup>

---

## 🛠️ 6. Outillages, Partage & Développement

<CardGroup cols={2}>
  <Card title="Miroirs de Sauvegarde GitHub (Forgejo)" icon="code-branch" href="/services/forgejo#2-depots-miroirs-de-sauvegarde-pull-mirrors">
    Procédure de renouvellement des jetons d'accès PAT GitHub dans `Settings → Mirror Settings` pour réactiver la synchro des dépôts.
  </Card>
  <Card title="Partage de Fichiers & ShareX (Zipline)" icon="share-nodes" href="/services/zipline#securite--authentification">
    Configuration des jetons d'accès API Zipline pour les téléversements automatisés depuis les clients lourds.
  </Card>
  <Card title="Déploiement Continu Git (Patrimo)" icon="chart-pie" href="/services/patrimo#fiche-service">
    Fonctionnement du pipeline CI/CD Webhook Coolify App et gestion des volumes nommés Docker.
  </Card>
  <Card title="Traitement PDF & Sécurité (Stirling PDF)" icon="file-pdf" href="/services/stirling-pdf#securite--authentification">
    Gestion du mode stateless et du contrôle d'accès Authentik Outpost sans double mire de connexion.
  </Card>
  <Card title="Maintenance & Dump MariaDB (PhotoPrism)" icon="database" href="/services/photoprism#acces-rapides--administration">
    Export et restauration SQL de la base de données PhotoPrism via `mariadb-dump` sur volume nommé local.
  </Card>
  <Card title="Ingestion & Import WebDAV (PhotoPrism)" icon="folder-cloud" href="/services/photoprism#ingestion-de-photos--webdav">
    Dépôt direct de fichiers RAW via WebDAV (`/import/`), gestion des identifiants et commande d'importation forcée.
  </Card>
  <Card title="Purger un Bannissement IP Home Assistant" icon="house-signal" href="/services/home-assistant#reverse-proxy--traefik">
    Procédure de déblocage et suppression du fichier `ip_bans.yaml` en cas d'auto-ban provoqué par le hairpin NAT local.
  </Card>
</CardGroup>
