---
title: "Ajouter un Équipement sur Headscale"
description: "Procédure d'enrôlement VPN Tailnet : profil Utilisateur Classique (OIDC SSO) vs profil Nœud Infrastructure (Pre-Auth Key)"
icon: "network-wired"
iconType: "duotone"
last_reviewed: "2026-08-23"
---

import { ips, domains } from "/snippets/variables.mdx";

<Badge color="green">🟢 Procédure Opérationnelle Validée</Badge>

<Info>
Cette procédure décrit les étapes exactes pour enregistrer un nouvel équipement (ordinateur, smartphone, serveur physique ou VM) sur le réseau overlay VPN **Headscale** de l'infrastructure IMS-WORLD.
</Info>

---

## 📌 Choix du Profil d'Enrôlement

Avant d'enregistrer une machine, déterminer le profil approprié :

| Profil | Type d'Équipement | Méthode d'Authentification | Exemples |
|---|---|---|---|
| **Utilisateur Classique** | Appareils personnels & mobiles avec navigateur Web | **Interactive via Authentik SSO OIDC** (`auth.ims-world.fr` + 2FA TOTP) | Smartphones (iOS/Android), Laptops, PC de bureau |
| **Profil `infrastructure`** | Serveurs physiques & virtuels headless (automatisés) | **Non-interactive via Clé de Pré-Authentification** (*Pre-Auth Key*) | MS-01, VM 104 Coolify, PBS 103, Mac Mini, RPi, NAS |

---

## 📱 Option A : Enrôler un Appareil "Utilisateur Classique" (OIDC Browser Login)

Ce mode est destiné aux utilisateurs humains sur des appareils disposant d'un navigateur Web.

<Steps>
  <Step title="Installation du Client Tailscale Native">
    Télécharger et installer le client officiel Tailscale sur l'appareil cible :
    - **macOS / iOS / Android / Windows** : Télécharger l'application officielle sur l'App Store / Play Store ou le site officiel Tailscale.
    - **Linux avec IHM** : Installer le paquet officiel `tailscale`.
  </Step>

  <Step title="Configuration du Serveur de Coordination Headscale">
    - **Sur l'Application Mobile / GUI (macOS, iOS, Windows, Android)** :
      1. Ouvrir l'application Tailscale.
      2. Accéder aux **Paramètres** (icône d'engrenage) ➔ **Accounts** ➔ **Add Account** ou **Change Server**.
      3. Définir l'URL du serveur de coordination custom :
         ```text
         https://vpn.ims-world.fr
         ```
    - **Sur Linux Desktop / macOS via Terminal CLI** :
      ```bash
      tailscale up --login-server https://vpn.ims-world.fr --accept-routes
      ```
  </Step>

  <Step title="Authentification SSO Authentik">
    1. L'application ou le terminal ouvre automatiquement une page Web pointant vers `vpn.ims-world.fr/register/...`.
    2. Redirection automatique vers le portail **Authentik SSO** (`https://auth.ims-world.fr`).
    3. Se connecter avec votre compte utilisateur personnel (`cmolotkoff`) et valider le **2FA TOTP**.
    4. Valider l'enregistrement de l'appareil. Le client Tailscale passe immédiatement au statut **Connected**.
  </Step>
</Steps>

---

## 🖥️ Option B : Enrôler un Serveur "Infrastructure" (Pre-Auth Key)

Ce mode est indispensable pour les serveurs sans interface graphique (*headless*) qui doivent se reconnecter automatiquement au démarrage sans intervention humaine.

<Steps>
  <Step title="Générer une Clé de Pré-Authentification (Pre-Auth Key)">
    La clé doit être associée à l'utilisateur Headscale dédié **`infrastructure`**.

    <Info>
    **💡 Que signifie l'Expiration d'une Pre-Auth Key ?**
    L'expiration s'applique **uniquement à la fenêtre de temps pendant laquelle le jeton (la chaîne de texte `authkey`) peut être utilisé pour effectuer le premier `tailscale up`**.
    - **Une fois la machine enregistrée**, sa connexion avec Headscale est **permanente et illimitée dans le temps**. 
    - L'expiration de la clé **n'aura AUCUN impact sur les machines déjà enrôlées** (elles ne seront JAMAIS déconnectées lorsque la clé expire).
    - Cela signifie simplement qu'au-delà de cette durée (ex: 90j), le jeton ne pourra plus servir à enregistrer de *nouveaux* serveurs supplémentaires.
    </Info>

    <Tabs>
      <Tab title="🌐 Via la Console Web Headplane">
        1. Se connecter à la console **Headplane Admin** : `https://admin.vpn.ims-world.fr/admin` *(suffixe `/admin` obligatoire)*.
        2. Dans le menu de gauche, sélectionner l'utilisateur **`infrastructure`**.
        3. Accéder à l'onglet **Pre-Auth Keys**.
        4. Cliquer sur **Create Pre-Auth Key** :
           - **Expiration** : Sélectionner `90d` ou `999d` (délai disponible pour réaliser le premier enregistrement).
           - **Reusable** : Cocher si vous prévoyez d'enregistrer plusieurs serveurs différents avec le même jeton.
        5. Copier la clé générée (ex: `a1b2c3d4e5f6...`).
      </Tab>
      <Tab title="⚡ Via la CLI Headscale (VM Coolify)">
        Exécuter la commande dans le conteneur Headscale sur la VM 104 :
        ```bash
        # Se connecter en SSH à la VM Coolify
        ssh cmolotkoff@100.64.0.4

        # Générer une Pre-Auth Key pour l'utilisateur infrastructure
        docker exec -it headscale-i136ix2bmrrbeovnyrh1o72w headscale preauthkeys create --user infrastructure --expiration 999d
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="Exécuter l'Enregistrement sur le Cibles">
    Selon que l'équipement cible est un serveur hôte physique/VM ou un conteneur Docker dédié, adapter le mode d'exécution :

    <Tabs>
      <Tab title="🖥️ Sur Serveur Physiques ou VM (Host Linux / Proxmox)">
        Se connecter en SSH au serveur hôte et exécuter directement la CLI Tailscale :
        ```bash
        sudo tailscale up \
          --login-server https://vpn.ims-world.fr \
          --authkey <VOTRE_PRE_AUTH_KEY> \
          --accept-routes \
          --snat-subnet-routes=false
        ```
      </Tab>
      <Tab title="🐳 Dans un Conteneur Docker (Docker Tailscale Client)">
        Si le client Tailscale s'exécute dans un conteneur Docker (ex: stack Coolify / Compose) :
        ```bash
        # 1. Identifier le nom exact du conteneur client Tailscale
        docker ps --format '{{.Names}}' | grep tailscale

        # 2. Exécuter l'enrôlement à l'intérieur du conteneur client
        docker exec -it <nom_conteneur_tailscale> tailscale up \
          --login-server https://vpn.ims-world.fr \
          --authkey <VOTRE_PRE_AUTH_KEY> \
          --accept-routes \
          --snat-subnet-routes=false
        ```
      </Tab>
    </Tabs>

    <Info>
    **Détail des flags réseau** :
    - `--login-server https://vpn.ims-world.fr` : Pointeur vers le serveur de coordination Headscale.
    - `--snat-subnet-routes=false` : Empêche Tailscale de masquer la vraie IP source lors du routage réseau.
    - `--accept-routes` : Accepte les routes propagées sur le Tailnet.
    </Info>
  </Step>

  <Step title="Vérification de l'Enrôlement">
    Vérifier le statut du nœud et l'attribution de son IP privée dans la plage `100.64.0.0/10` :

    ```bash
    # Sur l'hôte physique / VM
    tailscale status
    tailscale ip -4

    # Dans un conteneur client Tailscale
    docker exec -it <nom_conteneur_tailscale> tailscale status

    # Via le serveur Headscale (sur la VM 104 - UUID i136ix2bmrrbeovnyrh1o72w)
    docker exec -it headscale-i136ix2bmrrbeovnyrh1o72w headscale nodes list
    ```
  </Step>
</Steps>

---

## ⚙️ Étape Post-Configuration : Split-Horizon DNS (`extra_records`)

Si le nouvel équipement enrôlé héberge un service privé qui doit être protégé par le middleware Traefik `vpn-only.yaml` :

1. Éditer le fichier de configuration Headscale sur la VM 104 (`/data/coolify/services/i136ix2bmrrbeovnyrh1o72w/config/config.yaml`).
2. Ajouter le nom de domaine FQDN et l'IP Tailscale du nœud sous la section `extra_records` :
   ```yaml
   extra_records:
     - name: "nouveau-service.ims-world.fr"
       value: "100.64.0.X"
   ```
3. Redémarrer le conteneur Headscale :
   ```bash
   docker restart headscale-i136ix2bmrrbeovnyrh1o72w
   ```

---

<CardGroup cols={2}>
  <Card title="Headscale & Headplane" icon="network-wired" href="/services/headscale-headplane">
    Fiche service et console de gestion du réseau VPN Overlay.
  </Card>
  <Card title="Sécuriser un Service vpn-only" icon="shield" href="/procedures/securiser-service-vpn-only">
    Procédure d'isolation des services privés d'administration.
  </Card>
</CardGroup>
