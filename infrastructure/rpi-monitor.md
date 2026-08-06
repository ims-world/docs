---
title: "Raspberry Pi 3B+ & Écrans (Kiosk)"
description: "Affichage Kiosk physique du rack Labrax — Écran Wisecoco 7.84\" + OLED 0.91\""
icon: "desktop"
iconType: "duotone"
---

import { ips } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active (Afficheur Kiosk 2U)</Badge>

## Rôle & Fonctionnement Réel

<Info>
Le **Raspberry Pi 3B+** est une unité d'affichage physique dédiée. Contrairement à une sonde de monitoring actif (pas de healthchecks ICMP/HTTP ni d'alertes Ntfy configurés actuellement), son rôle est de piloter l'affichage visuel en façade du rack [Labrax](/infrastructure/labrax) via un **navigateur web headless en mode kiosk**.
</Info>

### Contrôle Physique par Bouton Poussoir
Un bouton poussoir raccordé aux GPIOs du Raspberry Pi permet d'interagir directement sans clavier :
- 🔘 **Appui court** : Basculer vers la source d'affichage suivante (rotation).
- ⏹️ **Appui long (3 secondes)** : Éteindre l'écran principal.

---

## Conception 3D & Matériel du Module 2U

<Card title="Module 3D MakerWorld — Raspberry Pi 2U" icon="cube" href="https://makerworld.com/fr/models/2233953-screen-module-for-10-inch-rack-raspberry-pi-2u#profileId-2521672">
  Module d'intégration 2U personnalisé pour rack 10 pouces (**Screen module for 10-inch rack - Raspberry Pi 2U**). Héberge les 2 écrans, le bouton poussoir de contrôle, 2 ports RJ45 Keystones et le Raspberry Pi 3B+.
</Card>

<Tabs>
  <Tab title="🖼️ Face Principale (Façade 2U & Vitre)">
    <Frame caption="Façade 2U avec vitre encastrée, écran Wisecoco 7.84 et OLED 0.91">
      ![Module 2U Raspberry Pi & Écran — Façade](/assets/rpi-module-front.png)
    </Frame>
  </Tab>
  <Tab title="📐 Vue Isométrique 3D & Intérieur Tray">
    <Frame caption="Vue isométrique 3D montrant l'intégration du Raspberry Pi 3B+ et du câblage interne">
      ![Module 2U Raspberry Pi & Écran — Vue 3D Intérieure](/assets/rpi-module-iso.png)
    </Frame>
  </Tab>
</Tabs>

## Fiche Technique des Écrans

| Composant / Écran | Spécifications | Usage Actuel & Prévu |
|---|---|---|
| **Écran Principal** | **Wisecoco 7.84 pouces LCD** (Résolution `1280x400`) | Rotation entre bannière PNG IMS et sites web en mode lisibilité. *(Prévu : Dashboard Grafana)* |
| **Écran Secondaire** | **Module OLED 0.91 pouces** (`128x32`, affichage blanc/bleu) | Affichage direct des métriques de statut et uptime. |
| **Carte Mère** | Raspberry Pi 3B+ (Broadcom BCM2837B0 quad-core 1.4GHz, 1 Go RAM) | Exécute le script de rotation et le navigateur Chromium headless. |
| **Bouton de Contrôle** | Bouton poussoir GPIO avec LED | Appui court = Change source / Appui long 3s = Éteint l'écran. |
| **Réseau & Tailnet** | Ethernet RJ45 + Client Tailscale (`{ips.rpi}`) | Hostname Tailnet: `ims-rpi-monitor` |
| **Statut** | <Badge color="green">🟢 Production</Badge> | Affichage Kiosk |

---

## Schéma d'Architecture d'Affichage Kiosk

<Frame caption="Architecture logicielle de rotation et contrôle GPIO du Raspberry Pi Kiosk">
```mermaid
graph TD
    subgraph SENSOR ["🍓 Raspberry Pi 3B+ (Module 2U MakerWorld — 100.64.0.12)"]
        SCRIPT["Script Python / Bash (Gestionnaire de Kiosk)"]
        BROWSER["Chromium Headless (Mode Kiosk)"]
        GPIO["Gestionnaire GPIO (Bouton Poussoir)"]
    end

    subgraph SOURCES ["🖼️ Sources d'Affichage en Rotation"]
        IMG_LOCAL["Bannière PNG Local IMS-WORLD"]
        READABILITY["Pages Web en mode Lisibilité"]
        GRAFANA["Dashboard Grafana (Objectif Futur)"]
    end

    subgraph DISPLAYS ["🖥️ Écrans Physique en Façade 2U"]
        MAIN_DISP["Écran Principal Wisecoco 7.84'' LCD (1280x400)"]
        OLED_DISP["Écran Secondaire OLED 0.91'' (128x32 Blanc/Bleu)"]
    end

    GPIO -->|Appui Court: Change Source\nAppui Long 3s: Power Off| SCRIPT
    SCRIPT --> SOURCES
    SOURCES --> BROWSER
    BROWSER --> MAIN_DISP
    SCRIPT -->|Uptime & Statut| OLED_DISP

    classDef rpi fill:#F97316,stroke:#FB923C,color:#fff;
    classDef disp fill:#2c3e50,stroke:#34495e,color:#fff;
    class SCRIPT,BROWSER,GPIO rpi;
    class MAIN_DISP,OLED_DISP disp;
```
</Frame>
