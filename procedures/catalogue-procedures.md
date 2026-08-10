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
  <Card title="Plan de Reprise d'Activité (PRA)" icon="triangle-exclamation" href="/procedures/plan-reprise-activite-pra">
    Procédure étape par étape pour reconstruire l'infrastructure complète en cas de sinistre physique majeur.
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
</CardGroup>
