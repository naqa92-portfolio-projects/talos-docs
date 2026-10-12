# talos-argocd : le dépôt GitOps

> **L'état désiré de tout le cluster : 35 composants d'infrastructure, les applications métier et l'API
> self-service Crossplane.**

| | |
|---|---|
| Rôle | Source de vérité de ce qui tourne dans le cluster |
| Outils | ArgoCD, Helm, Crossplane v2 (compositions Python), Kyverno, Cilium, Chainsaw |
| Visibilité | Privé |

## Ce qu'il contient

- `infrastructure/` : les composants, rangés par catégorie (`controllers`, `configs`, `observability`,
  `platform`). Chacun suit la même convention : `application.yaml` + `values.yaml` + `resources/`.
- `apps/prod/` : les applications métier, chacune décrite par un claim `App` de 15 lignes.
- `crossplane/` : le code Python des compositions, les exemples et les tests Chainsaw.
- `clusters/dev/` : le sous-ensemble rejoué sur le cluster Kind local de Backstage.

## La CI sur chaque pull request

| Contrôle | Ce qu'il garantit |
|---|---|
| Validation des Applications | Chaque source Git déclare explicitement son mode de lecture, et aucun fichier Kustomize ne réapparaît (garde-fou né de l'[incident](../incidents.md#prune-saveit)) |
| Cohérence des CRD | Les versions de test correspondent à celles de la production |
| kube-linter, Trivy | Bonnes pratiques et configuration |
| `argocd-diff-preview` | Le diff des manifestes rendus est publié en commentaire de la PR, suppressions comprises |
| Crossplane | Lint, code généré à jour, rendu hors cluster, puis 6 tests de bout en bout sur Kind |

## En chiffres

35 Applications d'infrastructure, 19 secrets synchronisés, 13 politiques réseau Cilium, 11 politiques
Kyverno, 2 API maison, environ 950 commits. Renovate propose les mises à jour avec un délai de
maturité : 3 jours pour un patch, 7 pour une version mineure ou majeure.

## Décisions liées

[005](../decisions/005-argocd-flux-argocd.md) · [006](../decisions/006-app-of-apps-sync-waves.md) ·
[007](../decisions/007-crossplane-api-self-service.md) · [008](../decisions/008-gateway-api-cilium.md) ·
[009](../decisions/009-secrets-infisical-eso.md) · [010](../decisions/010-kyverno-audit-garde-fous.md) ·
[012](../decisions/012-pas-de-rendered-manifests.md)
