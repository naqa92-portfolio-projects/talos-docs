# Interview questions

The questions I get asked, or should be asked, with the short answer. The detail is in
each decision page.

## Overview

??? question "In one sentence, what is Taloslab?"
    A complete Kubernetes platform that I run solo on Hetzner for €41/month. A developer
    fills in a Backstage form, and their application is online with its database, DNS, TLS,
    secrets and monitoring, entirely through GitOps.

??? question "Why build all this for a single user?"
    To practice the platform engineer job on a real system, not on a tutorial. Every
    choice has a real cost in euros, and every failure has to be fixed by me. The
    single-user constraint is deliberate: it explains several choices, like the absence of high availability
    or Kyverno in audit mode, and I can say what would change with a team.

??? question "What would you do differently?"
    Measure before blaming. I changed GitOps tool because of a bug I had blamed on
    ArgoCD, when it came from the eBPF instrumentation. And on a managed Kubernetes, measure etcd
    latency before migrating, not after.

## Infrastructure

??? question "Why Talos rather than Ubuntu + kubeadm?"
    Talos is immutable, with no SSH or shell, driven by API. There is nothing to patch by hand and no
    configuration drift, and the attack surface is minimal. The cluster is rebuilt often: a declarative OS
    suits that better.

??? question "Why not a managed Kubernetes?"
    I tried two. Infomaniak had recurring API timeouts; Scaleway had 0.73 s per etcd request
    on its shared control plane. I went back to Hetzner, where I control the network, including Cilium
    with Gateway API. → [004](decisions/004-retour-hetzner.md)

??? question "A single control plane, isn't that risky?"
    Yes: etcd has no fault tolerance. But three control planes triple the most expensive line item,
    for a service with no third-party users. In return: a daily etcd snapshot outside Hetzner and a
    full rebuild through Terraform + GitOps. → [002](decisions/002-control-plane-unique.md)

??? question "How did you pick the machine size?"
    By measuring. On 2 vCPU, the control plane climbed to 132% CPU during the initial
    sync, with no memory alert: the controllers restarted in a chain. On 4 vCPU, it
    drops to 28%, for €3.60 more per month.

## GitOps

??? question "Why ArgoCD? You had switched to Flux."
    I switched to Flux to escape a bug. When the same signature appeared on other components,
    proven to come from the eBPF instrumentation, the reason for leaving disappeared. I ran another
    A/B proof (334 `git fetch`, 0 errors) before going back. → [005](decisions/005-argocd-flux-argocd.md)

??? question "How do you handle deployment order without `dependsOn`?"
    Waves: the next wave waits until the whole previous one is synced and healthy.
    ArgoCD disables health for child Applications by default, so I re-enable it with a Lua
    script, without which the waves wait for nothing. → [006](decisions/006-app-of-apps-sync-waves.md)

??? question "Tell me about an incident."
    During a migration of the GitOps repo, ArgoCD ran an Application's old definition
    on the new files and deleted a PostgreSQL database. I restored it with no loss thanks to the
    logs archived on B2. A Kyverno policy stopped the cascade. Lesson: split the definition change
    and the content change into two commits. → [Incident](incidents.md#prune-saveit)

??? question "Why not the Rendered Manifests Pattern?"
    ArgoCD's native implementation cannot express 23 of my 37 Applications. Rendered in CI, the
    result diverges from the cluster for 17 charts out of 22. A render diff posted as a comment on each PR
    gives me the same review with no artifact to maintain. → [012](decisions/012-pas-de-rendered-manifests.md)

## Platform

??? question "Why Crossplane rather than a shared Helm chart?"
    Crossplane reconciles continuously and reports a readable status, and the developer sees only a 15-line
    API instead of all of a chart's values. The logic is in Python, tested end to end on
    Kind in CI. → [007](decisions/007-crossplane-api-self-service.md)

??? question "Where are the secrets?"
    In Infisical. External Secrets syncs them into the cluster, and Reloader restarts the pods on
    rotation. There is nothing in Git, and nothing in the Terraform state thanks to *write-only* attributes.
    → [009](decisions/009-secrets-infisical-eso.md)

??? question "Kyverno in audit mode, isn't that a bit light?"
    For best practices, yes, and it is deliberate: only one person deploys. The only
    blocking rule protects what cannot be rebuilt, the data, and it has already helped in an incident. With
    a team, I would switch to blocking rule by rule. → [010](decisions/010-kyverno-audit-garde-fous.md)

??? question "How is observability organized?"
    The Victoria stack (metrics, logs, traces) with Grafana and Slack alerts, plus Gatus for
    availability seen from the outside. The application is instrumented with the OpenTelemetry SDK, after the
    removal of an eBPF instrumentation that corrupted processes. → [011](decisions/011-retrait-obi-sdk-otel.md)

## SRE agent

??? question "You let an LLM change your production?"
    No. In production, the agent investigates read-only and proposes. A human decides, and a resource managed
    through GitOps is only fixed by pull request. The next work item, before any automatic mode, is
    to tighten its rights to the strict minimum. → [sre-agent](projets/sre-agent.md)

??? question "Why not use kagent or an LLM gateway?"
    A shared platform only pays off with several consumers, and there is only one.
    kagent would replace deterministic, tested code with prompts. I would re-evaluate it behind a reliability
    test, once several agents are in place.
