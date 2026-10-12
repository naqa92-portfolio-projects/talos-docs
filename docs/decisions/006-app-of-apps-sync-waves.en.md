# 006 · App-of-apps discovered by folder, ordered by waves

> **In one sentence:** adding a component means dropping a file in a folder, and startup order is
> guaranteed by waves that wait for the previous wave to be healthy.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-09 | `talos-argocd` |

## Context

A platform has strict dependencies: an operator's types (CRDs) must exist before the
resources that use them, and a certificate needs the Cloudflare secret, itself provided by
External Secrets. On the first pass on ArgoCD, 26 syncs launched at the same time had
brought a cluster to its knees.

## Decision

- **A root Application** recursively discovers every `application.yaml`. No central registry.
- **Waves** (`sync-wave`): wave N+1 starts only once wave N is synced **and**
  healthy. ArgoCD disabled health for child Applications by default: I restore it with a
  Lua script, without which the waves wait for nothing.
- The root **manages itself**: it is one of its own sources.
- **A single convention**: `<category>/<component>/application.yaml` + `values.yaml` + `resources/`.
  No exceptions, no umbrella folder.

## Alternatives rejected

| Alternative | Why not |
|---|---|
| ApplicationSets | More concurrency on Git operations (that was a factor in the bug in [decision 005](005-argocd-flux-argocd.md)), and more indirect to read |
| Progressive Syncs (RollingSync) | Feature still behind a flag, and it **disables auto-sync** on the generated Applications: we would lose self-healing |
| Kustomize | Removed: every file was declared two or three times (the file, the folder index, the parent index) |

## Trade-offs and cost

- Cold start is serialized, so slower than a parallel start, but it converges by itself
  (33/33 Applications at the last full startup).
- Automatic discovery has a downside: a path error can cause resources to be deleted
  (*prune*). It happened ([incident](../incidents.md#prune-saveit)). Since then, a CI check verifies
  every Application, and a Kyverno policy blocks the deletion of resources that hold
  data.
