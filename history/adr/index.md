---
title: "Décisions d'architecture"
description: "Index des ADR : chaque décision structurante du homelab, son statut et sa date."
icon: "scale-balanced"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Un ADR (Architecture Decision Record) consigne une décision, son contexte et les options écartées. Il est figé une fois accepté : une décision qui évolue est amendée par un bloc daté, ou remplacée par un nouvel ADR. Les règles sont décrites dans le [guide de style](/reference/contribuer#historique).

| ADR | Décision | Statut | Date |
| --- | --- | --- | --- |
| [001](/history/adr/adr-001-stack-monitoring-lgtm) | Supervision par la pile LGTM, Dozzle conservé pour les journaux en direct | Accepté | 12/08/2026 |
| [002](/history/adr/adr-002-pattern-forward-auth-authentik) | Forward-auth Authentik pour les applications sans authentification | Accepté | 12/08/2026 |
| [003](/history/adr/adr-003-partitionnement-stockage-nfs) | Stockage NFS en deux niveaux : `storage` et `storage-hot` | Accepté, amendé le 23/10/2026 | 12/08/2026 |
| [004](/history/adr/adr-004-isolation-reseau-trois-zones) | Réseau en trois zones d'exposition | Accepté, amendé le 23/08/2026 | 12/08/2026 |
| [005](/history/adr/adr-005-cpu-vm-coolify-mode-host) | Processeur de la VM 104 en mode `host` | Accepté | 14/08/2026 |
| [006](/history/adr/adr-006-exposition-port-ssh-forgejo-bbox) | Port 2222 ouvert sur la Bbox pour Forgejo | Accepté, amendé le 23/08/2026 | 14/08/2026 |
| [007](/history/adr/adr-007-calcul-inodes-mergerfs-path-hash) | Calcul d'inodes MergerFS et diagnostic des liens physiques | Remplacé par l'ADR-012 | 16/08/2026 |
| [008](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe) | Passthrough de l'iGPU vers la VM 104 | Accepté | 17/08/2026 |
| [009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only) | Services d'administration restreints au tailnet (`vpn-only`, `userland-proxy`) | Accepté | 18/08/2026 |
| [010](/history/adr/adr-010-maintien-port-ssh-22-et-hostname-cluster) | Port SSH 22 et nom d'hôte `pve` conservés dans le cluster | Accepté | 23/08/2026 |
| [011](/history/adr/adr-011-organisation-ims-infra-par-lots) | Organisation du dépôt IMS-INFRA par lots | Remplacé par l'ADR-013 | 19/10/2026 |
| [012](/history/adr/adr-012-retrait-de-mergerfs) | Retrait de MergerFS : bind-mounts ext4, aucun FUSE sous NFS | Accepté | 23/10/2026 |
| [013](/history/adr/adr-013-administration-par-ims-ops) | Administration du parc par ims-ops (Ansible, just) | Accepté | 23/10/2026 |
| [014](/history/adr/adr-014-sauvegarde-par-pool-unique) | Sauvegarde par un pool unique, alertes sur Ntfy | Accepté | 24/10/2026 |
