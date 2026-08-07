---
title: "Comment fonctionne cette documentation"
description: "Principes d'organisation, règles de contribution et structure de docs 2"
icon: "book-open"
iconType: "duotone"
---

import { ips, domains } from "/snippets/variables.mdx";

## Philosophie et Organisation

<Info>
Cette documentation regroupe la vérité terrain opérationnelle du homelab IMS-WORLD. Elle documente le **quoi, pourquoi et comment opérer**, en complément du code versionné.
</Info>

## Structure du projet `IMS-HOMELAB`

| Section | Rôle | Emplacement |
|---|---|---|
| **Vue d'ensemble** | Architecture globale et état actuel de l'infrastructure | `index.md` |
| **Infrastructure** | Fiches matérielles et virtuelles (PVE MS-01, Labrax, NAS, PBS, Coolify) | `infrastructure/` |
| **Réseau** | Bridges Proxmox, Traefik Reverse Proxy, DNS-01 OVH, Tailnet VPN | `reseau/` |
| **Services** | Fiches des applications en production (Authentik, Vaultwarden, HomeFlix, Headscale...) | `services/` |
| **Procédures** | Guides opératoires pas-à-pas (Migration, Dépannage courant, Ajout de disques) | `procedures/` |
| **Historique** | Chronologie des jalons et journal des modifications | `history/chronologie.md` |

---

## 🧩 Structure Standardisée des Fiches Services (`services/`)

Toute nouvelle fiche applicative ajoutée dans le dossier `services/` **DOIT impérativement respecter la trame universelle en 6 blocs** :

| Ordre | Section MDX | Contenu & Format Attendu |
|---|---|---|
| 1 | `<Badge color="green">🟢 Production Active</Badge>` | Badge de statut en haut de page |
| 2 | `## Accès Rapides & Administration` | `<Tabs>` avec Onglet `🌐 Interfaces Web` (Cards) et Onglet `⚡ Commandes CLI & Maintenance` |
| 3 | `## Fiche Service` | Tableau standardisé (Domaine, Hôte, UUID Coolify, Chemin VM, Technos/DB, Statut) |
| 4 | `## Architecture & Topologie` | Schéma Mermaid SVG d'architecture/flux + avertissement structural si nécessaire |
| 5 | `## Composants & Fonctionnement` | Tableau des conteneurs, sous-modules, groupes RBAC ou clés critiques |
| 6 | `## Stockage & Politique de Sécurité` | Volumes SSD/NAS, contraintes de hardlinks et règles de durcissement (Hardening `SIGNUPS_ALLOWED`, etc.) |
| 7 | `## Exploitation & Procédures` | `<Steps>` ou `<AccordionGroup>` pour les procédures opérationnelles spécifiques (ex: invitations, backup) |

---

## Règles de Rédaction et de Maintenance

### 1. Règle : Documentation vs Code Versionné (`ims-infra`)
- La documentation explique le fonctionnement et les choix d'architecture.
- Le code versionné (Ansible, Compose, Scripts, Configs Traefik) vit dans le dépôt `ims-infra`.
- Si un script ou un fichier compose dépasse quelques lignes, il doit être référencé depuis le code sans être dupliqué intégralement dans la doc.

### 2. Règle : Source Unique de Vérité (Anti-duplication)
- Les informations techniques (UUIDs Coolify, ports, adresses IP, domaines) doivent vivre à **un seul endroit principal** (la fiche service ou la fiche infrastructure).
- Les autres pages référencent cette source via des liens Markdown explicites plutôt que d'en dupliquer le texte.

### 3. Règle : Traçabilité du Vécu et des Pièges
- Chaque problème complexe résolu doit être consigné dans [Dépannage courant](/procedures/depannage-courant) avec son symptôme, sa cause exacte et sa résolution éprouvée.
- Toute évolution majeure d'architecture est consignée dans la [Chronologie du projet](/history/chronologie).

### 4. Stratégie de Versionnement de la Documentation (Git & Mintlify)
- **Version Actuelle (`v1.0`)** : La branche `main` héberge la documentation de référence officielle v1.0 (cutover réussi sur Proxmox MS-01, Coolify, Tailscale et Labrax 10").
- **Évolutions Futures (`v1.2`, `v2.0`)** : Lors d'évolutions d'infrastructure majeures (ex: cluster HA multi-nœuds, migration SAN 10G), créer une branche Git dédiée `v1.2` ou `v2.0` (ou un tag Release) liée au déploiement Mintlify pour maintenir les versions de doc en parallèle.
