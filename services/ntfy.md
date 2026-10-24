---
title: "Ntfy"
description: "Serveur de notifications push : toutes les alertes du homelab arrivent sur le topic ims-alerts, lu par l'application mobile."
icon: "bell"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "v2.28.0"
---

## Accès

<CardGroup cols={2}>
  <Card title="Ntfy" icon="bell" href="https://ntfy.ims-world.fr">
    Interface web.
  </Card>
  <Card title="Application mobile" icon="mobile">
    Serveur `https://ntfy.ims-world.fr`, topic `ims-alerts`, compte en lecture seule.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user list
docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy access
curl -H "Authorization: Bearer <jeton>" -d "Test" https://ntfy.ims-world.fr/ims-alerts
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `ntfy.ims-world.fr` |
| Accès | Public (réception sur le téléphone hors du tailnet) ; inscription fermée, accès refusé par défaut |
| Hôte | VM 104 |
| Coolify | UUID `j5akn2e9pr6g7c2pjvdj78w0`, chemin `/data/coolify/services/j5akn2e9pr6g7c2pjvdj78w0/` |
| Image | `binwiederhier/ntfy:v2.28.0` |
| Données | Base des utilisateurs, droits et jetons |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 11/08/2026 |

## Fonctionnement

Ntfy est public par choix : les alertes doivent arriver sur le téléphone en toutes circonstances. La protection tient à deux réglages : `NTFY_ENABLE_SIGNUP=false` et `NTFY_AUTH_DEFAULT_ACCESS=deny-all` (compte ou jeton obligatoire pour lire comme pour publier).

| Émetteur | Topic |
| --- | --- |
| Grafana (alertes de supervision) | `ims-alerts` |
| Uptime Kuma (indisponibilités) | `ims-alerts` |
| fail2ban (bannissements sur les hôtes) | `ims-alerts` |
| Proxmox VE et PBS (échecs de sauvegarde), utilisateur `proxmox` | `ims-alerts` |
| WUD (nouvelles versions d'images) | `ims-alerts` |

**Règle :** un utilisateur Ntfy par outil, en écriture seule, et un jeton par usage (exemple : utilisateur `proxmox`, jetons `pve` et `pbs`). Un jeton compromis ne permet ni de lire les alertes ni d'administrer le serveur, et se révoque sans toucher aux autres. Les comptes humains sont en lecture seule.

## Données et sauvegarde

Sauvegardé avec la VM 104. Une perte oblige à recréer utilisateurs et jetons, et à les reporter dans chaque outil.

## Exploitation

<AccordionGroup>
  <Accordion title="Raccorder un nouvel outil">
    ```bash
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy user add <outil>
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy access <outil> ims-alerts write-only
    docker exec -it ntfy-j5akn2e9pr6g7c2pjvdj78w0 ntfy token add --label="<usage>" <outil>
    ```
    Le mot de passe demandé à la création n'est pas utilisé. Le jeton se stocke dans l'outil, jamais dans un dépôt.
  </Accordion>
  <Accordion title="Créer un compte de lecture">
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

## Pièges connus

| Symptôme | Cause | Solution |
| --- | --- | --- |
| Jeton refusé par fail2ban (ims-ops) | Valeur saisie avec le préfixe `Bearer` | Ne garder que la valeur `tk_…` dans le coffre |
| Un hôte n'arrive pas à joindre Ntfy | L'hôte n'utilise pas le DNS du tailnet | ims-ops force l'adresse `192.168.1.52` pour fail2ban |

## Voir aussi

- [Supervision](/services/monitoring)
- [Politique de sauvegarde](/procedures/politique-sauvegardes) (notifications PVE et PBS)
