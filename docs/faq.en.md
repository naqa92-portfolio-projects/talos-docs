# Interview questions

The questions I get asked, or should be asked, with the short answer. The detail is in
each decision page.

## Overview

??? question "In one sentence, what is Taloslab?"
    A complete Kubernetes platform that I operate solo on Hetzner for €41/month. A developer
    fills in a Backstage form, and their application is online with its database, DNS, TLS, secrets and
    monitoring, entirely through GitOps.

??? question "Why build all this for a single user?"
    To practice the platform engineer job on a real system, not on a tutorial. Every
    choice has a real cost in euros, and every outage has to be fixed by me. The single-user
    constraint is deliberate: it explains several choices, such as the absence of high availability
    or Kyverno in audit mode, and I can say what would change with a team.

??? question "What would you do differently?"
    On managed Kubernetes, measure etcd latency from day one, before migrating the GitOps.
    And isolate earlier the components that instrument other processes: an eBPF probe can mimic
    the bug of another tool.

## Infrastructure

??? question "Why Talos rather than Ubuntu + kubeadm?"
    Talos is immutable, with no SSH or shell, driven by API. There is nothing to patch by hand and no
    configuration drift, and the attack surface is minimal. The cluster is recreated often: a declarative
    OS suits that better.

??? question "Why not managed Kubernetes?"
    I tried two. Infomaniak had recurring API timeouts; Scaleway, 0.73 s per etcd request on its
    shared control plane. I came back to Hetzner, where I control the network, Cilium
    with Gateway API included. → [004](decisions/004-retour-hetzner.md)

??? question "A single control plane, isn't that risky?"
    It is: etcd has no fault tolerance. But three control planes triple the most expensive line item,
    for a service with no third-party users. In exchange: a daily etcd snapshot outside Hetzner and a
    full rebuild through Terraform + GitOps. → [002](decisions/002-control-plane-unique.md)

??? question "How did you choose the machine size?"
    By measurement. On 2 vCPU, the control plane hit 132% CPU during the initial
    sync, with no memory alert: the controllers restarted in a chain. On 4 vCPU, it
    drops to 28%, for €3.60 more per month.

## GitOps

??? question "Why ArgoCD? You had moved to Flux."
    I tested Flux for four months, when Flux Operator's Web UI came out. Its architecture
    is better, but it is harder to pick up, and ArgoCD remains the market standard.
    The outage that had accelerated the move actually came from eBPF instrumentation: I verified it
    with an A/B (334 `git fetch`, 0 errors) before switching back. → [005](decisions/005-argocd-flux-argocd.md)

??? question "How do you handle deployment order without `dependsOn`?"
    Waves: the next wave waits until the whole previous one is synced and healthy.
    ArgoCD disables health for child Applications by default, so I re-enable it with a Lua
    script, without which the waves wait for nothing. → [006](decisions/006-app-of-apps-sync-waves.md)

??? question "Tell me about an incident."
    During a migration of the GitOps repo, ArgoCD applied the old definition of an Application to
    the new files and triggered a deletion. Defense in depth worked: the Kyverno policy
    blocked the cascade, and the database was restored to the second from the archived logs,
    with no data loss. Since then, this type of change is made in two commits, and a CI
    check prevents the cause from coming back. → [Incident](incidents.md#prune-saveit)

??? question "Why not the Rendered Manifests Pattern?"
    ArgoCD's native implementation cannot express 23 of my 37 Applications. Rendered in CI, the
    result diverges from the cluster for 17 charts out of 22. A render diff posted as a comment on each PR gives
    me the same review with no artifact to maintain. → [012](decisions/012-pas-de-rendered-manifests.md)

## Platform

??? question "Why Crossplane rather than a shared Helm chart?"
    Crossplane reconciles continuously and exposes a readable status, and the developer only sees a
    15-line API instead of all of a chart's values. The logic is in Python, tested end to
    end on Kind in CI. → [007](decisions/007-crossplane-api-self-service.md)

??? question "Where are the secrets?"
    In Infisical. External Secrets syncs them into the cluster, and Reloader restarts the pods on
    rotation. There is nothing in Git, and nothing in the Terraform state thanks to *write-only* attributes.
    → [009](decisions/009-secrets-infisical-eso.md)

??? question "Kyverno in audit mode, isn't that a bit light?"
    For best practices, yes, and it is deliberate: a single person deploys. The only
    blocking rule protects what cannot be rebuilt, the data, and it has already proven useful in an incident. With
    a team, I would switch to enforcement rule by rule. → [010](decisions/010-kyverno-audit-garde-fous.md)

??? question "How is observability organized?"
    The Victoria stack (metrics, logs, traces) with Grafana and Slack alerts, plus Gatus for
    availability as seen from outside. The application is instrumented with the OpenTelemetry SDK, after the
    removal of an eBPF instrumentation that was corrupting processes. → [011](decisions/011-retrait-obi-sdk-otel.md)

## SRE agent

??? question "You let an LLM modify your production?"
    No. The agent, still in development, runs in manual mode: it investigates and proposes, a human
    decides, and a GitOps-managed resource is only fixed through a pull request. An automatic mode
    will only come after tool-by-tool least-privilege hardening. → [sre-agent](projets/sre-agent.md)

??? question "Why not use kagent or an LLM gateway?"
    A shared platform only pays off with several consumers, and there is only one.
    kagent would replace deterministic, tested code with prompting. I would re-evaluate it behind a
    reliability test, once several agents are in place.
