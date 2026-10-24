# Consignes pour les assistants IA

Documentation Mintlify du homelab IMS-WORLD : pages MDX avec frontmatter YAML, navigation dans `docs.json`, variables dans `snippets/variables.mdx`.

Le guide complet est la page `history/comment-fonctionne-cette-doc.md` (« Contribuer et guide de style »). Le lire avant toute modification. Partir des modèles de `_modeles/` pour une nouvelle page.

## Règles impératives

- Français, forme impersonnelle : jamais « tu », « vous » ni « nous ». Étapes à l'infinitif.
- Aucun emoji. Titres en français, majuscule au premier mot seulement.
- Plans par type de page : respecter l'ordre et les intitulés exacts du guide.
- Une donnée à un seul endroit ; ailleurs, un lien.
- Aucun secret, même partiel. Placeholders entre chevrons : `<jeton>`, `<hôte>`.
- Variables : `{ips.x}` dans le texte, `<code>{ips.x}</code>` dans du code en ligne, valeur réelle dans un bloc de code.
- Ne pas modifier `last_reviewed` sans avoir relu et vérifié toute la page.
- ADR, incidents et chronologie sont figés : amender par un bloc `<Update>` daté, ne pas réécrire le passé.
- Les commandes d'administration du parc vivent dans le dépôt ims-ops : renvoyer vers lui, ne pas recopier son README.

## Vérifier avant de rendre la main

```bash
npx -y mint@4.2.997 validate
npx -y mint@4.2.997 broken-links
```

## Branches

Retouche d'une page : commit sur `main` possible. Chantier sur plusieurs pages ou déplacement de fichier : branche et pull request.
