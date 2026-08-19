---
title: "Incident — Perte du Périphérique /dev/dri post-reboot (Noyau Ubuntu)"
description: "Désynchronisation entre linux-image et linux-modules-extra post-unattended-upgrades et résolution par le métapaquet générique"
icon: "triangle-exclamation"
iconType: "duotone"
---

<Badge color="green">🟢 Résolu & Fix Préventif Appliqué</Badge> *(2026-08-19)*

---

## Symptômes

Après un redémarrage complet de l'infrastructure homelab, les services **HomeFlix (Jellyfin)** et **Sonarr** ont échoué au démarrage sur la VM IMS-Coolify avec le message d'erreur Docker suivant dans l'IHM Coolify et en CLI :

```text
Error response from daemon: error gathering device information while adding custom device "/dev/dri": no such file or directory
```

---

## Cause Racine (Root Cause)

Le module noyau **`i915`** (pilote d'affichage et de transcodage matériel de l'iGPU Intel Iris Xe) était **absent** pour la version du noyau sur laquelle la VM venait de démarrer (`6.8.0-138-generic`).

Sur les images d'installation minimales Ubuntu Cloud (24.04 LTS) :
- Le noyau de base (`linux-image-<version>`) et les modules additionnels non-essentiels (`linux-modules-extra-<version>`, qui contient `i915`) sont **deux paquets séparés**.
- Lors de l'installation initiale ou d'un fix précédent, le paquet `linux-modules-extra-6.8.0-137-generic` avait été installé "à la version" spécifique (de manière figée), sans installer le **métapaquet `linux-modules-extra-generic`**.
- Les mises à jour automatiques en arrière-plan (`unattended-upgrades`) ont progressivement téléchargé et installé les nouvelles images de noyau (`6.8.0-138-generic`) **sans tirer automatiquement le paquet `modules-extra` correspondant**.
- Tant que la VM ne redémarrait pas, elle continuait de s'exécuter sur le noyau `137` chargé en RAM. Lors du redémarrage complet du 19/08/2026, l'hyperviseur KVM a démarré la VM sur le dernier noyau installé (`138`), qui ne disposait pas de `i915.ko` → `/dev/dri/` n'a pas été créé au boot.

---

## Arbre de Diagnostic Pas-à-Pas

L'inspection s'effectue de haut en bas (Hôte Proxmox → VM → Driver Noyau) :

```bash
# 1. Hôte Proxmox (MS-01) — Le GPU est-il toujours en passthrough VFIO ?
lspci -nnk -s 00:02.0
cat /proc/cmdline | grep -o "intel_iommu=[a-z]*"
# Confirme : Kernel driver in use: vfio-pci

# 2. VM Coolify — Le périphérique PCI est-il vu par la VM ?
lspci -nnk | grep -A3 -i "VGA\|Display"
# Confirme : Le périphérique VGA/Display Intel 8086:4626 est bien présent sur le bus PCI.

# 3. VM Coolify — Le module i915 est-il présent/chargé ?
lsmod | grep -E "i915|xe"
uname -r
find /lib/modules/$(uname -r) -iname "i915*" -o -iname "xe.ko*"
# Résultat : Aucun fichier i915.ko trouvé dans /lib/modules/6.8.0-138-generic/

# 4. Confirmer l'écart entre linux-image et linux-modules-extra
dpkg -l | grep "linux-image-$(uname -r)"
dpkg -l | grep "linux-modules-extra-$(uname -r)"
# Résultat : linux-image-6.8.0-138-generic est présent, mais pas linux-modules-extra-6.8.0-138-generic.
```

---

## Action de Rétablissement à Chaud (Fix Réalisé)

```bash
# Installation du paquet extra pour la version courante
sudo apt update
sudo apt install -y linux-modules-extra-$(uname -r)

# Chargement dynamique du module dans le noyau en cours d'exécution
sudo modprobe i915

# Vérification du démarrage du composant DRM
ls -la /dev/dri
# Affiche : card0, renderD128

# Relance des conteneurs impactés
cd /data/coolify/services/w39uebmcnse7yctsft8hzed8
docker compose up -d
```

<Check>
Aucun redémarrage de la VM n'a été nécessaire. Dès que `modprobe i915` a été exécuté, `/dev/dri/renderD128` a réapparu et Docker a pu démarrer Jellyfin avec QuickSync QSV opérationnel.
</Check>

---

## Correctif Préventif (Prévention des Récurrences)

Pour éviter que les futures mises à jour automatiques du noyau par `unattended-upgrades` ne provoquent la même dérive silencieuse, le **métapaquet générique** a été installé :

```bash
# Permet au paquet linux-modules-extra de suivre automatiquement les mises à jour du noyau
sudo apt install -y linux-modules-extra-generic

# Nettoyage des noyaux obsolètes accumulés (124/134/136/137)
sudo apt autoremove --purge
```

---

## Impact sur les Fiches de Documentation

- **[ADR-008 — Passthrough GPU](/history/adr/adr-008-passthrough-gpu-igpu-iris-xe)** : Remplacement de l'instruction d'installation manuelle par `linux-modules-extra-generic`.
- **[Dépannage Courant](/procedures/depannage-courant#échec-démarrage-jellyfin---devdri-no-such-file-or-directory)** : Ajout de la procédure de diagnostic rapide.
