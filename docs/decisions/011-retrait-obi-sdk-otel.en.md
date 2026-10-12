# 011 · Remove eBPF instrumentation in favor of the OpenTelemetry SDK

> **In one sentence:** "no code change" eBPF instrumentation turned out to be intrusive; for an
> application whose code I control, the OpenTelemetry SDK does better, with no privileges.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-09-12 | `talos-argocd`, saveit application |

## Context

OBI (OpenTelemetry eBPF Instrumentation) traces applications without modifying their images: a
privileged DaemonSet places probes inside processes. Adopted in February 2026, it initially covered
all namespaces.

## What I measured

- The eBPF probes disrupted pipe reads in the instrumented processes. I **proved** it with an
  A/B on the PostgreSQL backups: excluding the pod from instrumentation was enough to restore them.
- The symptom looked like bugs in other components (ArgoCD, Talos kernel, CloudNativePG plugin):
  a privileged DaemonSet writing into process memory clouds every diagnosis.
- About 350 Mi of memory per node to trace a single Django application.

## Decision

Remove OBI, instrument the application with the **OpenTelemetry SDK**: ORM, HTTP and
worker spans, and a `trace_id` in every log emitted under a span. Traces go to VictoriaTraces,
metrics to VictoriaMetrics.

## Trade-offs and cost

Code has to be modified to trace. The "zero modification" argument only holds for third-party
applications, and there are none here.

## What I take from it

Before removing OBI, every suspicion was tested with an A/B: one proven (PostgreSQL), two not
reproduced (ArgoCD, Authentik). An 863-line workaround put in place for Authentik turned out to be
unnecessary and was removed, with evidence.
