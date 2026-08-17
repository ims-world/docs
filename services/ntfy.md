---
title: "Ntfy — Notifications Push"
description: "Serveur de notifications push pour l'alerting et les applications mobile"
icon: "bell"
iconType: "duotone"
last_reviewed: "2026-08-12"
app_version: "v2.27.0"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Production Active</Badge>

## Accès Rapides & Administration

<Tabs>
  <Tab title="🌐 Interface Web & Mobile">
    <Card title="Ntfy Web UI" icon="bell" href="https://ntfy.ims-world.fr">
      Interface web du serveur ntfy. Accessible sur `ntfy.ims-world.fr`.
    </Card>
    <Card title="App Mobile (Android / iOS)" icon="mobile">
      Configurer l'application avec le serveur `https://ntfy.ims-world.fr`, le topic `ims-alerts` et vos identifiants ntfy.
    </Card>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Se connecter à la VM Coolify
    ssh cmolotkoff@100.64.0.4

    # Gérer les utilisateurs (nom de conteneur Coolify : ntfy-j5akn2e9pr6g7c2pjvdj78w0)
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user list
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user add --role=admin cmolotkoff

    # Générer un token d'accès (ex: pour Grafana)
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy token add --label="grafana-alerting" cmolotkoff

    # Test d'envoi manuel
    curl -H "Authorization: Bearer <token>" -d "Alerte de test" https://ntfy.ims-world.fr/ims-alerts
    ```
  </Tab>
</Tabs>

---

## Fiche Service

| Propriété | Valeur |
|---|---|
| **Domaine** | `ntfy.ims-world.fr` |
| **Rôle** | Serveur de Notifications Push (Grafana Alerting & notifications système) |
| **Version** | `binwiederhier/ntfy:v2.27.0` |
| **Hôte d'Orchestration** | VM IMS-Coolify (VM 104) |
| **UUID Coolify** | `j5akn2e9pr6g7c2pjvdj78w0` |
| **Chemin sur la VM** | `/data/coolify/services/j5akn2e9pr6g7c2pjvdj78w0/` |
| **Topic Principal** | `ims-alerts` |
| **Exposition** | **Publique** (accès mobile hors VPN) |
| **Statut** | <Badge color="green">🟢 Production Active</Badge> |

<Warning>
**Exposition publique assumée pour l'alerting mobile** : Ntfy est délibérément accessible sur Internet sans filtrage Tailnet afin de garantir la réception des alertes push sur smartphone en mobilité hors du réseau privé. La sécurité repose sur les directives système `NTFY_ENABLE_SIGNUP=false` (pas d'inscription libre) et `NTFY_AUTH_DEFAULT_ACCESS=deny-all` (authentification obligatoire par compte ou token Bearer pour lire ou publier).
</Warning>

---

## Configuration & Authentification

- **Inscription désactivée** (`NTFY_ENABLE_SIGNUP=false`).
- **Accès par défaut refusé** (`NTFY_AUTH_DEFAULT_ACCESS=deny-all`).
- **Comptes & Tokens** : La gestion des droits s'effectue directement en CLI dans le conteneur (`ntfy user` et `ntfy token`). Les accès applicatifs (comme le Contact Point [Grafana](/services/monitoring#alerting--contact-point-ntfy)) utilisent des tokens d'API dédiés.

---

## Exploitation & Procédures CLI

<AccordionGroup>
  <Accordion title="Créer un utilisateur ou un token d'API">
    ```bash
    # Créer un compte utilisateur
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user add --role=admin nouvel_utilisateur

    # Attribuer les droits d'accès sur le topic ims-alerts
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy access nouvel_utilisateur ims-alerts read-write

    # Générer un token pour une application
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy token add --label="mon-application" cmolotkoff
    ```
  </Accordion>
</AccordionGroup>
