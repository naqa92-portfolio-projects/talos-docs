# 004 · Hetzner → Infomaniak → Scaleway → Hetzner

> **In one sentence:** I left Hetzner for a managed Kubernetes to cut cost and eliminate
> a bug, then came back because the free managed offers gave out on the reliability of their
> control plane.

| Status | Date | Repo |
|---|---|---|
| Adopted: Hetzner, separate control plane and workers | 2026-02 → 2026-09-12 | `talos-infra` |

## Timeline

| When | Where | Why we leave / arrive |
|---|---|---|
| 2026-02 | **Hetzner** + Talos, 3 control planes that also carry applications | Starting point |
| 2026-07 | **Infomaniak KaaS** (managed) | The managed control plane is free, so real workers for the same budget; and a corrupted-pipe bug seemed to come from Talos (~€33-38/month at Hetzner at the time) |
| 2026-09 | **Scaleway Kapsule** (managed), attempt | The Infomaniak API timed out very often, while the cluster is recreated frequently |
| 2026-09-12 | **Back to Hetzner**, 1 control plane + 2 workers | Scaleway disqualified in one day: **0.73 s per etcd request** on the shared control plane, with only 56 pods. The `cx33` type was available again at Hetzner |

## Decision

Back to "homemade" on Hetzner, this time with a dedicated control plane and two workers. I keep
control of the OS and the network (Cilium with Gateway API, which a managed offer did not let me configure
fully).

## What these migrations taught me

- **On a managed control plane, measure etcd latency before migrating GitOps.** The advertised
  limits (RAM, node count) were never reached: it was the undocumented latency that
  gave out.
- **Server type availability changes without notice.** At Hetzner, `cx43`/`cx53` disappeared
  and `cx33` went through three states in one hour. Availability must be checked at each `apply`.
- **The "Talos bug" was not one.** It came from the eBPF instrumentation (see
  [decision 011](011-retrait-obi-sdk-otel.md)). An unmeasured attribution weighed on a migration.

## Alternatives rejected

| Alternative | Why not |
|---|---|
| Stay on Infomaniak | Recurring API timeouts, incompatible with a cluster that is often recreated |
| Scaleway Kapsule | Etcd latency on the shared control plane; Cilium's Gateway API cannot be enabled (operator configuration out of reach) |
| ARM at Hetzner (`cax31`) | More expensive per GB of RAM, and applications published for amd64 only |

## Trade-offs and cost

Three migrations in two months. Each one was run like a project, with a PRD (design document), milestones and an
end measurement, and GitOps made it possible to rebuild the platform each time. Current cost:
**€41.36/month**.
