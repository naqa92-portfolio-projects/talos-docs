# talos-fluxcd : l'épisode Flux (archivé)

> **Le dépôt GitOps du cluster de mai à septembre 2026, sous Flux. Archivé au retour vers ArgoCD.**

| | |
|---|---|
| Période | 2026-05-30 → 2026-09-13 |
| Outils | Flux (HelmRelease, Kustomization), Crossplane |
| Visibilité | Privé, archivé |

## Pourquoi il a existé

Pour comparer Flux et ArgoCD sur un vrai cluster, au moment où la
[Web UI de Flux Operator](https://fluxoperator.dev/web-ui/) levait le principal reproche fait à Flux,
l'absence d'interface. Une panne récurrente côté ArgoCD a accéléré le passage. Toute l'histoire est dans
la [décision 005](../decisions/005-argocd-flux-argocd.md).

## Ce que Flux faisait mieux, et qu'il a fallu reconstruire sous ArgoCD

| Sous Flux | Équivalent mis en place sous ArgoCD |
|---|---|
| `dependsOn` + `wait` entre composants | Vagues ordonnées + contrôle de santé en Lua ([006](../decisions/006-app-of-apps-sync-waves.md)) |
| Contrôles de santé en CEL | Scripts de santé Lua pour les ressources Crossplane |
| Application en une seule passe | Ordre explicite, ce qui a révélé une dépendance cachée entre politiques réseau ([incident](../incidents.md#argocd-coupe-api)) |

## Ce que j'en retiens

Flux a la meilleure architecture, mais il est plus exigeant à prendre en main. ArgoCD reste le standard
du marché, d'où le retour. Et la panne a suivi le cluster sous Flux, ce qui a permis d'en trouver la
vraie cause, étrangère aux deux outils.
