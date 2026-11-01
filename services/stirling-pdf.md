---
title: "Stirling PDF"
description: "Outils PDF sans conservation des documents : fusion, découpe, conversion, reconnaissance de texte."
icon: "file-pdf"
iconType: "duotone"
last_reviewed: "2026-10-25"
app_version: "latest"
---

## Accès

Public, derrière une connexion Authentik.

<CardGroup cols={2}>
  <Card color="#16A34A" title="Stirling PDF" icon="file-pdf" href="https://pdf.ims-world.fr">
    Outils PDF.
  </Card>
</CardGroup>

```bash
# Depuis la VM 104
cd /data/coolify/services/p6lm9p4zf1caqruekeuocnje/ && docker compose logs -f --tail=100
```

## Fiche

| Propriété | Valeur |
| --- | --- |
| Domaine | `pdf.ims-world.fr` |
| Accès | Public ; forward-auth Authentik (fournisseur `IMS-StrilingPDF-Provider`) |
| Hôte | VM 104 |
| Coolify | UUID `p6lm9p4zf1caqruekeuocnje`, chemin `/data/coolify/services/p6lm9p4zf1caqruekeuocnje/` |
| Image | `stirlingtools/stirling-pdf:latest` |
| Données | Aucun document conservé ; volumes de configuration et de modèles OCR |
| Sauvegarde | Avec la VM 104 |
| Supervision | Sonde Uptime Kuma |
| Mise en service | 13/08/2026 |

## Fonctionnement

Les documents sont traités à la volée et ne sont pas conservés. La connexion propre à Stirling PDF est désactivée (`DOCKER_ENABLE_SECURITY=false`) : l'accès est déjà contrôlé par Authentik, une seconde connexion n'apporterait rien.

| Volume | Chemin | Contenu |
| --- | --- | --- |
| `stirling-training-data` | `/usr/share/tesseract-ocr/5/tessdata` | Modèles de langue de Tesseract 5 |
| `stirling-configs` | `/configs` | Configuration |
| `stirling-custom-files` | `/customFiles/` | Personnalisations de l'interface |
| `stirling-logs` | `/logs/` | Journaux |

## Données et sauvegarde

Sans objet : aucun document conservé.

## Exploitation

Ajouter une langue d'OCR : déposer le fichier `.traineddata` dans le volume `stirling-training-data`. L'image est en `latest` ; son épinglage est suivi dans la [feuille de route](/roadmap).

## Pièges connus

Sans objet.

## Voir aussi

<CardGroup cols={2}>
  <Card color="#FF6B00" title="Protéger une application par forward-auth" icon="list-check" href="/procedures/securiser-application-authentik-forward-auth">Procédure de protection par Authentik.</Card>
</CardGroup>
