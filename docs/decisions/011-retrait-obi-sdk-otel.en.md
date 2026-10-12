# 011 · Remove the eBPF instrumentation in favor of the OpenTelemetry SDK

> **In one sentence:** the "no code changes" instrumentation had been corrupting processes for seven
> months; for a single application whose code I control, the SDK does better, with no privilege.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-09-12 | `talos-argocd`, saveit application |

## Context

OBI (OpenTelemetry eBPF Instrumentation) traces applications without modifying their images: a
privileged DaemonSet places probes in processes. Adopted in February 2026, it initially covered
all namespaces.

## What it broke

- Pipe reads of instrumented processes were corrupted. Effect **proven** on the
  PostgreSQL backups: excluding the pod was enough to restore them.
- The same signature weighed on two migrations ([005](005-argocd-flux-argocd.md),
  [004](004-retour-hetzner.md)), after being blamed in turn on barman, the Talos kernel and the
  CloudNativePG plugin.
- About 350 Mi of memory per node to trace a single Django application.

## Decision

Remove OBI, instrument the application with the **OpenTelemetry SDK**: ORM, HTTP and
worker spans, and a `trace_id` in every log emitted under a span. Traces go to VictoriaTraces,
metrics to VictoriaMetrics.

## Trade-offs and cost

The code has to be modified to trace. The "zero modification" argument only holds for third-party
applications, and there are none here.

## What I take from it

Before removing OBI, each suspicion was A/B tested: one proven (PostgreSQL), two not
reproduced (ArgoCD, Authentik). An 863-line workaround put in place for Authentik turned out to be
useless and was removed, with proof.
