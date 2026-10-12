# 008 · Gateway API served by Cilium, with no separate ingress controller

> **In one sentence:** Cilium already does networking and policies; giving it HTTP ingress too
> removes one component to operate, at the price of a deliberate coupling.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-09 | `talos-infra`, `talos-argocd` |

## Context

I need an HTTPS entry point for public services. The legacy Ingress is replaced by the Gateway API,
the current Kubernetes standard. During the Infomaniak stint, I used Envoy Gateway.

## Decision

- **Cilium** serves the Gateway API, replacing kube-proxy, with the Hetzner load balancer driven
  by annotations.
- **A generic HTTPS listener** on the *wildcard* certificate: a new application is
  reachable without changing the Gateway.
- ArgoCD is exposed like the other services (TLS termination), no longer in *passthrough*.

## Alternatives rejected

| Alternative | Why not |
|---|---|
| Envoy Gateway / ingress-nginx | One more component to operate and update |
| TLS passthrough for ArgoCD | Under a *wildcard* certificate, HTTP/2 reuses another host's connection (*connection coalescing*, RFC 7540): intermittent 404s. Cilium 1.20 allows disabling the gRPC-web translation that forced passthrough |
| One certificate per host | Let's Encrypt quota (5 per week), and a file to edit for every new host |

## Trade-offs and cost

**Coupling to the CNI.** Moving to a managed Kubernetes, where the CNI is not mine, will require
reintroducing an ingress controller. That is precisely what blocked on Scaleway
([decision 004](004-retour-hetzner.md)).
