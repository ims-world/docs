---
title: "ADR-008 — Passthrough GPU (iGPU Iris Xe) vers la VM Coolify"
description: "Passthrough PCI complet du GPU intégré pour le transcodage matériel Jellyfin et PhotoPrism"
icon: "microchip"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Adopté & Validé</Badge> *(2026-08-17)*

---

## Contexte

Jellyfin (et dans une moindre mesure PhotoPrism) s'exécutent en conteneurs Docker à l'intérieur d'une **VM QEMU/KVM** (VM 104) — et non dans un conteneur LXC. Le passthrough GPU nécessite donc un détachement PCI complet du périphérique via **VFIO**, et non un simple montage de device passthrough (`/dev/dri`) comme ce serait le cas pour un LXC.

Le processeur (Intel Core i5-12600H) ne dispose que d'un seul processeur graphique (l'iGPU Intel Iris Xe intégrée, sans GPU dédié séparé). Un passthrough PCI complet retire l'accès à la console graphique native de l'hôte Proxmox VE lui-même. Ce choix est pleinement assumé : le serveur MS-01 est piloté à 100% à distance via SSH et l'interface Web Proxmox (aucun écran physique n'y est raccordé en usage normal).

---

## Décisions

### 1. Activation IOMMU (Kernel & GRUB)
Le serveur utilise GRUB classique (le fichier `/etc/kernel/proxmox-boot-uuids` est absent, `proxmox-boot-tool` non utilisé). L'activation s'effectue par édition directe de `/etc/default/grub` :

```bash
# Modifier GRUB_CMDLINE_LINUX_DEFAULT dans /etc/default/grub :
# GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"

sudo update-grub
sudo reboot
```

<Check>
VT-d était activé au niveau du BIOS — confirmé par la commande `dmesg | grep DMAR` après redémarrage, sans nécessiter d'accès physique au serveur.
</Check>

### 2. Isolation du Groupement IOMMU
```bash
readlink /sys/bus/pci/devices/0000:00:02.0/iommu_group
ls /sys/kernel/iommu_groups/<n>/devices/
```

<Check>
Groupement IOMMU idéal : l'iGPU (`8086:4626`, Intel Alder Lake-P) est **seule dans son groupe IOMMU** — aucun autre périphérique (pas de contrôleur audio HDMI ou composant système critique) n'est entraîné dans le passthrough.
</Check>

### 3. Binding VFIO au Démarrage
```bash
# /etc/modprobe.d/vfio.conf
options vfio-pci ids=8086:4626
softdep i915 pre: vfio-pci
```

```bash
echo -e "vfio\nvfio_iommu_type1\nvfio_pci" | sudo tee -a /etc/modules
sudo update-initramfs -u -k all
sudo reboot
```

*Vérification* : `lspci -nnk -s 00:02.0` confirme l'utilisation du driver `vfio-pci` côté hôte (au lieu de `i915`).

### 4. Passage de la VM en Chipset `q35`

<Warning>
**Incompatibilité `i440fx`** : Le passthrough PCIe (`pcie=1`) échoue avec l'erreur `q35 machine model is not enabled` sur le chipset par défaut `i440fx`. La bascule vers le chipset `q35` est obligatoire :

```bash
qm set 104 --machine q35
```
Changement appliqué et validé sans régression sur l'ensemble des 43 conteneurs Docker en production.
</Warning>

### 5. Attribution du GPU à la VM 104
```bash
qm set 104 --hostpci0 0000:00:02.0,pcie=1,x-vga=1
qm reboot 104
```

### 6. Installation des Pilotes dans la VM (Image Ubuntu Cloud)

<Warning>
**Absence de `/dev/dri/` & Métapaquet Générique Requis** : L'iGPU apparaît sur le bus PCI de la VM (`lspci`), mais le dossier `/dev/dri/` n'existe pas. Le module `i915` est absent de l'image Cloud minimale d'Ubuntu 24.04 (`linux-modules-extra` non installé par défaut). **Il faut impérativement installer le métapaquet générique `linux-modules-extra-generic`** (et non une version figée `$(uname -r)`) afin que les mises à jour automatiques du noyau (`unattended-upgrades`) conservent le module `i915` lors des futurs redémarrages. Voir l'[Fiche Incident Post-Mortem du 19/08/2026](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri).
</Warning>

```bash
sudo apt update
sudo apt install -y linux-modules-extra-generic linux-firmware intel-media-va-driver-non-free
sudo modprobe i915
```

*Résultat* : Les périphériques `/dev/dri/card0` et `/dev/dri/renderD128` (groupe `render`, GID `992`) deviennent immédiatement disponibles dans la VM.

---

## Conséquences & Validation

<Check>
**Validé en production sur HomeFlix (Jellyfin)** — Transcodage matériel HEVC/H.264 via `hevc_qsv` et `h264_qsv` confirmé dans les arguments `ffmpeg` (`-hwaccel qsv -c:v h264_qsv ... -codec:v:0 hevc_qsv`). Vitesse de transcodage observée à **29.7x le temps réel** avec une charge CPU négligeable.
</Check>

<Warning>
**Perte de la console vidéo de l'Hôte Proxmox** : L'hôte Proxmox perd son affichage graphique natif. Sans impact en exploitation courante (pilotage Web/SSH), mais un accès BIOS physique nécessiterait de libérer temporairement le GPU (`qm set 104 --delete hostpci0` + reboot host) ou de configurer Intel vPro/AMT.
</Warning>

<Info>
**Gestion des Droits Conteneurs** : Les conteneurs s'exécutant en root (Jellyfin, PhotoPrism) accèdent directement à `/dev/dri` sans conflit de GID. Tout conteneur non-root nécessiterait un `group_add` explicite sur le GID du groupe `render`.
</Info>
