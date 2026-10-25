---
title: "Feuille de route"
description: "Chantiers ouverts par priorité, idées à évaluer, et chantiers réalisés récemment."
icon: "list-check"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Une ligne par chantier. Un chantier terminé passe dans **Réalisés**, avec un lien vers la page qui le décrit ; le détail va dans la [chronologie](/history/chronologie).

## Priorité haute

| Chantier | Pourquoi | Où |
| --- | --- | --- |
| Protéger les données du NAS : photos Immich, dépôts Forgejo, originaux PhotoPrism | Aucune copie aujourd'hui ; perte définitive si un disque lâche | NAS, [politique de sauvegarde](/procedures/politique-sauvegardes#ce-qui-nest-pas-sauvegardé) |
| Copie hors site des sauvegardes PBS | Datastore et médiathèque sur le même HDD. Piste : Raspberry Pi avec disque, en synchronisation PBS | PBS |
| Premier test de restauration | Le plan de reprise n'a jamais été validé | [Plan de reprise](/procedures/plan-reprise-activite-pra#tester-les-sauvegardes) |
| Ajouter la VM 102 au pool `backup` | Windows non sauvegardée (`just backup-add 102`) | Pool `backup` |
| Décider de la sauvegarde de `lab-elo` | VM d'un tiers, hors pool | Pool `backup` |
| QDevice Corosync | Deux votes seulement : la perte d'un nœud fait perdre le quorum | Raspberry Pi |
| Alerte hors du MS-01 | Toute la supervision tourne sur la VM 104 : une panne du MS-01 ne produit aucune alerte | Mac Mini, Raspberry Pi ou VPS |

## Priorité moyenne

| Chantier | Pourquoi | Où |
| --- | --- | --- |
| Utilisateur PBS dédié pour `pbs-coolify` | Le stockage se connecte avec `root@pam` | PBS |
| Revoir les middlewares de chaque routeur de `vpn-only.yaml` | Suite de l'[incident Dozzle](/history/incidents/2026-08-24-bypass-sso-dozzle-traefik-file-provider) | [Traefik](/reseau/traefik-proxy) |
| Contrôle périodique des TXT `_acme-challenge` | Suite de l'[incident DNS](/history/incidents/2026-08-24-dns-wildcard-shadowing-share-ims-world-fr) | Zone OVH |
| Clore l'observation CrowdSec | Ouverte depuis le 26/08/2026 | [CrowdSec](/reseau/crowdsec) |
| Épingler IT-Tools, Stirling PDF, Zipline | Images en `latest` | VM 104 |
| Identifiants OVH du proxy hors du compose | En clair dans la configuration du proxy | [Traefik](/reseau/traefik-proxy) |
| Pare-feu Proxmox | Non activé ; règles par nœud d'abord, console ouverte pendant l'activation | Nœuds |
| Raccorder Windows (VM 102) en RDP dans Termix | Démarrage à la demande à étudier (plugin Proxmox Power) | [Termix](/services/termix) |
| Supprimer l'ancien job de sauvegarde du NAS (`backup-6b92bcb1-ad11`) | Désactivé depuis le 19/08/2026, il n'a plus d'usage | [Politique de sauvegarde](/procedures/politique-sauvegardes) |
| Renouveler la clé Headscale de `infra` | Expire le 18/10/2027 | [Headscale](/reseau/headscale-headplane) |
| Domotique : Hue, HomeKit, téléviseurs, application mobile, clé Zigbee | Intégrations à configurer | [Home Assistant](/infrastructure/vm-home-assistant) |
| Montage de l'extension du rack | Impression terminée | [Labrax](/infrastructure/labrax) |

## À évaluer

| Idée | Note |
| --- | --- |
| Accès Intel vPro / AMT sur le MS-01 | Console matérielle sans sortie vidéo |
| DNS secondaire (AdGuard Home, Pi-hole) | Sur le Mac Mini ou le Raspberry Pi |
| Exporteur Prometheus pour Jellyfin | Sessions, transcodages |
| Caddy à la place de Traefik | Essai : `xcaddy` avec OVH et CrowdSec |
| Portail self-service de VM temporaires (IMS-CLOUD) | Au-dessus de `just vm` et de l'API Proxmox |
| Certificats SSH courts (OPKSSH) | À la place de la clé statique de Termix |
| Second HDD | Disque ext4 indépendant ([procédure](/procedures/ajout-nouveau-disque#option-a-ajouter-un-disque-de-capacité)) |
| Sonde de détection d'intrusion réseau, Sentryx | — |
| Applications : Homepage, Imgcompress, Strix, Semgrep, Dawarich, Keep It Shot, Cap, Cyber Strike AI, Page Agent, Meetily | Une à la fois, selon le besoin |

## Réalisés

| Date | Chantier | Page |
| --- | --- | --- |
| 25/10/2026 | Refonte de la documentation | [Contribuer](/reference/contribuer) |
| 24/10/2026 | Sauvegarde par pool et alertes Ntfy, y compris le worker et Home Assistant | [ADR-014](/history/adr/adr-014-sauvegarde-par-pool-unique) |
| 24/10/2026 | Création et destruction de VM par ims-ops, template 9001 | [Administration avec ims-ops](/procedures/administration-ims-ops) |
| 24/10/2026 | Termix raccordé au parc | [Termix](/services/termix) |
| 24/10/2026 | Sonde Uptime Kuma et bail réservé pour Home Assistant | [Home Assistant](/infrastructure/vm-home-assistant) |
| 23/10/2026 | Socle commun appliqué par ims-ops | [Socle commun](/infrastructure/socle-commun) |
| 23/10/2026 | Retrait de MergerFS | [ADR-012](/history/adr/adr-012-retrait-de-mergerfs) |
| 22/10/2026 | NAS et worker dans le tailnet | [Architecture réseau](/reseau/architecture-reseau) |
| 22/09/2026 | n8n | [n8n](/services/n8n) |
| 14/09/2026 | Home Assistant en VM, WUD | [Home Assistant](/infrastructure/vm-home-assistant), [WUD](/services/whatsupdocker) |
| 13/09/2026 | Enrôlement au tailnet par Authentik | [Headscale](/reseau/headscale-headplane) |
| 23/08/2026 | Cluster à deux nœuds | [Mac Mini](/infrastructure/mac-mini) |
| 22/08/2026 | CrowdSec et pare-feu applicatif | [CrowdSec](/reseau/crowdsec) |

Les chantiers plus anciens sont dans la [chronologie](/history/chronologie).
