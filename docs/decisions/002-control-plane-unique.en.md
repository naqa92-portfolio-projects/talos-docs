# 002 · A single control plane, `cx33` minimum

> **In one sentence:** one control plane instead of three divides the cost of the most expensive line item, and the
> daily etcd backup covers the real recovery need.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-09 | `talos-infra` |

## Context

A highly available control plane needs three nodes (etcd needs a majority). The cluster
is single-user, and it is regularly destroyed and recreated.

## Decision

- **A single control plane**, dedicated to Kubernetes: it refuses applications (`NoSchedule` *taint*).
  Two workers carry the load.
- Floor size **`cx33`** (4 vCPU / 8 GB), set by measurement.

## The measurement that set the size

On a `cx23` (2 vCPU), the initial GitOps sync saturated the control plane at **132% CPU**.
No memory alert fired: no `OOMKilled`, no `MemoryPressure`. The real symptom was
a cascade of restarts (controller-manager, scheduler, GitOps controllers) caused by health
probe timeouts.

After moving to `cx33`: 28% CPU, no more restarts. The resize was done in
place in 1 min 9 s, with no etcd loss. Cost of the upgrade: +€3.60/month.

**Lesson:** on a control plane, the bottleneck is CPU at startup, not memory in steady state.
Size for the peak of the cascade, not for idle.

## Alternatives rejected

| Alternative | Why not |
|---|---|
| 3 control planes | ×3 on the most expensive line item, for a service with no third-party users |
| Control planes that also carry applications | That was the old topology: applications competed with etcd and the API |

## Trade-offs and cost

**etcd has a single member: no fault tolerance.** Losing the node forces a rebuild of the
cluster, not an in-place repair. The daily etcd snapshot to Backblaze B2 (outside Hetzner) and the
rebuild through Terraform + GitOps make this risk acceptable.

The resize is also **irreversible**: Hetzner can grow the disk but cannot shrink it.

## When to revisit

As soon as the cluster carries a service with third-party users, or an availability commitment.
