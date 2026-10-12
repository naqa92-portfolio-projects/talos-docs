# Decisions

Each page follows the same outline: **context → decision → alternatives rejected → trade-offs and cost →
when to revisit**. Trade-offs are written in black and white: a choice with no cost is not a choice.

| # | Decision | The trade-off accepted |
|---|---|---|
| [001](001-talos-hetzner-terraform.md) | Talos Linux on Hetzner, provisioned by a maintained Terraform module | Depending on the module's defaults, partly unusable as-is |
| [002](002-control-plane-unique.md) | A single control plane, `cx33` minimum | No fault tolerance on etcd: losing the node forces a rebuild |
| [003](003-pas-de-haute-disponibilite.md) | No high availability: self-healing rather than continuity | Every component is a single point of failure, by choice |
| [004](004-retour-hetzner.md) | Hetzner → Infomaniak → Scaleway → Hetzner | Three migrations in two months to find the right price / reliability trade-off |
| [005](005-argocd-flux-argocd.md) | ArgoCD → Flux → ArgoCD | Four months on Flux because of a misattributed bug |
| [006](006-app-of-apps-sync-waves.md) | App-of-apps discovered by folder, ordered by waves | A serialized cold start, slower |
| [007](007-crossplane-api-self-service.md) | A homemade `App` / `Database` API with Crossplane | One more abstraction layer to maintain and test |
| [008](008-gateway-api-cilium.md) | Gateway API served by Cilium, with no separate ingress controller | Coupling to the CNI, to be paid again when moving to a managed Kubernetes |
| [009](009-secrets-infisical-eso.md) | Secrets in Infisical, synced by External Secrets | Depending on a SaaS to start the cluster |
| [010](010-kyverno-audit-garde-fous.md) | Kyverno: audit for best practices, blocking for data | Best-practice violations are flagged, not prevented |
| [011](011-retrait-obi-sdk-otel.md) | Remove the eBPF instrumentation in favor of the OpenTelemetry SDK | Changing the application code to trace it |
| [012](012-pas-de-rendered-manifests.md) | No *Rendered Manifests Pattern*, a render diff in CI instead | No Git history of the rendered manifests |

## What was rejected without being adopted

A short list of tools evaluated then refused, with the reason in one line:

| Tool | Why not |
|---|---|
| Terragrunt | A single cluster, no multi-environment need |
| Kargo, promotion by environment folders | No persistent environment to promote: a permanent staging would cost 24/7 for one person |
| Per-PR preview environments | Full plan costed (~+€15/month, 17 files across 3 repos), rejected: GitOps adds latency where the ephemeral must be disposable |
| vCluster | Crossplane is cluster-wide: everything would have to be recreated in each vCluster |
| ARM (`cax31`) | More expensive per GB of RAM (€1.57 vs €1.27) and application images are amd64 only |
| Backup encryption (AGE) | Bucket is already private; key management costs more than the risk it covers |
| Self-hosted Infisical | The free tier covers the need with nothing to operate |
| kagent, LiteLLM, agentgateway | A shared gateway only pays off from two consumers; there is only one |
| Read-only "everything except Secrets" for Backstage | Impossible in Kubernetes RBAC: a `*` rule includes Secrets, and no webhook filters reads |
