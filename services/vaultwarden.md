---
title: "Vaultwarden"
description: "Gestionnaire de mots de passe compatible Bitwarden, connexion par Authentik."
icon: "vault"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "1.32.7"
---

import { domains } from "/snippets/variables.mdx";

## Accès

Coffre public avec connexion Authentik ; le panneau d'administration ne répond que depuis le tailnet.

<CardGroup cols={2}>
  <Card title="Coffre web" icon="shield-halved" href="https://vault.ims-world.fr">
    Interface web ; extensions et applications Bitwarden configurées sur ce serveur.
  </Card>
  <Card title="Administration" icon="gear" href="https://vault.ims-world.fr/admin">
    Panneau d'administration, restreint au tailnet.
  </Card>
</CardGroup>

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | {domains.vault} |
| Accès | Public ; OIDC Authentik, mot de passe maître pour le déchiffrement. `/admin` restreint au tailnet |
| Hôte | VM 104 |
| Coolify | UUID `i5ae953riutbot9afjcboptb`, chemin `/data/coolify/services/i5ae953riutbot9afjcboptb/` |
| Image | `vaultwarden/server:1.32.7` |
| Données | Volume `i5ae953riutbot9afjcboptb_vaultwarden-data` : base SQLite, `rsa_key.pem`, pièces jointes |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | Août 2026 |

## Fonctionnement

Un seul conteneur, base SQLite en mode WAL. Le domaine est figé dans `config.json` (`vault.ims-world.fr`). Les inscriptions sont fermées (`SIGNUPS_ALLOWED=false`) : un nouvel utilisateur passe par une invitation.

## Données et sauvegarde

Le volume est sauvegardé avec la VM 104. `rsa_key.pem` doit être restauré avec la base : sans lui, les pièces jointes sont illisibles. Pour une copie ponctuelle cohérente, arrêter brièvement le conteneur :

```bash
docker stop vaultwarden-i5ae953riutbot9afjcboptb
sudo tar czf ~/vaultwarden-$(date +%F).tar.gz -C /var/lib/docker/volumes/i5ae953riutbot9afjcboptb_vaultwarden-data/_data .
docker start vaultwarden-i5ae953riutbot9afjcboptb
```

## Exploitation

Mise à jour : changer le tag dans Coolify et redéployer. Lire les notes de version avant un changement de version majeure.

## Pièges connus

Chaque piège donne le symptôme observé, sa cause et la correction.

<AccordionGroup>
  <Accordion title="La connexion par Authentik échoue sans message">
    **Cause** : Authentik n'envoie pas `email_verified: true`.

    **Solution** : ajouter un mappage de propriété qui renvoie `email_verified: true`.
  </Accordion>
</AccordionGroup>

## Voir aussi

<CardGroup cols={2}>
  <Card title="Authentik" icon="key" href="/services/authentik">Connexion unique des services.</Card>
  <Card title="Plan de reprise" icon="life-ring" href="/procedures/plan-reprise-activite-pra">Restauration après une panne.</Card>
</CardGroup>
