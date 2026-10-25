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

Le serveur de coordination est public, car un client doit le joindre avant d'être dans le tailnet. Headplane ne répond que depuis le tailnet.

<CardGroup cols={2}>
  <Card title="Coordination" icon="network-wired" href="https://vpn.ims-world.fr">Adresse à donner aux clients Tailscale. Public.</Card>
  <Card title="Headplane" icon="sliders" href="https://admin.vpn.ims-world.fr/admin">Machines, utilisateurs, clés, DNS. Tailnet.</Card>
</CardGroup>

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

Deux sortes d'utilisateurs Headscale, selon le type de machine.

<CardGroup cols={2}>
  <Card title="infra" icon="server">Toutes les VM et tous les conteneurs, nœuds et VM de tiers compris. Clé de pré-authentification réutilisable, utilisée par le rôle Ansible `tailscale`.</Card>
  <Card title="Comptes personnels" icon="laptop">Téléphones, ordinateurs. Enrôlement par OIDC Authentik (groupes administrateurs et `membres`).</Card>
</CardGroup>

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

Chaque piège donne le symptôme observé, sa cause et la correction.

<AccordionGroup>
  <Accordion title="admin.vpn.ims-world.fr renvoie 404">
    **Cause** : Headplane n'a pas de route sur `/`.

    **Solution** : utiliser `/admin`.
  </Accordion>
  <Accordion title="preauthkeys create --user infra refusé">
    **Cause** : depuis la version 0.26, `--user` attend l'ID.

    **Solution** : `headscale users list`, puis `--user <ID>`.
  </Accordion>
  <Accordion title="Alerte de clé d'hôte SSH sur une nouvelle machine">
    **Cause** : Headscale réattribue d'anciennes adresses.

    **Solution** : vérifier l'empreinte, puis `ssh-keygen -R`.
  </Accordion>
  <Accordion title="Erreur de configuration après mise à jour vers 0.29">
    **Cause** : directive `randomize_client_port` retirée.

    **Solution** : la supprimer de `config.yaml`.
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card title="Ajouter une machine au tailnet" icon="list-check" href="/procedures/ajout-machine-headscale">Enrôler un appareil ou une VM.</Card>
  <Card title="Authentik" icon="key" href="/services/authentik">Connexion unique des services.</Card>
</CardGroup>
