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

### 4. Definition of Done (DoD) IMS-WORLD
<Warning>
**Règle d'or de gouvernance** : Aucune modification de conteneur, de sous-domaine, de volume NFS ou de règle de pare-feu n'est considérée comme terminée tant que la documentation n'est pas mise à jour.
</Warning>

Une Pull Request / un commit est valide uniquement si la checklist suivante est complétée :
- [ ] La **Fiche Service** (`services/*.md`) ou **Infra** est à jour (avec `last_reviewed` et `app_version`).
- [ ] L'UUID et le chemin sur la VM sont inscrits dans [vm-coolify.md](/infrastructure/vm-coolify).
- [ ] La zone de confiance est déclarée dans [matrice-securite-exposition.md](/reseau/matrice-securite-exposition).
- [ ] Les commandes CLI sont validées selon le principe **Command-Paste Safety** (100% exécutables par simple copier-coller).
- [ ] La commande `mintlify broken-links` s'exécute avec **0 lien mort**.

### 5. Stratégie de Versionnement & Fraîcheur (`last_reviewed`)
- **Frontmatter YAML obligatoire** : Chaque fiche service ou hôte possède un paramètre `last_reviewed` (date ISO YYYY-MM-DD) et `app_version` (tag d'image Docker figé).
- **Revue trimestrielle** : Toute fiche dont la date `last_reviewed` dépasse 6 mois doit faire l'objet d'un audit de conformité avec les conteneurs réellement en cours d'exécution sur le serveur MS-01.

