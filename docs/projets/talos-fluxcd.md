# talos-fluxcd : l'épisode Flux (archivé)

> **Le dépôt GitOps du cluster de mai à septembre 2026, sous Flux. Archivé au retour vers ArgoCD.**

| | |
|---|---|
| Période | 2026-05-30 → 2026-09-13 |
| Outils | Flux (HelmRelease, Kustomization), Crossplane |
| Visibilité | Privé, archivé |

## Pourquoi il a existé

Pour fuir un bug chronique attribué à ArgoCD. Un PoC sur le même cluster donnait 0 erreur sous Flux,
contre 16 sous ArgoCD. Le bug venait en réalité d'un autre composant : toute l'histoire est dans la
[décision 005](../decisions/005-argocd-flux-argocd.md).

## Ce que Flux faisait mieux, et qu'il a fallu reconstruire sous ArgoCD

| Sous Flux | Équivalent mis en place sous ArgoCD |
|---|---|
| `dependsOn` + `wait` entre composants | Vagues ordonnées + contrôle de santé en Lua ([006](../decisions/006-app-of-apps-sync-waves.md)) |
| Contrôles de santé en CEL | Scripts de santé Lua pour les ressources Crossplane |
| Application en une seule passe | Ordre explicite, ce qui a révélé une dépendance cachée entre politiques réseau ([incident](../incidents.md#argocd-coupe-api)) |

## Ce que j'en retiens

Changer d'outil n'a pas supprimé le problème : la même signature est réapparue sous Flux, sur Velero.
C'est ce qui a remis le diagnostic en cause.
