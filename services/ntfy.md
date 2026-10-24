---
title: "Ntfy — Notifications Push"
description: "Serveur de notifications push pour l'alerting et les applications mobile"
icon: "bell"
iconType: "duotone"
last_reviewed: "2026-09-14"
app_version: "v2.28.0"
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
      Configurer l'application avec le serveur `https://ntfy.ims-world.fr`, le topic `ims-alerts` et un compte en lecture seule.
    </Card>
  </Tab>
  <Tab title="⚡ Commandes CLI & Maintenance">
    ```bash
    # Se connecter à la VM Coolify
    ssh cmolotkoff@100.64.0.4

    # Gérer les utilisateurs (nom de conteneur Coolify : ntfy-j5akn2e9pr6g7c2pjvdj78w0)
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user list
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy access

    # Jeton d'un outil : toujours sur l'utilisateur de cet outil, jamais sur un compte admin
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy token add --label="<usage>" <outil>

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
| **Version** | `binwiederhier/ntfy:v2.28.0` |
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
- **Comptes et jetons** : gérés en CLI dans le conteneur (`ntfy user`, `ntfy access`, `ntfy token`).
- **Un utilisateur par outil, en écriture seule** sur `ims-alerts`, avec un jeton par usage. Exemple : utilisateur `proxmox`, jetons `pve` et `pbs`. Un jeton compromis ne permet alors ni de lire les alertes ni d'administrer le serveur, et se révoque sans toucher aux autres outils.
- Les comptes humains (application mobile) ont un accès en lecture seule. Le [point de contact Grafana](/services/monitoring#alerting--contact-point-ntfy) suit la même règle.

---

## Exploitation & Procédures CLI

<AccordionGroup>
  <Accordion title="Raccorder un nouvel outil (utilisateur dédié et jeton)">
    ```bash
    # 1. Créer l'utilisateur de l'outil (rôle user, mot de passe demandé puis inutilisé)
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user add <outil>

    # 2. Écriture seule sur le topic d'alertes
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy access <outil> ims-alerts write-only

    # 3. Un jeton par usage, à stocker dans l'outil (jamais dans un dépôt)
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy token add --label="<usage>" <outil>
    ```
  </Accordion>
  <Accordion title="Créer un compte de lecture (application mobile)">
    ```bash
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user add <personne>
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy access <personne> ims-alerts read-only
    ```
  </Accordion>
  <Accordion title="Révoquer un jeton">
    ```bash
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy token list <outil>
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy token remove <outil> <jeton>
    ```
  </Accordion>
</AccordionGroup>
