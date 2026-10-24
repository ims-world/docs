---
title: "Headscale et Headplane"
description: "Serveur de coordination du tailnet (Headscale) et son interface d'administration (Headplane), hébergés sur la VM 104."
icon: "network-wired"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v0.29.3 / 0.7.1"
---

import { ips } from "/snippets/variables.mdx";

## Rôle

Headscale est une implémentation libre du serveur de coordination Tailscale. Il distribue les clés et les adresses ; le trafic circule ensuite directement entre les machines, en WireGuard, sans passer par lui. Headplane est son interface web d'administration.

## Accès

| Accès | Adresse | Zone |
| --- | --- | --- |
| Coordination (clients Tailscale) | `https://vpn.ims-world.fr` | Publique : les clients doivent le joindre avant d'être dans le tailnet |
| Headplane | `https://admin.vpn.ims-world.fr/admin` | Tailnet |

```bash
# Depuis la VM 104
docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale nodes list
docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale users list
docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale preauthkeys list --user <ID>
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `vpn.ims-world.fr`, `admin.vpn.ims-world.fr` |
| Accès | Coordination publique ; Headplane restreint au tailnet, OIDC Authentik |
| Hôte | VM 104 |
| Coolify | UUID `i136ix2bmrrbeovnyrh1o72w`, chemin `/data/coolify/services/i136ix2bmrrbeovnyrh1o72w/` |
| Image | `headscale/headscale:v0.29.3`, `ghcr.io/tale/headplane:0.7.1` |
| Données | `db.sqlite` (machines, utilisateurs, clés), `noise_private.key` (identité du serveur) |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma sur `vpn.ims-world.fr` |
| Mise en service | Août 2026 |

## Fonctionnement

| Utilisateur Headscale | Machines | Enrôlement |
| --- | --- | --- |
| `infra` | Toutes les VM et tous les conteneurs, nœuds compris, y compris les VM de tiers | Clé de pré-authentification réutilisable, utilisée par le rôle Ansible `tailscale` |
| Comptes personnels | Téléphones, ordinateurs | OIDC Authentik (groupes administrateurs et `membres`) |

La clé de pré-authentification active de `infra` expire le 18/10/2027. Elle est conservée dans le coffre d'ims-ops ; les anciennes clés sont expirées. La renouveler avant cette date et mettre à jour le coffre.

Les adresses des machines sont dans l'[architecture réseau](/reseau/architecture-reseau#adresses-des-hôtes).

## DNS interne (`extra_records`)

Certains noms sont résolus, pour les clients du tailnet, vers l'IP tailnet de Traefik (<code>{ips.coolify}</code>) plutôt que vers l'IP publique. Le trafic reste ainsi dans le tailnet et Traefik voit l'IP tailnet du client, condition du filtrage `vpn-only`.

```yaml
# /data/coolify/services/i136ix2bmrrbeovnyrh1o72w/config/config.yaml (ou depuis Headplane)
dns:
  extra_records:
    - name: "admin.vpn.ims-world.fr"
      type: "A"
      value: "100.64.0.4"
    - name: "home.ims-world.fr"
      type: "A"
      value: "100.64.0.4"
    # un enregistrement par service restreint au tailnet
```

Ajouter l'enregistrement fait partie de la procédure [Restreindre un service au tailnet](/procedures/securiser-service-vpn-only).

## Enrôlement par OIDC

```bash
tailscale up --login-server https://vpn.ims-world.fr
```

Le client affiche une adresse vers Authentik. Après connexion et second facteur, la machine reçoit son adresse en `100.64.0.x`. L'enrôlement est ouvert aux administrateurs et au groupe `membres` ; Headplane reste réservé aux administrateurs.

## Données et sauvegarde

`db.sqlite` et `noise_private.key` sont sauvegardés avec la VM 104. Perdre `noise_private.key` oblige à réenrôler toutes les machines.

## Exploitation

- Retirer une machine : `just tailnet-remove <nom>` depuis ims-ops (refusé pour les hôtes déclarés à la main), ou `headscale nodes delete -i <id>`.
- Enrôler un hôte géré : voir [Ajouter une machine au tailnet](/procedures/ajout-machine-headscale).

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `admin.vpn.ims-world.fr` renvoie 404 | Headplane n'a pas de route sur `/` | Utiliser `/admin` |
| `preauthkeys create --user infra` refusé | Depuis la version 0.26, `--user` attend l'ID | `headscale users list`, puis `--user <ID>` |
| Alerte de clé d'hôte SSH sur une nouvelle machine | Headscale réattribue d'anciennes adresses | Vérifier l'empreinte, puis `ssh-keygen -R` |
| Erreur de configuration après mise à jour vers 0.29 | Directive `randomize_client_port` retirée | La supprimer de `config.yaml` |

## Voir aussi

- [Ajouter une machine au tailnet](/procedures/ajout-machine-headscale)
- [Authentik](/services/authentik)
