# talos-fluxcd: the Flux episode (archived)

> **The cluster's GitOps repo from May to September 2026, under Flux. Archived on the return to ArgoCD.**

| | |
|---|---|
| Period | 2026-05-30 → 2026-09-13 |
| Tools | Flux (HelmRelease, Kustomization), Crossplane |
| Visibility | Private, archived |

## Why it existed

To compare Flux and ArgoCD on a real cluster, just as the
[Flux Operator Web UI](https://fluxoperator.dev/web-ui/) removed the main criticism of Flux,
the lack of a UI. A recurring outage on the ArgoCD side sped up the move. The whole story is in
[decision 005](../decisions/005-argocd-flux-argocd.md).

## What Flux did better, and had to be rebuilt under ArgoCD

| Under Flux | Equivalent put in place under ArgoCD |
|---|---|
| `dependsOn` + `wait` between components | Ordered waves + Lua health check ([006](../decisions/006-app-of-apps-sync-waves.md)) |
| CEL health checks | Lua health scripts for Crossplane resources |
| Application in a single pass | Explicit order, which revealed a hidden dependency between network policies ([incident](../incidents.md#argocd-coupe-api)) |

## What I take from it

Flux has the better architecture, but it is more demanding to pick up. ArgoCD remains the market
standard, hence the return. And the outage followed the cluster under Flux, which made it possible to find its
real cause, unrelated to either tool.
