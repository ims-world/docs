---
title: "Raspberry Pi 3B+ & Écrans"
description: "Sonde de monitoring physique indépendante et affichage métrologique du rack Labrax"
icon: "desktop"
iconType: "duotone"
---

<Note>
🍓 **Type d'Instance** : **Sonde Physique Indépendante** (Raspberry Pi 3B+ ARMv8) — Système de monitoring externe fonctionnant hors de l'hyperviseur Proxmox.
</Note>

## Rôle & Emplacement

<Info>
Le **Raspberry Pi 3B+** est une sonde de surveillance autonome intégrée au rack [Labrax](/infrastructure/labrax). Son rôle est d'assurer un **monitoring matériel hors-bande** et de piloter le petit écran de statut LCD/OLED encastré en façade du rack.
</Info>

## Fiche Technique

| Propriété | Valeur |
|---|---|
| **Matériel** | Raspberry Pi 3B+ (Broadcom BCM2837B0 quad-core 1.4GHz) |
| **Mémoire RAM** | 1 Go LPDDR2 |
| **OS** | Raspberry Pi OS Lite (64-bit) |
| **Réseau LAN** | Ethernet RJ45 `192.168.1.x` (100Mbps) |
| **Affichage** | Écran LCD de statut frontel du rack [Labrax](/infrastructure/labrax) |
| **Alimentation** | Micro-USB via le bus 5V interne / PicoPSU |
| **Statut** | 🟢 Production (Actif) |

## Fonctionnalités & Avantages Hors-Bande

<CardGroup cols={2}>
  <Card title="Monitoring Hors-Bande" icon="heart-pulse">
    Fonctionne de façon 100% indépendante du serveur principal MS-01. Continue de surveiller le réseau et d'alerter même si Proxmox ou le NAS est éteint/en reboot.
  </Card>
  <Card title="Pilote d'Écran Frontal" icon="display">
    Pilote l'affichage physique du logo **IMS** et les télémétries de température / charge du rack [Labrax](/infrastructure/labrax).
  </Card>
</CardGroup>

## Schéma d'Indépendance du Monitoring

```mermaid
graph TD
    subgraph SENSOR ["🍓 Raspberry Pi 3B+ (Sonde Hors-Bande)"]
        MON["Agent Monitoring & Ping Probe"]
        DISP["Contrôleur Écran Status (Façade Labrax)"]
    end

    subgraph PROXMOX_HOST ["🖥️ Cluster MS-01 (Sous Surveillance)"]
        PVE["Proxmox VE 9.2.3"]
        VM104["VM Coolify (104)"]
        NAS["LXC NAS (100)"]
    end

    subgraph NETWORK ["🌐 Réseau & Alerting"]
        SWITCH["Switch NETGEAR GS308EV4"]
        ALERT["Alertes Ntfy / Push notification"]
    end

    MON -->|Ping ICMP & HTTP Healthchecks| PVE
    MON -->|Healthchecks| VM104
    MON -->|Healthchecks| NAS
    MON -->|Métrologie| DISP
    MON -->|Envoi Alerte en cas de Panne| ALERT

    classDef rpi fill:#F97316,stroke:#FB923C,color:#fff;
    classDef pve fill:#2c3e50,stroke:#34495e,color:#fff;
    class MON,DISP rpi;
    class PVE,VM104,NAS pve;
```

## Intégration dans le Rack Labrax

<Tip>
L'écran piloté par le Raspberry Pi est directement intégré au panneau supérieur du rack [Labrax](/infrastructure/labrax), sous le ventilateur Noctua Industrial PPC, et affiche l'état d'activité du cluster en temps réel.
</Tip>
