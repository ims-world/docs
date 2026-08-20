---
title: "Sécuriser un Service avec vpn-only (File Provider)"
description: "Procédure pas-à-pas pour restreindre l'accès HTTPS d'un sous-domaine au seul réseau privé Tailscale via vpn-only.yaml"
icon: "shield-check"
iconType: "duotone"
---

<Badge color="green">🟢 Procédure Standard Validée</Badge>

## Objectif

Cette procédure décrit la séquence exacte pour ajouter un nouveau service ou une nouvelle interface web d'administration au filtrage privé **`vpn-only`** (restreint au réseau VPN Overlay Tailscale `100.64.0.0/10`).

Elle applique l'architecture centralisée validée dans l'[ADR-009](/history/adr/adr-009-bug-docker-proxy-middleware-vpn-only).

---

## Procédure Pas-à-Pas (9 Étapes)

<Steps>
  <Step title="Étape 1 — Identifier le conteneur cible réel">
    Vérifier le nom de réseau et le mode d'isolation du conteneur en SSH sur la VM Coolify :
    ```bash
    docker inspect <nom-conteneur> | grep -A5 '"Networks"'
    docker inspect <nom-conteneur> | grep NetworkMode
    ```
    <Warning>
    **Cas des conteneurs en `network_mode: service:...`** : Si un conteneur (ex. `qBittorrent`) partage l'espace de nommage réseau d'un conteneur VPN (`Gluetun`), il n'a aucune IP propre. Le nom de conteneur cible à renseigner dans Traefik est **le conteneur VPN porteur de la pile réseau** (`gluetun-...`), pas le conteneur applicatif.
    </Warning>
  </Step>

  <Step title="Étape 2 — Confirmer le nom exact et le port interne">
    ```bash
    docker ps --format "table {{.Names}}\t{{.Ports}}" | grep -i <service>
    ```
    Le port à utiliser dans Traefik est le **port interne du conteneur** (ex. `8080`, `8989`, `3000`), et non le port éventuellement publié sur l'hôte.
  </Step>

  <Step title="Étape 3 — Vérifier l'attachement au réseau coolify">
    S'assurer que le conteneur cible est attaché au réseau externe `coolify` pour que le proxy `coolify-proxy` puisse le joindre par son nom DNS interne Docker.
  </Step>

  <Step title="Étape 4 — Ajouter le bloc Router & Service dans vpn-only.yaml">
    Dans l'IHM Coolify (**Proxy → Dynamic configurations → éditer `vpn-only.yaml`**), ajouter les blocs sous `routers` et `services` :

    ```yaml
    http:
      routers:
        # ... routers existants ...
        <service>-admin:
          rule: "Host(`<sous-domaine>.ims-world.fr`)"
          entryPoints: [https]
          service: <service>-admin
          middlewares: [vpn-only, admin-gzip]
          tls: {certResolver: letsencrypt}

      services:
        # ... services existants ...
        <service>-admin:
          loadBalancer:
            servers:
              - url: "http://<nom-conteneur-exact>:<port-interne>"
    ```
  </Step>

  <Step title="Étape 5 — Retirer le domaine de l'UI Coolify (Obligatoire)">
    Dans l'interface Coolify sur la ressource concernée (**Domains / Manage domains**), **supprimer le nom de domaine**.
    <Warning>
    **Suppression obligatoire** : Si le domaine reste saisi dans l'UI Coolify, Coolify génère un routeur automatique parallèle sans le middleware `vpn-only`, créant une concurrence de routeurs non déterministe qui peut ré-exposer le service publiquement.
    </Warning>
  </Step>

  <Step title="Étape 6 — Nettoyer les labels Traefik du docker-compose.yml">
    Dans le fichier Compose du service, supprimer tous les labels Traefik de routage (`traefik.http.routers...`). Conserver uniquement la configuration applicative brute (volumes, variables d'environnement, santé).
  </Step>

  <Step title="Étape 7 — Redéployer la ressource concernée">
    Déployer le service depuis l'IHM Coolify pour appliquer le retrait du domaine et du réseau.
  </Step>

  <Step title="Étape 8 — Vérifier l'absence de labels résiduels">
    ```bash
    docker inspect <nom-conteneur> | grep "traefik.http.routers"
    ```
    Cette commande ne doit **rien retourner** (le routage est entièrement assuré par le provider file `vpn-only.yaml`).
  </Step>

  <Step title="Étape 9 — Validation des accès WAN vs Tailnet">
    Exécuter les tests de validation d'étanchéité :
    ```bash
    # 1. Test depuis le WAN (4G/5G mobile, hors VPN)
    curl -sI https://<sous-domaine>.ims-world.fr | head -1
    # → Résultat attendu : HTTP/1.1 403 Forbidden

    # 2. Test depuis un appareil connecté au Tailnet
    curl -sI https://<sous-domaine>.ims-world.fr | head -1
    # → Résultat attendu : HTTP/1.1 200 OK (ou 302 Redirect)
    ```
  </Step>
</Steps>

---

## 🛠️ Dépannage Réseau (En cas de 403 Forbidden persistant depuis le Tailnet)

Si l'accès est rejeté en **HTTP 403** même pour un utilisateur authentifié sur le Tailnet post-reboot :

1. **Vérifier le statut SNAT Tailscale** :
   ```bash
   sudo tailscale status --json | grep -i snat
   # Si "SNATSubnetRoutes": true -> Exécuter :
   sudo tailscale set --snat-subnet-routes=false
   ```
2. **Vérifier l'arrivée des paquets intacts sur l'interface VPN** :
   ```bash
   sudo tcpdump -i tailscale0 port 443
   ```
   *Si `tcpdump` affiche votre vraie IP Tailscale (`100.64.0.x`) mais que Traefik reçoit `10.0.1.1`*, le masquage survient dans les règles NAT `POSTROUTING`. Se référer au [Post-Mortem du 18-20/08/2026](/history/incidents/2026-08-18-blocage-traefik-vpn-only-docker-proxy).
