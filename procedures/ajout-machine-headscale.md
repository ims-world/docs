---
title: "Ajouter une machine au tailnet"
description: "Enrôler un appareil personnel par Authentik, un hôte géré par ims-ops, ou une machine hors gestion par clé de pré-authentification."
icon: "network-wired"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : 5 à 15 minutes · Risque : faible · Interruption de service : aucune

## Quand l'utiliser

| Machine | Méthode | Utilisateur Headscale |
| --- | --- | --- |
| Téléphone, ordinateur personnel | [A — Connexion par Authentik](#option-a-appareil-personnel) | Compte de la personne |
| Hôte déclaré dans ims-ops, VM créée par `just vm` | [B — Rôle Ansible](#option-b-hôte-géré-par-ims-ops) | `infra` |
| Machine hors gestion (appliance, conteneur Docker) | [C — Clé de pré-authentification](#option-c-machine-hors-gestion) | `infra` |

Le fonctionnement du tailnet est décrit dans [Headscale](/reseau/headscale-headplane).

## Prérequis

- A : compte Authentik dans le groupe `membres` ou administrateurs.
- B : hôte déclaré dans l'inventaire ims-ops ; pour un conteneur LXC, accès à `/dev/net/tun` ([socle commun](/infrastructure/socle-commun#tailnet)).
- C : accès à Headplane ou à la VM 104.

## Étapes

### Option A : appareil personnel

<Steps>
  <Step title="Installer Tailscale">
    Application officielle (App Store, Play Store, site Tailscale) ou paquet `tailscale` sous Linux.
  </Step>
  <Step title="Indiquer le serveur">
    Application : **Paramètres → Accounts → Change Server**, `https://vpn.ims-world.fr`. Ligne de commande :
    ```bash
    tailscale up --login-server https://vpn.ims-world.fr
    ```
  </Step>
  <Step title="Se connecter">
    La page d'enregistrement redirige vers Authentik : connexion, second facteur, validation.
  </Step>
</Steps>

### Option B : hôte géré par ims-ops

La clé de pré-authentification de `infra` est lue dans le coffre d'ims-ops : elle n'est jamais copiée sur l'hôte ni dans un snippet Proxmox.

```bash
just check <hôte> tailscale
just setup <hôte> tailscale
```

Puis renseigner `ip_tailscale` de l'hôte dans l'inventaire. Une VM créée par `just vm … full` est enrôlée et inscrite automatiquement.

<Warning>
L'ancienne méthode (template 8000 et snippet Cloud-Init `vendor-data.yaml` contenant la clé en clair) est abandonnée depuis le 24/10/2026. Le template et le snippet ont été supprimés.
</Warning>

### Option C : machine hors gestion

<Steps>
  <Step title="Créer une clé">
    Headplane (`https://admin.vpn.ims-world.fr/admin`) : utilisateur `infra`, onglet **Pre-Auth Keys**, **Create**. En ligne de commande, depuis la VM 104 :
    ```bash
    docker exec -it headscale-i136ix2bmrrbeovnyrh1o72w headscale users list
    docker exec -it headscale-i136ix2bmrrbeovnyrh1o72w headscale preauthkeys create --user <ID> --expiration 1h
    ```
    Une clé à usage unique et de courte durée suffit pour une machine. L'expiration ne concerne que l'enrôlement : une machine enrôlée reste connectée après l'expiration de la clé.
  </Step>
  <Step title="Enrôler">
    ```bash
    sudo tailscale up --login-server https://vpn.ims-world.fr --authkey <clé>
    ```
    Dans un conteneur Docker : la même commande par `docker exec -it <conteneur> tailscale up …`. Home Assistant OS utilise son module Tailscale ([procédure](/infrastructure/vm-home-assistant#module-tailscale)).
  </Step>
</Steps>

`--snat-subnet-routes=false` ne concerne que la VM 104, où Traefik doit voir l'IP réelle des clients.

## Vérification

```bash
tailscale status
tailscale ip -4
# Depuis la VM 104
docker exec headscale-i136ix2bmrrbeovnyrh1o72w headscale nodes list
```

<Check>
La machine apparaît avec une adresse en `100.64.0.x` et le bon utilisateur.
</Check>

Si un service de la machine doit être joint par un nom, ajouter l'enregistrement dans les `extra_records` ([Headscale](/reseau/headscale-headplane#dns-interne-extra_records)) et l'adresse dans l'[architecture réseau](/reseau/architecture-reseau#adresses-des-hôtes).

## Retour arrière

`just tailnet-remove <nom>` depuis ims-ops, ou `headscale nodes delete -i <id>` depuis la VM 104.

## Dépannage

| Symptôme | Cause | Solution |
| --- | --- | --- |
| `tailscale` ne démarre pas dans un LXC | `/dev/net/tun` non transmis | Ajouter les deux lignes de la [configuration LXC](/infrastructure/socle-commun#tailnet), redémarrer le conteneur |
| `--user infra` refusé | `--user` attend l'ID | `headscale users list` |
| Alerte de clé d'hôte SSH | Adresse déjà utilisée par une ancienne machine | Vérifier l'empreinte, puis `ssh-keygen -R` |
