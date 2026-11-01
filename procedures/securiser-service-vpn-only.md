---
title: "Restreindre un service au tailnet"
description: "Rendre une interface web joignable seulement depuis le tailnet et le LAN, par un routeur dans vpn-only.yaml."
icon: "lock"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Durée : 15 minutes · Risque : faible (une erreur expose ou bloque le service, sans autre effet) · Interruption de service : quelques secondes au redéploiement

## Quand l'utiliser

Pour toute interface d'administration qui n'a pas à être joignable depuis Internet : outils d'administration, interfaces des services de téléchargement, tableaux de bord. Le principe est décrit dans l'[ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only) et dans [Traefik](/reseau/traefik-proxy#services-restreints-au-tailnet).

Pour un service public avec authentification, voir plutôt [forward-auth](/procedures/securiser-application-authentik-forward-auth) ou [OIDC](/procedures/integration-service-authentik-oidc).

## Prérequis

- Accès à l'interface Coolify et SSH sur la VM 104.
- Le service déployé dans Coolify et fonctionnel.

## Étapes

<Steps>
  <Step title="Déclarer le nom dans le DNS du tailnet">
    Ajouter le sous-domaine aux `extra_records` de Headscale (depuis Headplane, ou dans `/data/coolify/services/i136ix2bmrrbeovnyrh1o72w/config/config.yaml`) :

    ```yaml
    dns:
      extra_records:
        - name: "<sous-domaine>.ims-world.fr"
          type: "A"
          value: "100.64.0.4"
    ```

    Sans cet enregistrement, un client du tailnet situé hors du LAN résout l'IP publique et arrive chez Traefik avec son IP Internet : il est refusé. Après une modification du fichier à la main, redémarrer Headscale :

    ```bash
    docker restart headscale-i136ix2bmrrbeovnyrh1o72w
    ```
  </Step>
  <Step title="Identifier le conteneur et son port interne">
    ```bash
    docker ps --format "table {{.Names}}\t{{.Ports}}" | grep -i <service>
    docker inspect <conteneur> --format '{{.HostConfig.NetworkMode}}'
    ```
    Utiliser le port interne du conteneur, pas un port publié sur l'hôte. Si le conteneur partage le réseau d'un autre (`network_mode: service:…`, cas de qBittorrent derrière Gluetun), la cible est le conteneur qui porte le réseau.
  </Step>
  <Step title="Vérifier que Traefik peut joindre le conteneur">
    Le conteneur doit être attaché au réseau Docker `coolify`.
  </Step>
  <Step title="Déclarer le routeur et le service">
    Dans Coolify, **Servers → Proxy → Dynamic configurations**, éditer `vpn-only.yaml` :

    ```yaml
    http:
      routers:
        <service>-admin:
          rule: "Host(`<sous-domaine>.ims-world.fr`)"
          entryPoints: [https]
          service: <service>-admin
          middlewares: [vpn-only, admin-gzip]
          tls: {certResolver: letsencrypt}
      services:
        <service>-admin:
          loadBalancer:
            servers:
              - url: "http://<conteneur>:<port-interne>"
    ```

    Si le service a aussi besoin d'un forward-auth Authentik, l'ajouter ici avec le suffixe `@docker` (par exemple `authentik-dozzle@docker`) : les labels du conteneur sont ignorés dès qu'un routeur du fichier porte le même nom d'hôte.
  </Step>
  <Step title="Retirer le domaine de Coolify">
    Dans la ressource, vider le champ **Domains**, retirer les labels de routage Traefik du compose, puis redéployer.

    <Warning>
    Un domaine laissé dans Coolify crée un second routeur, sans `vpn-only`, qui peut rendre le service public.
    </Warning>
  </Step>
  <Step title="Mettre à jour la matrice">
    Ajouter la ligne du service dans la [matrice d'exposition](/reseau/matrice-securite-exposition), zone tailnet.
  </Step>
</Steps>

## Vérification

```bash
# Aucun label de routage ne doit rester
docker inspect <conteneur> | grep "traefik.http.routers"

# Hors tailnet (partage de connexion du téléphone)
curl -sI https://<sous-domaine>.ims-world.fr | head -1   # attendu : 403

# Depuis le tailnet
curl -sI https://<sous-domaine>.ims-world.fr | head -1   # attendu : 200 ou redirection
```

<Check>
Le navigateur hors tailnet affiche la page « Page non trouvée » ; le code HTTP reste 403.
</Check>

## Retour arrière

Retirer le routeur et le service de `vpn-only.yaml`, puis remettre le domaine dans Coolify et redéployer.

## Dépannage

Refus depuis le tailnet : lire l'IP source dans les journaux de Traefik.

```bash
docker logs coolify-proxy --tail 50 | grep " 403 "
```

| IP vue par Traefik | Cause | Solution |
| --- | --- | --- |
| IP publique du client | Le nom n'est pas dans les `extra_records` | Étape 1 |
| `10.0.1.1` (pont Docker) | SNAT Tailscale réactivé, ou `userland-proxy` actif | `sudo tailscale set --snat-subnet-routes=false` ; vérifier `/etc/docker/daemon.json` |

Si `sudo tcpdump -i tailscale0 port 443` montre la bonne IP mais que Traefik voit `10.0.1.1`, voir l'[incident du 18/08/2026](/history/incidents/2026-08-18-blocage-traefik-vpn-only-docker-proxy).
