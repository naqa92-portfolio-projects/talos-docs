# 010 · Kyverno: audit for best practices, blocking for data

> **In one sentence:** best-practice rules flag without blocking, because there is only one
> user; the only blocking rule protects what is unrecoverable, the data.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-04 | `talos-argocd` |

## Decision

| Policy | Mode | Why |
|---|---|---|
| Unprivileged containers, non-root, no host namespaces, with requests / limits, allowed registries | **Audit** | Visibility is enough for a single user; blocking makes sense when several teams deploy |
| Validation of `App` / `Database` claims, uniqueness of hostnames | **Audit** | Same reason |
| Deletion of a Crossplane resource without the `allow-delete` annotation | **Blocking** | A database deleted by mistake cannot be rebuilt from Git |

The policies are written in **CEL** (Kubernetes' native expression language), the old `ClusterPolicy`
format being phased out.

## What the blocking rule has already prevented

During the [incident of 2026-09-15](../incidents.md#prune-saveit), this policy refused
the deletion of the Crossplane resources and stopped the cascade.

## Trade-offs and cost

Bad practices are visible in reports, not prevented. A non-compliant deployment
goes through.

## When to revisit

As soon as a second person or team deploys on the cluster: switch to blocking, rule by rule.
