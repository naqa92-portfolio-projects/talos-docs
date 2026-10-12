# 007 · A homemade `App` / `Database` API with Crossplane

> **In one sentence:** a developer describes their application in 15 lines; the platform generates the
> accompanying Kubernetes resources and applies best practices on their behalf.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-03 | `talos-argocd` |

## Context

Deploying an application properly on this platform takes about ten objects: Deployment,
Service, HTTPS route, HTTP redirect, network policy, registry pull secrets, monitoring.
Writing them by hand for each application means copying mistakes.

## Decision

Two homemade types exposed by Crossplane v2:

```yaml
apiVersion: taloslab.cc/v1alpha1
kind: App
metadata:
  name: saveit
spec:
  image: ghcr.io/…/saveit:0.2.11
  hostname: saveit.taloslab.cc
  size: medium          # small | medium | large → requests / limits
  migrate: true         # database migrations before startup (initContainer)
  workerCommand: "…"    # second Deployment for background jobs
  metrics: true         # ServiceMonitor + dashboard
```

| The developer writes | The platform generates |
|---|---|
| `kind: App` | ServiceAccount, Deployment (+ worker), Service, 2 HTTPRoutes (HTTPS + redirect), CiliumNetworkPolicy, ExternalSecrets, ConfigMap, ServiceMonitor |
| `kind: Database` | CloudNativePG PostgreSQL cluster, PodMonitor, scheduled backup and continuous archiving to B2 |

- The composition logic is written **in Python** (`function-pythonic`): readable, testable, rendered
  outside the cluster by `crossplane render`.
- **End-to-end Chainsaw tests** on an ephemeral Kind cluster in CI, plus a check that the
  generated code is up to date.
- Each resource's status is surfaced in Backstage with readable conditions.

## Alternatives rejected

| Alternative | Why not |
|---|---|
| A shared Helm chart | No status or continuous reconciliation; each team can override anything |
| Pure YAML compositions (patch & transform) | Unreadable as soon as there are conditions |
| Crossplane Operations (alpha), Usage / deletion protection | Excessive for two applications; a Kyverno policy does the same job more simply |
| Crossplane multi-team RBAC | Designed for several teams, pointless for one person |

## Trade-offs and cost

One more layer to maintain. Defaults live in the API definition (XRD) and only
apply at admission, so test examples must write them explicitly. Changing a field's type
requires a migration procedure.
