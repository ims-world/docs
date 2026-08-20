---
title: "Incident — Double Blocage HTTP 403 du Middleware vpn-only (docker-proxy & Tailscale SNAT)"
description: "Analyse des deux pannes successives de masquage d'IP source (userland-proxy Docker & ts-postrouting Tailscale SNAT) et résolutions définitives"
icon: "shield-exclamation"
iconType: "duotone"
---

<Badge color="green">🟢 Résolu & Fix Définitif Implémenté</Badge> *(2026-08-19)*

---

## Synthèse de l'Incident

Deux dysfonctionnements distincts provoquant exactement le même symptôme applicatif (**HTTP 403 Forbidden** systématique sur les services filtrés `vpn-only` Traefik) sont survenus coup sur coup :

1. **Phase 1 (18/08/2026)** : Masquage de l'IP cliente par le relais applicatif **`docker-proxy`** (`userland-proxy`).
2. **Phase 2 (19-20/08/2026 Post-Reboot MS-01)** : Masquage de l'IP cliente par les règles **`ts-postrouting` SNAT de Tailscale** sur le trafic forwardé vers les bridges Docker.

---

## Phase 1 — Incident `docker-proxy` (18/08/2026)

### Symptôme
Lors des accès aux services d'administration (`qbit`, `radarr`, `sonarr`, `prowlarr`, `monitoring`), l'intégralité des requêtes des utilisateurs légitimes du VPN Tailscale (`100.64.0.0/10`) était rejetée en `HTTP 403 Forbidden`.

### Cause Racine (Root Cause)
Le démon Docker s'exécutait avec le paramètre par défaut **`userland-proxy: true`**. Lorsqu'un port est publié (`-p 80:80`), le binaire `docker-proxy` intercepte le trafic et ouvre une nouvelle connexion TCP vers Traefik sourcée depuis l'IP de la passerelle du bridge Docker (**`10.0.1.1`**). Traefik ne recevant que `10.0.1.1`, le middleware `vpn-only` rejetait tout le monde.

### Fix Phase 1 (CIS Docker Benchmark)
Inscription de `"userland-proxy": false` dans `/etc/docker/daemon.json` sur la VM Coolify. Conforme au **CIS Docker Benchmark (Section 2.12)**, Docker bascule le routage sur les règles noyau iptables `DNAT` + `MASQUERADE` et `net.ipv4.route_localnet`.

---

## Phase 2 — Incident Post-Reboot SNAT Tailscale (19-20/08/2026)

### Symptôme
Après un redémarrage complet (*cold reboot*) du serveur physique MS-01, le filtrage `vpn-only` a de nouveau bloqué tout le monde en `HTTP 403 Forbidden`.

### Démarche Diagnostic Réseau
1. **Validation du Tunnel Client** :
   ```bash
   # Capture sur l'interface Tailscale pendant une requête HTTP
   sudo tcpdump -i tailscale0 port 443
   ```
   *Résultat* : Le paquet arrive intact sur l'interface `tailscale0` avec la **vraie** IP source du client Tailscale (ex: `100.64.0.3`).
2. **Logs Traefik** :
   Les logs Traefik affichent toujours l'IP source `10.0.1.1` (passerelle du bridge `coolify`).
3. **Conclusion** : Le paquet arrive intact sur `tailscale0`, mais la réécriture d'adresse (SNAT) se produit *après*, entre l'entrée sur l'interface VPN et l'arrivée au conteneur Traefik.

### Cause Racine (Root Cause)
Le démon `tailscaled` installe sa propre chaîne iptables NAT nommée **`ts-postrouting`** avec une règle basée sur un marquage de paquets (*fwmark*) :

```text
Chain ts-postrouting (1 references)
 pkts bytes target     prot opt in     out     source          destination
   15   960 MASQUERADE  0    --  *     *      0.0.0.0/0        0.0.0.0/0    mark match 0x40000/0xff0000
```

Tailscale applique un `fwmark` sur tout paquet qu'il **route/forward vers un autre réseau** (conçu à l'origine pour les subnet routers).
Notre trafic correspond exactement à ce cas :
1. Le paquet entre sur `tailscale0` à destination de la VM (`100.64.0.4`).
2. La règle `DNAT` de Docker redirige le paquet vers l'IP du conteneur Traefik (`10.0.1.2` sur le bridge `coolify`).
3. Le paquet changeant d'interface réseau de sortie, Tailscale applique la règle `ts-postrouting MASQUERADE`, réécrivant l'IP source du client avec l'IP locale du bridge (`10.0.1.1`) avant qu'il n'atteigne Traefik.

Ce comportement agit au niveau des chaînes iptables de Tailscale et est **totalement indépendant** du paramètre `userland-proxy` de Docker.

### Fix Phase 2
Exécution de la commande de configuration Tailscale :

```bash
sudo tailscale set --snat-subnet-routes=false
```

Cette commande désactive le masque SNAT automatique de Tailscale sur le trafic d'inter-réseaux forwardé, conservant l'IP source d'origine intacte (`100.64.0.x`) jusqu'au conteneur Traefik.

---

## Pistes Écartées lors du Diagnostic

Avant d'isoler la chaîne `ts-postrouting`, une règle `DNAT` manuelle avait été insérée en test dans la chaîne `PREROUTING` :

```bash
sudo iptables -t nat -I PREROUTING -i tailscale0 -p tcp --dport 443 -j DNAT --to-destination 10.0.1.2:443
sudo iptables -t nat -I PREROUTING -i tailscale0 -p tcp --dport 80 -j DNAT --to-destination 10.0.1.2:80
```

*Résultat* : Cette règle acheminait correctement le trafic (compteurs `pkts` > 0), mais ne résolvait pas le masquage d'IP (qui survient ultérieurement lors de la phase `POSTROUTING`). **Cette règle manuelle a été immédiatement retirée**, s'avérant inutile une fois la commande `tailscale set --snat-subnet-routes=false` appliquée.

---

## Hypothèse de l'Apparition Post-Reboot

Le fix `userland-proxy: false` initial avait été appliqué via un simple `systemctl restart docker` (sans reboot de l'hôte). Lors du reboot complet du MS-01, les démons `docker` et `tailscaled` ont démarré à froid en parallèle. L'ordre d'insertion de la chaîne `ts-postrouting` par rapport aux chaînes Docker dans la table iptables `POSTROUTING` a pu varier, faisant basculer la correspondance `fwmark`. L'application de `--snat-subnet-routes=false` garantit un comportement déterministe et explicite quel que soit l'ordre de boot.

---

## Check-List de Vérification & Persistance Post-Reboot

### 1. Vérification de l'État Tailscale
```bash
# Vérifier que le réglage SNAT est désactivé
sudo tailscale status --json | grep -i snat
# -> Attendu : "SNATSubnetRoutes": false
```

### 2. Validation de la Persistance au Redémarrage
- **Restart du démon `tailscaled`** : `sudo systemctl restart tailscaled` → ✅ **Persistance validée**.
- **Reboot Physique du MS-01** : Inscrit au plan de maintenance pour validation définitive lors de la prochaine fenêtre.

### 3. Tests de Connectivité Réels
```bash
# Depuis un appareil authentifié sur le Tailnet (ex. 100.64.0.3)
curl -sI https://monitoring.ims-world.fr
curl -sI https://coolify.ims-world.fr
# -> Attendu : HTTP 200 OK (IP source préservée dans les logs Traefik)
```
