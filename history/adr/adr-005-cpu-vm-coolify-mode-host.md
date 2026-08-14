---
title: "ADR-005 — Configuration CPU de la VM Coolify en mode host (x86-64-v2)"
description: "Décision d'exposer le CPU physique Intel i5-12600H à la VM Coolify plutôt que le profil générique Proxmox kvm64"
icon: "microchip"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-14)*

---

## Contexte

Par défaut, Proxmox assigne un type de CPU générique et conservateur (`kvm64`) à ses machines virtuelles afin de maximiser la portabilité et la migration à chaud entre hôtes physiques hétérogènes. Ce profil masque volontairement les jeux d'instructions modernes du processeur physique réel — notamment les extensions du niveau **x86-64-v2** (SSE4.1, SSE4.2, SSSE3, POPCNT).

Certains binaires Node.js modernes (modules natifs C/C++ compilés comme `sharp`, utilisé par plusieurs applications web de la stack) exigent impérativement les instructions `x86-64-v2` et échouent au démarrage si elles ne sont pas exposées par le CPU virtuel.

---

## Décision

Nous avons décidé de basculer le type de CPU de la **VM IMS-Coolify (VMID 104)** du profil générique vers le mode **`host`** :

```bash
# Configuration du CPU host sur l'hyperviseur Proxmox (Host MS-01)
qm set 104 --cpu host
qm reboot 104
```

Le processeur physique réel de l'hôte (Intel Core i5-12600H) est désormais exposé intégralement à la VM Coolify.

---

## Conséquences

### Positives
- **Résolution définitive des échecs de démarrage** liés aux binaires natifs exigeant `x86-64-v2` ou plus (ex: modules d'optimisation d'image Node.js `sharp`), sans aucun contournement applicatif.
- **Performances maximales** : Accès direct à l'ensemble des instructions vectorielles et matérielles du processeur Intel Alder Lake.
- **Conformité aux recommandations Proxmox** : Dans un environnement mono-hôte, le type `host` est la recommandation officielle par défaut.

### Négatives / Contraintes
- **Perte de portabilité de migration à chaud** : La VM ne peut plus être migrée à chaud vers un hôte Proxmox équipé d'une architecture CPU différente.
- **Sans impact en l'état** : L'infrastructure reposant sur un seul hyperviseur principal (Minisforum MS-01), cette contrainte est sans effet. Décision à réévaluer si un second nœud Proxmox intègre le cluster à l'avenir.
