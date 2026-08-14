---
title: "ADR-006 — Exposition directe du port TCP 2222 (Forgejo Git SSH) sur la Bbox"
description: "Décision d'ouvrir une règle de redirection NAT Bbox sur le port TCP 2222 pour le trafic Git SSH sans passer par Traefik"
icon: "network-wired"
iconType: "duotone"
---

## Statut
<Badge color="green">🟢 Accepté & Déployé</Badge> *(2026-08-14)*

---

## Contexte

Les opérations de versioning Git à travers le protocole SSH (commandes `git clone`, `git push`, `git fetch`) s'effectuent sur des connexions TCP brutes (couche 4). Contrairement aux requêtes Web HTTP/HTTPS (couche 7) gérées par Traefik, ce trafic ne s'intercale pas naturellement dans un reverse proxy HTTP sans configuration complexe de routage SNI/TCP.

Pour permettre l'utilisation fluide des commandes Git CLI depuis l'extérieur tout en conservant le port SSH principal de l'hôte sécurisé (port 4242), un port SSH dédié (`2222`) a été attribué au conteneur Forgejo.

---

## Décision

Nous avons décidé d'exposer publiquement le port TCP **`2222`** en configurant une règle de redirection de port (NAT) sur le routeur Bbox :

| Règle Bbox | Port Externe | Port Interne | IP Cible |
|---|---|---|---|
| `Forgejo_SSH` | 2222 | 2222 | `192.168.1.52` (VM IMS-Coolify) |

Dans le `docker-compose.yml` du service [Forgejo](/services/forgejo), le port est mappé directement sur l'hôte : `ports: - '2222:2222'`.

---

## Conséquences

### Positives
- **Expérience développeur standard** : Commandes Git SSH utilisables depuis n'importe quel réseau externe sans VPN obligatoire (`git clone ssh://git@forge.ims-world.fr:2222/...`).
- **Isolation du SSH système** : Le port 2222 du conteneur est totalement étanche vis-à-vis du port SSH système de la VM (4242).

### Négatives / Contraintes de Sécurité
- **Contournement du Reverse Proxy & d'Authentik** : Ce flux TCP brut ne bénéficie ni du filtrage Traefik (`vpn-only`) ni de la protection SSO d'Authentik.
- **Sécurité basée sur les clés SSH** : La protection du port 2222 repose intégralement sur le moteur de gestion des clés publiques SSH et le durcissement interne du conteneur Forgejo.
