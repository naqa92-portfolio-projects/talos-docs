# talos-fluxcd: the Flux episode (archived)

> **The cluster's GitOps repo from May to September 2026, under Flux. Archived on the return to ArgoCD.**

| | |
|---|---|
| Period | 2026-05-30 → 2026-09-13 |
| Tools | Flux (HelmRelease, Kustomization), Crossplane |
| Visibility | Private, archived |

## Why it existed

To escape a chronic bug blamed on ArgoCD. A PoC on the same cluster gave 0 errors under Flux,
against 16 under ArgoCD. The bug actually came from another component: the whole story is in
[decision 005](../decisions/005-argocd-flux-argocd.md).

## What Flux did better, and had to be rebuilt under ArgoCD

| Under Flux | Equivalent put in place under ArgoCD |
|---|---|
| `dependsOn` + `wait` between components | Ordered waves + health check in Lua ([006](../decisions/006-app-of-apps-sync-waves.md)) |
| Health checks in CEL | Lua health scripts for Crossplane resources |
| Applied in a single pass | Explicit order, which revealed a hidden dependency between network policies ([incident](../incidents.md#argocd-coupe-api)) |

## What I take from it

Changing tool did not remove the problem: the same signature reappeared under Flux, on Velero.
That is what put the diagnosis back in question.
