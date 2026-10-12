# 001 · Talos Linux on Hetzner, provisioned by a maintained Terraform module

> **In one sentence:** I use a maintained community Terraform module rather than writing or
> forking my own provisioning, and I only override what is broken.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-02, renewed in 2026-09 | `talos-infra` |

## Context

I need a reproducible Kubernetes cluster that I can destroy in the evening and recreate the next day to
control cost. Kubernetes "by hand" on VMs means managing the OS, updates, SSH and
hardening.

## Decision

- **Talos Linux** as the OS: immutable, no SSH or shell, fully driven by API. There is nothing to
  patch by hand, and the attack surface is minimal.
- **Terraform module `hcloud-k8s/kubernetes/hcloud`** (pinned version): it installs Talos, Cilium,
  the Hetzner integrations (load balancer, encrypted volumes) and etcd backup.
- Principles set from the first PRD (design document): **no Terragrunt, no fork, no SOPS**.

## Alternatives rejected

| Alternative | Why not |
|---|---|
| Write my own Terraform resources | Reinventing what a maintained module already does, and maintaining it alone |
| Fork the module | Losing upstream updates; every bump becomes a manual merge |
| Managed Kubernetes | Tried twice (Infomaniak, Scaleway), see [decision 004](004-retour-hetzner.md) |

## Trade-offs and cost

Not all of the module's defaults were usable. The image builder's server type
(`cpx11`) could no longer be ordered in Europe, and the default Talos version was newer than the
available `talosctl` client. Both are overridden explicitly, with the reason in a comment.

## When to revisit

If the module is no longer maintained, or if a multi-cluster need appears (Terragrunt would become
relevant again).
