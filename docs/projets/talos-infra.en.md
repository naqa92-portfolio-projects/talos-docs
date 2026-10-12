# talos-infra: the infrastructure

> **One `task apply` creates the servers, installs the OS and Kubernetes, then hands over to ArgoCD.**

| | |
|---|---|
| Role | Provision the cluster and bootstrap it into GitOps |
| Tools | Terraform, `hcloud-k8s` module, Talos Linux, Cilium, Taskfile, Devbox |
| Visibility | Private |

## What it contains

- **Terraform**: Hetzner servers (1 control plane + 2 workers), network, firewall, load balancer,
  installation of ArgoCD and the root Application. The Terraform state is stored outside Hetzner
  (Backblaze B2).
- **The full lifecycle as tasks**: `apply` (creation + convergence), `destroy` (clean
  teardown, DNS neutralized), PostgreSQL restore, TLS quota check, tests of the network
  and Kyverno policies.
- **The operations runbooks**: disaster recovery, backup and restore, adding a
  component or a route, network policies, sizing.
- **The PRDs**: nearly 50 closed design documents, with their decisions, milestones and
  measured results.

## Key points

- **The cluster is disposable.** It is destroyed and recreated regularly to control cost. Everything that
  must survive (Terraform state, etcd snapshots, PostgreSQL logs, Velero backups) lives on
  Backblaze B2.
- **No secret in the Terraform state** thanks to *write-only* attributes.
- **Terraform bootstraps ArgoCD then lets go**: ArgoCD then manages itself from Git, with no ownership
  conflict with Terraform.
- **A register of workarounds**, each with its upstream cause and exit condition, re-read at every
  update of the component concerned.

## Related decisions

[001](../decisions/001-talos-hetzner-terraform.md) · [002](../decisions/002-control-plane-unique.md) ·
[003](../decisions/003-pas-de-haute-disponibilite.md) · [004](../decisions/004-retour-hetzner.md)
