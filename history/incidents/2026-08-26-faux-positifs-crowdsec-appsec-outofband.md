---
title: "Incident — Faux Positifs AppSec WAF en Cascade (Grafana, Jellyfin, Patrimo)"
description: "Neutralisation des bannissements automatiques sur le scénario crowdsec-appsec-outofband via tuning de profiles.yaml suite à de faux positifs CRS répétés"
icon: "shield-exclamation"
iconType: "duotone"
---

<Badge color="amber">🟡 Fix Appliqué — Profil de Neutralisation des Bans Out-of-Band en Phase d'Observation</Badge> *(2026-08-26)*

*Rédigé le 26/08/2026 — Périmètre CrowdSec v1.7.8 & AppSec WAF.*

---

## Symptôme Initial

Du 25/08 au 26/08/2026, **quatre bannissements successifs** d'adresses IP sur du trafic 100% légitime ont été déclenchés par le moteur WAF CrowdSec AppSec :
- Blocages intempestifs lors de la navigation sur **Grafana** (`monitoring.ims-world.fr`), **Patrimo** (`patrimo.ims-world.fr`), **HomeFlix / Jellyfin** (`homeflix.ims-world.fr`) et Zabbix.
- Chaque bannissement était provoqué par une règle OWASP CRS différente (ex: règle `920420` sur le header `Content-Type`, règles sur les requêtes AJAX de suivi de progression vidéo Jellyfin, etc.).
- **Point commun** : Tous les bans étaient instanciés par le même scénario d'accumulation **`crowdsecurity/crowdsec-appsec-outofband`**, devenu trop sensible pour un parc applicatif effectuant de nombreux appels API/AJAX en arrière-plan.

---

## Fausses Pistes & Première Tentative d'Ajustement

1. **Exclusion règle par règle (CRS 920420)** : Une première tentative consistait à désactiver spécifiquement la règle CRS `920420` dans un fichier de configuration custom `appsec-config`.
2. **Limite de l'approche** : Très rapidement, d'autres règles CRS légitimes pour du WAF web généraient de nouveaux faux positifs sur les appels AJAX de Jellyfin ou Grafana. Traiter les faux positifs règle par règle s'avérait sans fin et risquait de trouer la couverture WAF globale.

---

## Cause Racine Identifiée

Le scénario **`crowdsecurity/crowdsec-appsec-outofband`** écoute l'analyse hors-bande (asynchrone) des logs WAF. Lorsqu'une session accumule plusieurs alertes CRS de faible gravité (bruit d'arrière-plan d'applications web modernes très réactives), ce scénario déclenche un **bannissement automatique (Remediation: ban)** d'une durée de 1h à plusieurs jours.

---

## Fix Appliqué & Stratégie de Tuning

Plutôt que de désactiver aveuglément les règles OWASP CRS une à une, la stratégie retient un **ajustement du profil d'action (`profiles.yaml`)** :

1. **Profil de neutralisation en tête de `/etc/crowdsec/profiles.yaml`** :
   Ajout d'un profil prioritaire capturant le scénario `crowdsecurity/crowdsec-appsec-outofband` avec l'instruction `on_success: continue` et sans directive de remédiation par bannissement.

```yaml
# /etc/crowdsec/profiles.yaml (Extrait du profil outofband)
name: ignore_appsec_outofband_bans
filters:
  - Alert.GetScenario() == 'crowdsecurity/crowdsec-appsec-outofband'
on_success: continue
# Aucune remédiation 'ban' configurée : l'alerte est journalisée mais aucun ban n'est émis
```

---

## Impact & Périmètre de Protection Préservé

- **Les alertes restent enregistrées & visibles** : L'ensemble des détections AppSec hors-bande reste parfaitement logué et consultable dans la console **Shield** (`shield.ims-world.fr`) et les dashboards Grafana.
- **Aucune perte de protection sur les vraies menaces** :
  - Les attaques de **Bruteforce** (SSH, HTTP, Authentik) continuent de bannir automatiquement.
  - Les tentatives de **Scans agressifs** et vulnérabilités ciblées restent bannies.
  - Les règles **Inband bloquantes** (AppSec WAF en temps réel) continuent de rejeter les requêtes malveillantes en HTTP 403.

---

## Statut & Validation Post-Fix

- **Statut** : Profil appliqué et rechargé (`systemctl reload crowdsec`) le 26/08/2026.
- **Phase d'observation** : En attente de confirmation après plusieurs jours d'utilisation normale prolongée sur Jellyfin, Grafana et les applications interactives du homelab.
