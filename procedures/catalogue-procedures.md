---
title: "Catalogue des Procédures Opérationnelles"
description: "Index centralisé de toutes les procédures d'administration, d'exploitation, de sécurité et de PRA"
icon: "book-bookmark"
iconType: "duotone"
last_reviewed: "2026-08-24"
---

import { ips, domains } from "/snippets/variables.mdx";

<Info>
Ce catalogue répertorie **l'ensemble des procédures opératoires du homelab IMS-WORLD**. Les procédures sont structurées en 4 catégories et sont également directement accessibles depuis le menu latéral *Procédures & PRA*.
</Info>

---

## ⚡ 1. Urgence & Diagnostic

<CardGroup cols={2}>
  <Card title="Commandes d'Urgence Break-Glass" icon="bolt" href="/procedures/commandes-urgence">
    Aide-mémoire des commandes de secours SSH, déblocage des accès et redémarrage des composants critiques.
  </Card>
  <Card title="Dépannage Courant & Pièges Vécus" icon="wrench" href="/procedures/depannage-courant">
    Base de connaissances des pannes résolues (droits Linux ACL, sockets containerd, double routeur Traefik).
  </Card>
  <Card title="Clé de Récupération Authentik" icon="key" href="/services/authentik#acces-rapides--administration">
    Générer une clé administrateur d'urgence à usage unique en cas de perte du WebAuthn 2FA ou du mot de passe admin.
  </Card>
  <Card title="Administrer & Débannir avec CrowdSec (cscli / Shield)" icon="shield-check" href="/services/crowdsec#acces-rapides--administration">
    Commandes CLI `cscli` pour gérer les décisions de ban, vérifier les allowlists anti-auto-ban et accéder à l'UI Shield.
  </Card>
</CardGroup>

---

## ⚙️ 2. Exploitation & Déploiement

<CardGroup cols={2}>
  <Card title="Déploiement d'un Nouveau Service Coolify" icon="cube" href="/procedures/deploiement-service">
    Check-list de déploiement Docker Compose, attribution d'UUID, configuration du réseau et masquage DNS.
  </Card>
  <Card title="Sauvegarde Manuelle de Coolify" icon="cube" href="/procedures/sauvegarde-manuelle-coolify">
    Guide de sauvegarde à chaud via Proxmox NVMe/PBS, dumps SQL applicatifs et résolution du piège PBS namespace not found.
  </Card>
  <Card title="Ajout d'un Nouveau Disque (LVM / NFS)" icon="hard-drive" href="/procedures/ajout-nouveau-disque">
    Guide pas-à-pas pour formater, monter et étendre un volume de stockage sur Proxmox VE, NAS ou VM Coolify.
  </Card>
  <Card title="Backup & Restauration Vaultwarden" icon="shield-halved" href="/services/vaultwarden#acces-rapides--administration">
    Procédure d'arrêt à chaud et d'archivage sécurisé des fichiers WAL du coffre-fort SQLite Vaultwarden.
  </Card>
  <Card title="Ajouter un Hôte au Monitoring (Alloy)" icon="plus" href="/services/monitoring#exploitation--procedures">
    Installation d'Alloy en binaire systemd, configuration du push Remote-Write (`10.10.10.2`) et validation Grafana.
  </Card>
  <Card title="Créer une Règle d'Alerte Grafana → Ntfy" icon="bell" href="/services/monitoring#exploitation--procedures">
    Création de requêtes PromQL instantanées, réglage du *No data state* à Normal et association du Contact Point Ntfy.
  </Card>
</CardGroup>

---

## 🔒 3. Sécurité & Authentification SSO

<CardGroup cols={2}>
  <Card title="Sécuriser un Service avec vpn-only" icon="shield-check" href="/procedures/securiser-service-vpn-only">
    Procédure pas-à-pas pour isoler un sous-domaine d'administration sur le réseau privé Tailscale via vpn-only.yaml.
  </Card>
  <Card title="Sécuriser une App avec Authentik Outpost" icon="shield-keyhole" href="/procedures/securiser-application-authentik-forward-auth">
    Procédure complète pour protéger une application web sans SSO natif via Traefik et l'Outpost Proxy Authentik.
  </Card>
  <Card title="Intégrer un Service avec Authentik OIDC SSO" icon="key" href="/procedures/integration-service-authentik-oidc">
    Guide pas-à-pas et dictionnaire complet des URLs de redirection (Redirect URIs / Callbacks) par application.
  </Card>
  <Card title="Ajouter un Équipement sur Headscale" icon="network-wired" href="/procedures/ajout-machine-headscale">
    Procédure d'enrôlement VPN Tailnet : Profil Utilisateur Classique (OIDC SSO) vs Profil Nœud Infrastructure (Pre-Auth Key).
  </Card>
  <Card title="Inviter un Utilisateur (Authentik)" icon="user-plus" href="/services/authentik#exploitation--procedures-inviter-un-utilisateur">
    Générer un lien d'invitation à usage unique dans Authentik et affecter les rôles RBAC automatiquement.
  </Card>
  <Card title="Gestion des Utilisateurs & Tokens Ntfy" icon="bell" href="/services/ntfy#exploitation--procedures-cli">
    Commandes CLI Docker pour créer des comptes Ntfy, accorder des ACLs sur le topic `ims-alerts` et générer des tokens API.
  </Card>
</CardGroup>

---

## 🚨 4. Reprise d'Activité (PRA / DRP)

<CardGroup cols={2}>
  <Card title="Plan de Reprise d'Activité (PRA / DRP)" icon="shield-virus" href="/procedures/plan-reprise-activite-pra">
    Stratégie globale de résilience, matrice RTO/RPO et procédures de basculement de secours.
  </Card>
  <Card title="Simulation Crash NVMe & Restauration DRP" icon="skull-crossbones" href="/procedures/simulation-crash-restauration">
    Runbook théorique de restauration à froid d'urgence de la VM Coolify depuis PBS suite à un crash NVMe (avec avertissement).
  </Card>
  <Card title="Politique de Sauvegarde & Tâches Planifiées" icon="shield-check" href="/infrastructure/politique-sauvegardes">
    Vue d'ensemble de la protection des données, chronologie nocturne, règle d'anti-circularité et backups vzdump local.
  </Card>
</CardGroup>
