---
title: "Incident — DNS Wildcard Shadowing sur share.ims-world.fr"
description: "Erreur 503 causée par le masquage du wildcard DNS OVH dû aux enregistrements résiduels _acme-challenge TXT"
icon: "globe"
iconType: "duotone"
---

<Badge color="green">🟢 Résolu — Enregistrements ACME TXT Nettoyés sur OVH</Badge> _(2026-08-24)_

_Rédigé le 24/08/2026 — Zone `ims-world.fr` (OVH)._

---

## Symptôme Initial

Zipline (conteneur `zipline-kbcknnnkswmcnlgmupxoyheh`) démarre normalement (logs applicatifs propres, healthcheck OK), mais `share.ims-world.fr` renvoie **503 "no available server"** dans le navigateur.

---

## Fausses Pistes Explorées (dans l'Ordre)

1. **CrowdSec / Bouncer Traefik** — Écarté d'emblée : un `503` générique n'est pas la signature d'un blocage CrowdSec (qui produit un `403`, avec trace explicite dans les access logs Traefik, `backend "-"`).
2. **Backend Zipline Injoignable** — Écarté : test direct depuis le conteneur Traefik (`wget` vers `zipline-kbcknnnkswmcnlgmupxoyheh:3000`) répond `200 OK`. Le conteneur est sur le bon réseau Docker (`kbcknnnkswmcnlgmupxoyheh`), labels Traefik corrects (port 3000, règle `Host` correcte), Traefik a bien ce réseau attaché.
3. **Traefik n'a pas Rechargé la Config** — Écarté : API Traefik (`/api/http/routers`) inaccessible en `404` car `--api.insecure=false` (comportement normal en prod Coolify, pas une anomalie).
4. **Réseau Local (Wi-Fi, Résolveur Bloqué)** — Écarté après test : `nslookup auth.ims-world.fr` résout correctement via le resolver Tailscale (`100.100.100.100`). Les premiers `dig @8.8.8.8` en timeout étaient dus à Tailscale interceptant le port 53 sortant, pas à un vrai blocage réseau.

---

## Cause Racine Identifiée

**Résolution DNS cassée pour `share.ims-world.fr` uniquement**, malgré un wildcard `*.ims-world.fr` actif et fonctionnel sur la zone (confirmé via un sous-domaine aléatoire qui résout vers `176.151.43.50`).

`dig share.ims-world.fr CNAME` / `AAAA` retournent `NOERROR` avec 0 réponse et le SOA en `AUTHORITY` — signature d'un **nœud DNS existant mais vide** (_empty non-terminal_) pour `share`, pas d'une absence totale du nom.

En cause : trois enregistrements résiduels dans la zone OVH :

```dns
_acme-challenge.share    120 IN TXT  "hkv3VIo_..."
_acme-challenge.share    120 IN TXT  "HEzpUrsiz1..."
_acme-challenge.share    120 IN TXT  "2AAxLaW9..."
```

`_acme-challenge.share` est un sous-domaine de `share` : sa seule présence dans la zone crée un nœud vide pour `share.ims-world.fr` lui-même. Or **un wildcard ne s'applique jamais si le nom exact existe déjà dans la zone**, même vide (comportement standard RFC DNS, pas un bug OVH). Résultat : `share` sort du champ du wildcard et ne résout plus vers `176.151.43.50`, d'où le 503 côté navigateur (résolution DNS échouée, remontée par certains navigateurs comme une erreur serveur générique plutôt qu'une erreur DNS explicite).

**Origine des 3 enregistrements** : résidus de plusieurs cycles de validation ACME DNS-01 (Traefik/Let's Encrypt, certresolver `letsencrypt`) jamais nettoyés après émission du certificat — 3 tokens différents = 3 tentatives successives. Le certificat existant pour `share` fonctionnait déjà (trafic HTTPS normal loggé le 23/08 avant l'incident), ces TXT résiduels ne servaient donc plus à rien.

---

## Portée du Problème

Même schéma potentiellement présent sur d'autres sous-domaines de la zone ayant un `_acme-challenge.<nom>` sans A/CNAME propre par ailleurs : `radarr`, `system`, `tools`, `coolify-old`. À valider après le fix (voir ci-dessous).

**Précision** : `docs.ims-world.fr` (déploiement Mintlify) est bien sur la même zone `ims-world.fr` — voir `_acme-challenge.docs` dans le Fix Appliqué ci-dessous. Cet enregistrement résiduel n'a jamais cassé la résolution (le CNAME propre de `docs` vers `cname.mintlify.builders.` prime sur le nœud vide), mais a été nettoyé par précaution en même temps que les autres résidus.

---

## Fix Appliqué

Suppression des enregistrements `_acme-challenge.*` résiduels dans la zone OVH `ims-world.fr` :

- `_acme-challenge.share` (x3)
- `_acme-challenge.docs` (safe, pas cassé — `docs` a un CNAME actif vers `cname.mintlify.builders.`)
- `_acme-challenge.radarr`
- `_acme-challenge.system`
- `_acme-challenge.tools`
- `_acme-challenge.coolify-old`

---

## Validation Post-Fix

```bash
dig +short share.ims-world.fr
dig +short radarr.ims-world.fr
dig +short system.ims-world.fr
dig +short tools.ims-world.fr
```

**Résultat attendu** : chaque commande retourne `176.151.43.50` (résolution wildcard restaurée). Si `radarr`/`system`/`tools` se mettaient également à résoudre alors qu'ils ne le faisaient pas avant l'incident, cela confirmerait qu'ils étaient cassés silencieusement par le même mécanisme, sans qu'aucun 503 n'ait encore été remarqué dessus.

---

## Leçon / Piège à Retenir

<Warning>
  **Un enregistrement `_acme-challenge.<nom>` non nettoyé après validation ACME peut masquer un wildcard DNS pour `<nom>` lui-même**, même si un certificat valide existe déjà et que le service fonctionnait normalement jusque-là. Symptôme trompeur côté client : 503 générique plutôt qu'une erreur de résolution DNS explicite.
</Warning>

**Action de fond à envisager** : automatiser le nettoyage des TXT `_acme-challenge.*` après émission (hook post-validation côté Traefik/ACME), ou vérifier périodiquement la zone OVH pour des résidus — plutôt que de découvrir le problème un service à la fois.
