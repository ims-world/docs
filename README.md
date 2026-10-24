# Documentation du homelab IMS-WORLD

Sources de la documentation publiée sur [docs.ims-world.fr](https://docs.ims-world.fr), construite avec Mintlify.

- Contenu : pages MDX, navigation dans `docs.json`.
- Règles de rédaction et plans de page : page « Contribuer et guide de style » (`history/comment-fonctionne-cette-doc.md`).
- Modèles de page : `_modeles/` (non publiés).
- Aperçu local : `npx -y mint@4.2.997 dev`.
- Contrôles (identiques en CI) : `npx -y mint@4.2.997 validate` et `npx -y mint@4.2.997 broken-links`.

L'administration du parc (Ansible, `just`) vit dans le dépôt ims-ops.
