# talos-argocd: the GitOps repo

> **The desired state of the whole cluster: 35 infrastructure components, the business applications and the
> Crossplane self-service API.**

| | |
|---|---|
| Role | Source of truth for what runs in the cluster |
| Tools | ArgoCD, Helm, Crossplane v2 (Python compositions), Kyverno, Cilium, Chainsaw |
| Visibility | Private |

## What it contains

- `infrastructure/`: the components, sorted by category (`controllers`, `configs`, `observability`,
  `platform`). Each follows the same convention: `application.yaml` + `values.yaml` + `resources/`.
- `apps/prod/`: the business applications, each described by a 15-line `App` claim.
- `crossplane/`: the compositions' Python code, the examples and the Chainsaw tests.
- `clusters/dev/`: the subset replayed on Backstage's local Kind cluster.

## CI on every pull request

| Check | What it guarantees |
|---|---|
| Application validation | Each Git source explicitly declares its read mode, and no Kustomize file comes back (a guardrail born from the [incident](../incidents.md#prune-saveit)) |
| CRD consistency | Test versions match the production ones |
| kube-linter, Trivy | Best practices and configuration |
| `argocd-diff-preview` | The diff of rendered manifests is posted as a PR comment, deletions included |
| Crossplane | Lint, generated code up to date, render outside the cluster, then 6 end-to-end tests on Kind |

## In numbers

35 infrastructure Applications, 19 synced secrets, 13 Cilium network policies, 11 Kyverno
policies, 2 homemade APIs, about 950 commits. Renovate proposes updates with a maturity
delay: 3 days for a patch, 7 for a minor or major version.

## Related decisions

[005](../decisions/005-argocd-flux-argocd.md) · [006](../decisions/006-app-of-apps-sync-waves.md) ·
[007](../decisions/007-crossplane-api-self-service.md) · [008](../decisions/008-gateway-api-cilium.md) ·
[009](../decisions/009-secrets-infisical-eso.md) · [010](../decisions/010-kyverno-audit-garde-fous.md) ·
[012](../decisions/012-pas-de-rendered-manifests.md)
