---
title: "Comment fonctionne cette documentation"
description: "Principes d'organisation, règles de contribution et structure de docs 2"
---

## Philosophie et Organisation

<Info>
Cette documentation regroupe la vérité terrain opérationnelle du homelab IMS-WORLD. Elle documente le **quoi, pourquoi et comment opérer**, en complément du code versionné.
</Info>

## Structure du projet `docs 2`

| Section | Rôle | Emplacement |
|---|---|---|
| **Vue d'ensemble** | Architecture globale et état actuel de l'infrastructure | `README.md` |
| **Infrastructure** | Fiches matérielles et virtuelles (PVE MS-01, Labrax, NAS, PBS, Coolify) | `infrastructure/` |
| **Réseau** | Bridges Proxmox, Traefik Reverse Proxy, DNS-01 OVH, Tailnet VPN | `reseau/` |
| **Services** | Fiches des applications en production (Authentik, Vaultwarden, HomeFlix, Headscale...) | `services/` |
| **Procédures** | Guides opératoires pas-à-pas (Migration, Dépannage courant, Ajout de disques) | `procedures/` |
| **Historique** | Chronologie des jalons et journal des modifications | `CHANGELOG.md` |

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
