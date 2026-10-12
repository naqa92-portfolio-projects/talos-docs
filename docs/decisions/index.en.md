# Decisions

Every page follows the same outline: **context → decision → rejected alternatives → trade-offs and cost →
when to revisit**. Trade-offs are spelled out in black and white: a choice with no cost is not a choice.

| # | Decision | The trade-off accepted |
|---|---|---|
| [001](001-talos-hetzner-terraform.md) | Talos Linux on Hetzner, provisioned by a maintained Terraform module | Depending on the module's defaults, partly unusable as is |
| [002](002-control-plane-unique.md) | A single control plane, `cx33` minimum | No fault tolerance on etcd: losing the node forces a rebuild |
| [003](003-pas-de-haute-disponibilite.md) | No high availability: self-healing rather than continuity | Every component is a single point of failure, by choice |
| [004](004-retour-hetzner.md) | Hetzner → Infomaniak → Scaleway → Hetzner | Three provider switches, absorbed by IaC and GitOps |
| [005](005-argocd-flux-argocd.md) | ArgoCD → Flux → ArgoCD | Flux has the better architecture, ArgoCD dominates the market: I chose the market |
| [006](006-app-of-apps-sync-waves.md) | App-of-apps discovered by folder, ordered by waves | A serialized cold start, slower |
| [007](007-crossplane-api-self-service.md) | A home-grown `App` / `Database` API with Crossplane | One more abstraction layer to maintain and test |
| [008](008-gateway-api-cilium.md) | Gateway API served by Cilium, with no separate ingress controller | Coupling to the CNI, to be paid again when moving to managed Kubernetes |
| [009](009-secrets-infisical-eso.md) | Secrets in Infisical, synced by External Secrets | Depending on a SaaS to start the cluster |
| [010](010-kyverno-audit-garde-fous.md) | Kyverno: audit for best practices, enforcement for data | Best-practice violations are reported, not prevented |
| [011](011-retrait-obi-sdk-otel.md) | Remove eBPF instrumentation in favor of the OpenTelemetry SDK | Changing application code in order to trace it |
| [012](012-pas-de-rendered-manifests.md) | No *Rendered Manifests Pattern*, a render diff in CI instead | No Git history of rendered manifests |

## Evaluated and rejected

A short list of tools evaluated then declined, with the reason in one line:

| Tool | Why not |
|---|---|
| Terragrunt | A single cluster, no multi-environment need |
| Kargo, promotion through environment folders | No persistent environment to promote to: a permanent staging would cost 24/7 for a single person |
| Per-PR preview environments | Full plan costed (~+€15/month, 17 files across 3 repos), dropped: GitOps adds latency where ephemeral should be disposable |
| vCluster | Crossplane is cluster-wide: everything would have to be recreated in each vCluster |
| ARM (`cax31`) | More expensive per GB of RAM (€1.57 vs €1.27) and application images are amd64 only |
| Backup encryption (AGE) | Bucket already private; key management costs more than the risk covered |
| Self-hosted Infisical | The free tier covers the need with nothing to operate |
| kagent, LiteLLM, agentgateway | A shared gateway only pays off from two consumers; there is only one |
| Read-only "everything except Secrets" for Backstage | Impossible in Kubernetes RBAC: a `*` rule includes Secrets, and no webhook filters reads |
