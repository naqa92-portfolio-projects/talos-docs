# 012 · No *Rendered Manifests Pattern*, a render diff in CI instead

> **In one sentence:** committing rendered manifests cost more than it brought at this
> scale; a render diff posted as a comment on each PR gives the same review without the artifact.

| Status | Date | Repo |
|---|---|---|
| Rejected twice, definitively on 2026-09-15 | | `talos-argocd` |

## Context

The *Rendered Manifests Pattern* consists of committing the final rendered YAML (Helm, Kustomize) so that Git
shows exactly what is running. The pattern is highly recommended in the GitOps literature.

## Why not

1. **ArgoCD's native implementation (Source Hydrator) cannot express this repo**: 23 Applications
   out of 37 combine a remote chart, values and local manifests. Both gaps are open
   upstream requests.
2. **Rendering in CI does not faithfully reproduce the cluster**: 17 charts out of 22 read the real capabilities
   of the API (`.Capabilities`), which a CI does not know, and 7 emit Helm hooks that the pattern loses.
3. **The attempt broke production.** During the migration, a component was rendered with its default
   values while still displayed `Healthy` (see [incidents](../incidents.md#fenetre-migration)).

## Decision

`argocd-diff-preview` in CI: each PR receives as a comment the diff of the manifests rendered on both
sides. It is the same information for the review, with no artifact to maintain.

## Trade-offs and cost

No Git history of the rendered manifests: you cannot `git blame` a final object.

## When to revisit

If one day the history of rendered manifests is needed to find a regression, if a cold
start fails for lack of access to the chart repos, or if the upstream requests are delivered.
