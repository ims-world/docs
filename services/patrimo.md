---
title: "Patrimo"
description: "Application personnelle Node.js, déployée automatiquement par Coolify à chaque push sur GitHub."
icon: "chart-pie"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "git-main"
---

## Accès

Public, avec compte applicatif.

<CardGroup cols={2}>
  <Card color="#16A34A" title="Patrimo" icon="chart-pie" href="https://patrimo.ims-world.fr">
    Application web.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
docker ps -qf "name=patrimo" | xargs -I {} docker logs --tail 100 {}
docker volume ls | grep patrimo
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `patrimo.ims-world.fr` |
| Accès | Public ; compte applicatif |
| Hôte | VM 104 |
| Coolify | Application Git (pas de dossier sous `/data/coolify/services/`) |
| Image | Construite depuis le dépôt GitHub privé (branche principale), `docker-compose.yaml` du dépôt |
| Données | PostgreSQL dans un volume Docker nommé |
| Sauvegarde | Avec la VM 104 (volumes Docker compris) |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 14/08/2026 |

## Fonctionnement

Contrairement aux autres services, décrits par un compose statique dans Coolify, Patrimo est une **Application** Coolify : l'application GitHub de Coolify reçoit chaque push sur la branche principale et reconstruit l'image. La version en service est donc toujours le dernier commit de cette branche.

## Données et sauvegarde

La base est dans un volume Docker nommé sur la VM 104 : elle est incluse dans la sauvegarde nocturne de la VM. Pour une copie ponctuelle, voir la [politique de sauvegarde](/procedures/politique-sauvegardes) (export PostgreSQL).

## Exploitation

Déployer : pousser sur la branche principale. Revenir en arrière : pousser un commit d'annulation, ou redéployer un déploiement précédent depuis Coolify.

## Pièges connus

Sans objet.

## Voir aussi

<CardGroup cols={2}>
  <Card color="#64748B" title="Déployer un service" icon="list-check" href="/procedures/deploiement-service">Étapes d'un nouveau service.</Card>
</CardGroup>
