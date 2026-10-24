---
title: "Déployer un service"
description: "Mettre un nouveau service en production sur Coolify : déploiement, exposition, authentification, supervision, sauvegarde et documentation."
icon: "rocket"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : 1 heure environ · Risque : faible · Interruption de service : aucune pour les services existants

## Quand l'utiliser

Pour tout nouveau service Docker sur la VM 104 ou le worker 105. Pour un hôte ou une VM, voir plutôt [Administration avec ims-ops](/procedures/administration-ims-ops).

## Prérequis

- Accès à Coolify et SSH sur la VM 104.
- Image avec un tag de version précis (jamais `latest`).
- Choix de l'exposition fait : public avec authentification, ou restreint au tailnet ([matrice](/reseau/matrice-securite-exposition)).

## Étapes

<Steps>
  <Step title="Créer la ressource dans Coolify">
    Service Docker Compose (ou modèle Coolify), tag d'image figé. Secrets en variables d'environnement avec **Is Secret**. Conteneur attaché au réseau `coolify`.
  </Step>
  <Step title="Placer les données">
    Base de données : volume local de la VM (jamais sur NFS). Fichiers volumineux : partage du NAS (`/mnt/nas-storage` ou `/mnt/nas-hot`). Vérifier qu'aucun fichier de configuration attendu n'a été remplacé par un dossier vide (`file <chemin>`).
  </Step>
  <Step title="Valider sur un domaine temporaire">
    Démarrer sur `<nom>-ng.ims-world.fr`, vérifier les journaux, le certificat et le fonctionnement.
  </Step>
  <Step title="Exposer sur le domaine définitif">
    - Service public : domaine dans le champ **Domains** de Coolify, authentification par [OIDC](/procedures/integration-service-authentik-oidc) ou [forward-auth](/procedures/securiser-application-authentik-forward-auth).
    - Service restreint : champ **Domains** vide, routeur dans `vpn-only.yaml` ([procédure](/procedures/securiser-service-vpn-only)).

    Supprimer ensuite le domaine temporaire.
  </Step>
  <Step title="Superviser">
    Ajouter une sonde [Uptime Kuma](/services/uptime-kuma) : domaine public, ou nom du conteneur pour un service restreint.
  </Step>
  <Step title="Vérifier la sauvegarde">
    Rien à faire pour les données de la VM 104 ou du worker (pool `backup`). Si des données importantes sont sur le NAS, les signaler : elles ne sont pas sauvegardées ([politique](/procedures/politique-sauvegardes#ce-qui-nest-pas-sauvegardé)).
  </Step>
  <Step title="Documenter">
    Créer la fiche depuis `_modeles/service.md`, l'ajouter à `docs.json`, ajouter la ligne dans la [matrice d'exposition](/reseau/matrice-securite-exposition) et un bloc dans la [chronologie](/history/chronologie).
  </Step>
</Steps>

## Vérification

```bash
# Hors tailnet
curl -sI https://<nom>.ims-world.fr | head -1   # public : 200 ou redirection ; restreint : 403
# Depuis le tailnet
curl -sI https://<nom>.ims-world.fr | head -1   # 200 ou redirection
```

<Check>
La sonde Uptime Kuma est au vert, WUD voit l'image, la fiche est publiée.
</Check>

## Retour arrière

Arrêter puis supprimer la ressource dans Coolify, retirer le routeur de `vpn-only.yaml` s'il existe, la sonde Uptime Kuma, l'application Authentik et la ligne de la matrice.

## Dépannage

Voir [Dépannage courant](/procedures/depannage-courant#docker-et-coolify).
