---
title: "Incidents"
description: "Index des incidents analysés : date, impact, statut et actions restant à mener."
icon: "triangle-exclamation"
iconType: "duotone"
last_reviewed: "2026-10-25"
---

Chaque incident notable fait l'objet d'une analyse : symptôme, cause, correction et actions. Les règles et le modèle sont décrits dans le [guide de style](/reference/contribuer#incident).

| Date | Incident | Statut | Actions ouvertes |
| --- | --- | --- | --- |
| 23/10/2026 | [Échecs ESTALE sur les sauvegardes PBS et HomeFlix](/history/incidents/2026-10-23-estale-nfs-mergerfs-backups-et-homeflix) | Résolu (retrait de MergerFS) | Aucune : alertes de sauvegarde envoyées sur Ntfy depuis le 24/10/2026 |
| 26/08/2026 | [Faux positifs du WAF CrowdSec AppSec](/history/incidents/2026-08-26-faux-positifs-crowdsec-appsec-outofband) | En observation depuis le 26/08/2026 | Confirmer la fin des faux positifs et clore |
| 24/08/2026 | [Wildcard DNS masqué sur `share.ims-world.fr`](/history/incidents/2026-08-24-dns-wildcard-shadowing-share-ims-world-fr) | Résolu | Nettoyage automatique ou contrôle périodique des TXT `_acme-challenge` |
| 24/08/2026 | [Contournement du SSO de Dozzle](/history/incidents/2026-08-24-bypass-sso-dozzle-traefik-file-provider) | Résolu | Revoir les middlewares de chaque routeur de `vpn-only.yaml` |
| 19/08/2026 | [Descripteurs NFS périmés sur Jellyfin](/history/incidents/2026-08-19-stale-nfs-filehandle-jellyfin-mergerfs) | Résolu (définitivement le 23/10/2026) | Aucune |
| 19/08/2026 | [Perte de `/dev/dri` après redémarrage](/history/incidents/2026-08-19-perte-gpu-passthrough-dev-dri) | Résolu | Aucune |
| 18/08/2026 | [Double blocage 403 du middleware `vpn-only`](/history/incidents/2026-08-18-blocage-traefik-vpn-only-docker-proxy) | Résolu | Aucune |
