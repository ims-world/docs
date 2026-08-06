---
title: "Mac Mini 2014 (Hôte Standby)"
description: "Ancien hôte de production — serveur de secours, agent Tailnet et fallback"
icon: "apple"
iconType: "duotone"
---

import { ips } from "/snippets/variables.mdx";

<Badge color="amber">🟡 Standby Chaud (Machine de Secours)</Badge>

## Rôle & Emplacement

<Info>
Le **Mac Mini 2014** a été l'hôte principal de l'infrastructure. Aujourd'hui, il est conservé en **mode Standby chaud** dans le rack [Labrax](/infrastructure/labrax) pour servir de nœud de secours et de fallback en cas de panne physique de l'hyperviseur principal.
</Info>

## Fiche Technique

| Propriété | Valeur |
|---|---|
| **Matériel** | Apple Mac Mini (Late 2014) |
| **Processeur** | Intel Core i5 Dual-Core |
| **Stockage** | SSD SATA 500 Go |
| **Réseau LAN** | Ethernet Gigabit `192.168.1.x` |
| **Tailscale IP** | `{ips.macmini}` |
| **Hostname Tailnet** | `macmini-standby` / `coolify-old.ims-world.fr` |
| **Port SSH Réel** | **`4242`** (port `22` occupé par Endlessh tarpit) |
| **Statut** | <Badge color="amber">🟡 Standby Chaud</Badge> |

## Particularités & Piège SSH (Endlessh)

<Warning>
**Piège du port SSH 22** : Le port 22 standard sur le Mac Mini est volontairement routé vers **Endlessh** (tarpit anti-bot qui piège les scans automatisés en leur envoyant des bannières SSH infinies).

Pour vous connecter en SSH au Mac Mini, spécifier impérativement le port **`4242`** :
```bash
ssh -p 4242 cmolotkoff@{ips.macmini}
```
</Warning>

## Rôle de Secours & Redirection DNS

Pendant la phase de validation post-cutover, l'accès à l'ancienne instance Coolify du Mac Mini est conservé via l'enregistrement DNS intermédiaire dans [Headscale](/services/headscale-headplane) :

```yaml
extra_records:
  - name: "coolify-old.ims-world.fr"
    value: "{ips.macmini}"
```

## Procédure de Bascule d'Urgence (Fallback)

En cas de crash physique majeur de l'hyperviseur MS-01 :
<Steps>
  <Step title="Vérification de l'alimentation">
    S'assurer que le Mac Mini est sous tension dans le rack [Labrax](/infrastructure/labrax).
  </Step>
  <Step title="Reprise du Port Forward Bbox">
    Rediriger le port-forwarding de la Bbox (ports 80 & 443) vers l'IP LAN du Mac Mini.
  </Step>
  <Step title="Démarrage des conteneurs secours">
    Relancer les conteneurs Docker de secours archivés sur le SSD du Mac Mini.
  </Step>
</Steps>
