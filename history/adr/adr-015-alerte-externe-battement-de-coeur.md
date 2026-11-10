---
title: "ADR-015 — Alerte hors du homelab par battement de cœur"
description: "Les nœuds et la VM Coolify signalent leur présence toutes les 5 minutes à Healthchecks.io, qui alerte par courriel si le signal cesse."
icon: "scale-balanced"
iconType: "duotone"
last_reviewed: "2026-11-03"
---

## Statut

Accepté — 03/11/2026.

## Contexte

Toute la chaîne d'alerte est à la maison : Grafana, Uptime Kuma et Ntfy tournent sur la VM 104, derrière Traefik. Une coupure de courant ou d'accès Internet éteint en même temps ce qui tombe et ce qui devrait prévenir : aucune alerte ne part. C'est aussi le cas d'une panne de la VM 104 seule, qui porte la supervision et Ntfy. Un outil de supervision placé sur le Mac Mini ne change rien, puisqu'il partage le courant et l'accès Internet du MS-01.

## Décision

| Sujet | Décision |
| --- | --- |
| Principe | Battement de cœur : chaque hôte appelle une URL toutes les 5 minutes ; le service externe alerte quand les appels cessent |
| Service | Healthchecks.io, offre gratuite (20 contrôles) |
| Hôtes | `pve`, `pve-macmini`, `ims-coolify` (VM 104) : un contrôle chacun |
| Réglage des contrôles | Période de 5 minutes, délai de grâce de 10 minutes |
| Canal d'alerte | Courriel uniquement |
| Mise en œuvre | Rôle `heartbeat` d'ims-ops : tâche cron exécutée par `nobody` ; URL dans le coffre, jamais affichée dans un diff |

Ce que le dispositif couvre : coupure de courant, coupure d'accès Internet, nœud planté ou figé, VM 104 arrêtée. Ce qu'il ne couvre pas : une panne de Traefik avec la VM 104 en marche, signalée seulement par la supervision interne.

## Alternatives écartées

| Option | Raison du rejet |
| --- | --- |
| Uptime Kuma sur le Mac Mini | Même courant et même accès Internet que le MS-01 : muet lors d'une coupure |
| Better Stack | 10 battements de cœur gratuits, mais outil commercial orienté vers le payant |
| UptimeRobot | Battement de cœur absent de l'offre gratuite |
| Healthchecks ou Uptime Kuma sur un VPS | Serveur supplémentaire à maintenir et à surveiller pour trois contrôles |
| Cronitor, Dead Man's Snitch | Offres gratuites minimes, hébergement hors UE |
| SMS, Telegram, ntfy.sh public | SMS payants ; les autres canaux ajoutent une dépendance. Le courriel suffit et arrive sur le téléphone par le réseau mobile |

## Conséquences

- Une coupure générale est signalée en 15 minutes au plus.
- Le choix du service n'enferme pas : changer de prestataire revient à changer les URL dans le coffre. Healthchecks.io est un logiciel libre, auto-hébergeable si besoin.
- Un contrôle supplémentaire se déclare dans `host_vars` (voir le README du rôle `heartbeat`).
- Une maintenance prévue (arrêt d'un nœud) déclenche une alerte : mettre le contrôle en pause sur Healthchecks.io avant l'intervention.

## Amendements

<Update label="10/11/2026" description="URL hors de la ligne de commande">
  La tâche cron est exécutée par root et lit l'URL dans `/etc/ims-heartbeat.curlrc` (fichier réservé à root) : l'URL n'apparaît plus dans la tâche planifiée ni dans la liste des processus. Le jeton Ntfy de fail2ban suit le même principe (`/etc/fail2ban/ntfy.header`).
</Update>
