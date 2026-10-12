# 005 · ArgoCD → Flux → ArgoCD

> **In one sentence:** I left ArgoCD because of a bug I had blamed on ArgoCD; four months
> later, the same signature appeared elsewhere, proven to come from another component, and I
> went back to ArgoCD after an A/B proof.

| Status | Date | Repo |
|---|---|---|
| Adopted: ArgoCD | Flux on 2026-05-30, back to ArgoCD on 2026-09-13 | `talos-argocd` (formerly `talos-fluxcd`) |

## Context

In spring 2026, the ArgoCD component that reads Git (`repo-server`) failed chronically at
cluster startup (`git fetch exit 128`, output filled with null bytes). Four stacked
workarounds were not enough. A Flux PoC on the same cluster gave **0 errors**, against 16 on the
ArgoCD side. I migrated.

## What changed the diagnosis

1. The same signature (null bytes in a subprocess's output) was **proven** on the
   PostgreSQL backups to be an effect of the OBI eBPF instrumentation. Excluding the OBI pod
   was enough to make the backup pass. And OBI was also instrumenting ArgoCD's namespace for
   the whole life of the bug.
2. Under Flux, Velero showed the same signature: changing GitOps tool had not removed the
   problem.

## The proof before going back

Dedicated bench, uprobes verified before each measurement window: **334 instrumented `git fetch`,
0 errors**. Then, in production, 0 occurrences over more than 40,000 log lines, across five full
startups.

**Scope of the result, without overstating it:** the bench had neither the same ArgoCD version nor the same
kernel as the incident. The OBI cause is very likely, but neither proven nor ruled out for the
historical incident.

## Decision

Back to ArgoCD. This is not a *revert*: the repo was rewritten, this time with an explicit
startup ordering ([decision 006](006-app-of-apps-sync-waves.md)). Its absence also explained part of
the initial load.

Why go back rather than stay on Flux: **this bug was the only reason for leaving**. The argument
"Flux launches no subprocess" had never been a need, only a property noticed
after the fact. With the reason gone, the decision went too. The ArgoCD building blocks already built could be
reused: SSO, render diff in CI (`argocd-diff-preview`), Backstage plugin.

## Trade-offs and cost

- ArgoCD has no `dependsOn` between Applications, unlike Flux. It needed waves and a
  health check written in Lua.
- Four months on a tool chosen for the wrong reason, and two full migrations.

## What I take from it

- **An eBPF component is a default suspect** in any "impossible" diagnosis on a pod it
  instruments. It is limited per workload, never to a whole namespace.
- **A suspect is not a culprit.** An unmeasured attribution costs as much as the bug.
- I planned a graduated fallback in case the cause came back, and I forbade myself from putting the
  old workarounds back.
