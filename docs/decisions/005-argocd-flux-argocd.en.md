# 005 · ArgoCD → Flux → ArgoCD

> **In one sentence:** I tested Flux in real conditions for four months. Its architecture is
> better, but ArgoCD is the market standard, and the outage that had pushed me to leave
> turned out to be unrelated to ArgoCD: I switched back.

| Status | Date | Repo |
|---|---|---|
| Adopted: ArgoCD | Flux on 2026-05-30, back to ArgoCD on 2026-09-13 | `talos-argocd` (formerly `talos-fluxcd`) |

## Why I tested Flux

Two reasons, arriving at the same time:

1. **The release of the [Flux Operator Web UI](https://fluxoperator.dev/web-ui/).** The lack of a
   UI was the main criticism of Flux against ArgoCD. A built-in UI, with OIDC SSO
   and a delivery pipeline graph, removed that argument. It was the time to compare the two
   tools on a real cluster.
2. **A recurring outage on the ArgoCD side.** The component that reads Git (`repo-server`) failed at
   cluster startup (`git fetch exit 128`). A Flux PoC on the same cluster gave 0 errors,
   against 16 on the ArgoCD side.

## What four months of Flux taught me

**Flux has the better architecture.** Specialized controllers (sources, Kustomize, Helm) rather than
a central server. Native dependencies between components (`dependsOn` + `wait`), CEL health
checks, and the application of a component in a single pass.

**But it is harder to pick up.** There are more objects to understand, state is spread
across several controllers, and debugging goes through the CLI more than through an overview.

## Why I went back to ArgoCD

1. **The market.** ArgoCD is the most requested GitOps engine in platform teams. Mastering it
   in depth (ordering, Application health, render diff in CI) is worth more for my
   engagements.
2. **The outage was not ArgoCD's.** The same signature (null bytes in a subprocess's output)
   appeared on other components. I isolated it with an A/B: it came from the OBI eBPF
   instrumentation ([decision 011](011-retrait-obi-sdk-otel.md)). Before switching back, I ran another proof on a
   dedicated bench: **334 instrumented `git fetch`, 0 errors**. Then, in production, 0 occurrences over more than
   40,000 log lines, across five full startups.

## Decision

Back to ArgoCD. This is not a *revert*: the repo was rewritten, and it takes over what Flux did
better. Startup ordering becomes explicit ([decision 006](006-app-of-apps-sync-waves.md)),
and health checks written in Lua replace Flux's CEL ones.

## Trade-offs and cost

- ArgoCD has no `dependsOn` between Applications: it took waves and a Lua health check.
- Two complete GitOps migrations. Both were absorbed without touching the applications.

## What I take from it

- Comparing two tools on a real cluster teaches more than a feature matrix.
- A component that instruments other processes is a default suspect in any "impossible"
  diagnosis. Every suspicion is verified with an A/B before acting.
