---
title: "Commandes d'Urgence & 1-Liners"
description: "Aide-mémoire des commandes copier-coller d'administration et de diagnostic rapide"
icon: "bolt"
iconType: "duotone"
---

<Info>
Cette page rassemble les **commandes copier-coller indispensables** pour intervenir rapidement sur le cluster Proxmox VE, la VM Coolify, le réseau Tailscale ou le stockage NAS.
</Info>

## 🖥️ Proxmox VE & Hyperviseur

<CodeGroup>
```bash État Général du Cluster
# Lister le statut de toutes les VM et conteneurs LXC
pvesh get /nodes/localhost/tasks --limit 10
pct list
qm list
```

```bash Redémarrage d'urgence d'un Guest
# Forcer l'arrêt puis le redémarrage d'un conteneur LXC (ex: LXC 100 NAS)
pct stop 100 && pct start 100

# Redémarrer la VM Coolify (VM 104)
qm reboot 104
```

```bash Inspecter la Mémoire & les Disques Hôte
# Vérifier la consommation RAM / SWAP réelle de l'hôte MS-01
free -h && zfs list 2>/dev/null || df -h /

# Vérifier la santé SMART du HDD 3To physique (depuis le host MS-01)
smartctl -H /dev/disk/by-id/ata-APPLE_HDD_ST3000DM001_Z1F3N0NZ
```
</CodeGroup>

## 🐳 Docker & Proxy Traefik (VM 104 Coolify)

<CodeGroup>
```bash Redémarrage Rapide Traefik Proxy
# Redémarrer le conteneur Traefik sans toucher aux applications
docker restart coolify-proxy
```

```bash Inspecter les Logs d'un Service
# Afficher les 100 dernières lignes de log d'un conteneur
docker logs --tail 100 -f <container_name_or_id>

# Purger les logs Docker qui remplissent le disque
sudo truncating-logs --all 2>/dev/null || sudo sh -c 'truncate -s 0 /var/lib/docker/containers/*/*-json.log'
```

```bash Vérifier les Montages FUSE / NFS dans un Conteneur
# Tester l'accès au stockage NAS depuis un conteneur applicatif
docker exec -it <container_id> df -h /mnt/nas-storage
```
</CodeGroup>

## 🔐 Réseau & VPN Tailscale / Headscale

<CodeGroup>
```bash Statut & Diagnostic VPN
# Vérifier la connexion au Tailnet Headscale
tailscale status

# Forcer la ré-authentification d'une machine distante
tailscale up --login-server=https://vpn.ims-world.fr --reset
```

```bash Test de Filtrage Middleware vpn-only
# Tester le blocage WAN (doit renvoyer un 403 Forbidden hors VPN)
curl -Iv https://qbit.ims-world.fr

# Tester la résolution interne du proxy Traefik
curl -Iv -H "Host: auth.ims-world.fr" http://127.0.0.1
```
</CodeGroup>

## 📁 Stockage & Montage NFS / FUSE

<CodeGroup>
```bash Re-montage d'urgence NFS
# Forcer le re-montage de tous les volumes NFS du fstab
sudo mount -a -t nfs -o remount

# Démonter un point de montage NFS bloqué (Stale file handle)
sudo umount -l -f /mnt/nas-storage
```

```bash Vérification des Fichiers de Config Coolify
# Corriger les permissions ACL sans altérer le propriétaire root/docker
sudo setfacl -R -m u:cmolotkoff:rwX /data/coolify/services
sudo setfacl -R -d -m u:cmolotkoff:rwX /data/coolify/services
```
</CodeGroup>
