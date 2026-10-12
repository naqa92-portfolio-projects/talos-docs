# 003 · No high availability: self-healing rather than continuity

> **In one sentence:** every component is a single point of failure, by a written and costed
> economic choice; the platform can repair and rebuild itself, but not stay up during an outage.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-09 | `talos-infra`, `talos-argocd` |

## Context

High availability is not a setting, it is a sum of costs. The service has one user,
and downtime has no business cost.

## Decision

No continuity mechanism: one replica per application, one PostgreSQL instance, no
PodDisruptionBudget, no autoscaling, a single zone.

**What is covered instead:**

- ArgoCD reconciles every 1 to 2 minutes: any change made outside Git is reverted.
- ExternalDNS rewires DNS to the new IP by itself after a full rebuild.
- External Secrets + Reloader: rotating a secret restarts the affected pods with no intervention.
- Gatus and Slack alerts to detect, B2 backups to restore.

## What each lever would have cost

| Lever | Cost |
|---|---|
| Several replicas | ~+0.5 Gi of reserved RAM per replica and per application |
| PostgreSQL with 2 instances | One more pod, doubled storage |
| PodDisruptionBudget | Free, but useless without replicas; it would even block node maintenance |
| Autoscaling (HPA) | Assumes capacity that is paid for and permanently unused |
| HA control plane | ×3 on the most expensive line item |
| Multi-zone | Duplicated workers, cross-zone traffic, volumes that cannot be attached across zones |

## Trade-offs and cost

A node loss or an update causes an interruption, limited to the time pods take to restart
elsewhere.

## When to revisit

As soon as a service has third-party users or an availability commitment is made.
