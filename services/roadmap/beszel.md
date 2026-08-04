---
title: "Beszel (Feuille de Route 2026)"
description: "Monitoring léger CPU/RAM/Disque multi-hôtes et alertes de télémétrie"
icon: "chart-line"
iconType: "duotone"
---

<Info>
⏳ **Statut : Planifié (Feuille de Route 2026)** — Solution de monitoring ultra-léger et moderne (agent Rust/Go + dashboard Web) pour suivre la charge du MS-01, du Mac Mini et des conteneurs Docker.
</Info>

## Fiche Technique Prévisionnelle

| Propriété | Valeur Projetée |
|---|---|
| **Domaine Visé** | `status.ims-world.fr` |
| **Composants** | `beszel-hub` (Hub central) + `beszel-agent` (Sondes sur chaque hôte) |
| **Consommation** | < 15 MB RAM par agent |
| **Accès** | Restricted **Tailnet-only** via middleware `vpn-only` |
| **Statut** | ⏳ Feuille de Route |

## Architecture de Monitoring Multi-Hôtes

```mermaid
graph TD
    subgraph HUB ["📊 Beszel Hub Central (VM 104)"]
        DASH["Beszel Dashboard (status.ims-world.fr)"]
        ALERT_SYS["Notification Engine (Ntfy / Discord)"]
    end

    subgraph AGENTS ["📡 Sondes Beszel Agent (Multi-Nœuds)"]
        A_MS01["Agent Host Proxmox MS-01 (192.168.1.41)"]
        A_VM["Agent VM Coolify (100.64.0.10)"]
        A_MAC["Agent Mac Mini Standby (100.64.0.7)"]
        A_RPI["Agent Raspberry Pi 3B+ (192.168.1.x)"]
    end

    A_MS01 -->|Métrique CPU/RAM/Temp/Disque| DASH
    A_VM -->|Métrique Docker Stats| DASH
    A_MAC -->|Métrique Charge Standby| DASH
    A_RPI -->|Métrique Monitoring Hors-Bande| DASH

    DASH -->|Alerte Seuil Dépassé| ALERT_SYS
```
