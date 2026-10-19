---
title: "ADR-011 : Organisation d'IMS-INFRA par Lots"
description: "Structuration du dépôt IMS-INFRA en lots documentés avec un Makefile comme point d'entrée, et règles de connexion d'Ansible"
icon: "file-code"
iconType: "duotone"
last_reviewed: "2026-10-19"
---

<Badge color="green">🟢 Décision Acceptée & Implémentée</Badge>

## Contexte & Enjeu

Le dépôt IMS-INFRA avait été conçu pour des VMs issues du template 9000, qui ne sont jamais entrées en service. La production (nœuds Proxmox, LXC, VM 104) a été configurée à la main. En conséquence :
- Le groupe d'inventaire `[vms]` ne contenait qu'une adresse morte ;
- Plusieurs rôles ne servaient à rien (`node_exporter`, alors que la supervision utilise Alloy ; `nfs_client`, sans client NFS géré) ;
- Les playbooks de durcissement ciblaient `all`, et auraient appliqué `PermitRootLogin no` aux nœuds Proxmox dès l'ajout de ces derniers à l'inventaire ;
- Les runbooks pointaient vers des chemins absolus d'un ancien emplacement ;
- Il n'était plus possible de savoir ce qui était scripté et ce qui ne l'était pas.

---

## Décisions Actées

1. **Organisation par lots** : Chaque fonctionnalité est un dossier `lots/NN-<nom>/` avec son `README.md`. Le `README.md` racine sert de sommaire, avec une colonne de statut.
2. **Un seul point d'entrée (le Makefile)** : Chaque action a une cible (`make help`). Les commandes destructrices exigent `CONFIRM=oui`.
3. **Script ou Ansible** : Script shell pour une action ponctuelle sur une machine, Ansible pour un état à maintenir sur plusieurs hôtes.
4. **Connexion d'Ansible** : Compte `cmolotkoff` + sudo, jamais root. IP Tailscale pour les hôtes du tailnet, rebond par pve pour les LXC sans Tailscale. `host_key_checking` activé.
5. **Périmètre des playbooks** : Le durcissement (`security.yml`) ne cible que `[vms]`. Chaque playbook cible un groupe explicite, jamais `all`.
6. **Qualité** : `ansible-lint` en profil production, simulation `--check --diff` avant toute application, échec explicite si une variable obligatoire manque.
7. **Documentation à deux niveaux** : Le dépôt donne les commandes, Mintlify explique le contexte et les choix (section [Infrastructure as Code](/iac/index)).

---

## Conséquences

- Le code mort est retiré (`node_exporter`, `nfs_client`, `monitoring.yml`, `storage.yml`, anciens runbooks). Il sera recréé dans un lot dédié si le besoin réapparaît.
- Le durcissement SSH et fail2ban des hôtes existants reste manuel. Il n'est pas repris par Ansible tant qu'un audit n'a pas confirmé qu'un rôle reproduit exactement l'état actuel.
- L'agent Alloy reste installé à la main : candidat pour un futur lot « Supervision ».
- Ajouter une fonctionnalité impose de créer son lot, ses cibles make, sa ligne au sommaire et sa page Mintlify.

---

## Alternatives Écartées

- **Refonte complète en Infrastructure as Code** (audit puis convergence de toute la couche système) : plus ambitieuse, mais disproportionnée pour le besoin actuel. Elle reste possible lot par lot.
- **Tests Molecule** : coût de maintenance élevé pour six hôtes. La simulation `--check --diff` sur le vrai parc est préférée.
- **Terraform** : écarté de longue date, les VMs ne sont pas assez volatiles pour le justifier.
