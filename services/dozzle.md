---
title: "Dozzle"
description: "Journaux des conteneurs Docker de la VM 104 en direct, dans le navigateur."
icon: "list"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v11.0.1"
---

## Accès

Restreint au tailnet, puis connexion Authentik réservée aux administrateurs.

<CardGroup cols={2}>
  <Card color="#16A34A" title="Dozzle" icon="list" href="https://logs.ims-world.fr">
    Journaux en direct de chaque conteneur.
  </Card>
</CardGroup>

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `logs.ims-world.fr` |
| Accès | Tailnet, puis forward-auth Authentik (administrateurs) |
| Hôte | VM 104 |
| Coolify | UUID `ejdn7jiuwiyixrmp8nffjkcj`, chemin `/data/coolify/services/ejdn7jiuwiyixrmp8nffjkcj/` |
| Image | `amir20/dozzle:v11.0.1` |
| Données | Aucune ; socket Docker monté en lecture seule |
| Sauvegarde | Sans objet |
| Supervision | Sonde Uptime Kuma |
| Mise en service | Août 2026 |

## Fonctionnement

Dozzle lit le socket Docker de la VM 104 et diffuse les journaux en direct. Il ne stocke rien.

Le routeur est dans `vpn-only.yaml`, avec le middleware `authentik-dozzle@docker` : sans ce middleware, les labels du conteneur sont ignorés et le SSO ne s'applique pas ([incident du 24/08/2026](/history/incidents/2026-08-24-bypass-sso-dozzle-traefik-file-provider)).

## Dozzle ou Loki

| Besoin | Dozzle | Grafana et Loki |
| --- | --- | --- |
| Voir ce qui se passe maintenant, en un clic | Oui | Possible (mode Live), plus lent à atteindre |
| Historique après la suppression d'un conteneur | Non | 30 jours |
| Recherche sur plusieurs hôtes | Non (VM 104 seule) | Oui (LogQL) |
| Alertes sur le contenu des journaux | Non | Oui |

Les deux sont gardés ([ADR-001](/history/adr/adr-001-stack-monitoring-lgtm)).

## Données et sauvegarde

Sans objet.

## Exploitation

```bash
cd /data/coolify/services/ejdn7jiuwiyixrmp8nffjkcj/ && docker compose logs -f --tail=100
```

## Pièges connus

Voir l'incident cité plus haut : tout middleware d'authentification d'un service restreint au tailnet se déclare dans `vpn-only.yaml`.

## Voir aussi

<CardGroup cols={2}>
  <Card color="#16A34A" title="Supervision" icon="chart-line" href="/services/monitoring">Métriques, journaux et alertes.</Card>
</CardGroup>
