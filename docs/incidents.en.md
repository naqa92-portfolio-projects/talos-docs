# What broke

Failures are the most instructive part of the project. Each one follows the same format: **symptom →
cause → fix → lesson**.

## A GitOps migration destroyed a database {#prune-saveit}

**Symptom.** While removing Kustomize from the GitOps repo (15 September 2026), two application
namespaces went into `Terminating`. The saveit application's PostgreSQL cluster and its two
volumes were deleted.

**Cause.** Between the merge of a commit and the application of an Application's new definition,
ArgoCD runs the **old** definition on the **new** files. The old one could only read the folder through
its Kustomize file, which had just been deleted. ArgoCD then read the folder
in non-recursive mode, found almost nothing, and deleted the rest (*prune*). The `prune: false`
added in the same commit protected nothing: it only took effect with the new definition.

**Fix.** Point-in-time recovery (PITR) from the PostgreSQL logs archived on B2:
**no data loss**, since the archive chain was continuous up to six seconds after the deletion
started. A Kyverno policy refused the deletion of Crossplane resources and stopped
the cascade.

Three days earlier, a first attempt at the same work had already caused a database to be deleted because of a
misunderstood file pattern. That attempt had been rolled back.

**Lessons.**

1. When a change touches both a definition and the content it reads, it needs **two
   commits**: first the definition, verified on the cluster, then the content.
2. A protection written in the commit that creates the risk does not protect that commit.
3. A render diff in CI compares **final** states: it does not see the transition window.
4. A blocking rule on data deletion is worth more than ten audit rules.
5. A backup is only as good as its restore. The procedure was scripted and had already been used
   two days earlier, during the cluster rebuild.

## A "Healthy" component rendered with its default values {#fenetre-migration}

**Symptom.** The same day, on another piece of work, the Gatus availability probe was recreated without
its configuration, without its volumes and with the default service account, while ArgoCD showed it
`Healthy` and `Synced` the whole time.

**Cause.** The same transition window: the old definition read a restructured values file
and recognized nothing in it. **A render with default values is a valid render**, so there was
no error.

**Fix.** Rolled back in about a minute.

**Lesson.** Judge a migration by the pods' creation date, never by `Healthy` / `Synced` alone.

## The control plane saturated with no memory alert

**Symptom.** At cluster startup, the Gateway became unreachable, and the controller-manager, the
scheduler and the GitOps controllers restarted in a chain.

**Cause.** On a 2 vCPU server, the GitOps cascade pushed control plane CPU to **132%**.
Health probes timed out. No classic memory signal fired.

**Fix.** Move to 4 vCPU (`cx33`): 28% CPU, no more restarts, +€3.60/month.

**Lesson.** Size for the startup peak, and watch control plane CPU before
memory. → [Decision 002](decisions/002-control-plane-unique.md)

## A bug blamed on the wrong component for seven months

**Symptom.** Subprocesses read null bytes instead of their output: ArgoCD (`git
fetch`), PostgreSQL backups, Velero.

**Cause.** The OBI eBPF instrumentation, which places probes in processes. The effect was proven on
PostgreSQL. The problem had first been blamed on ArgoCD, the Talos kernel, barman and the
CloudNativePG plugin.

**Fix.** OBI removed, instrumentation through the OpenTelemetry SDK.

**Lesson.** A component that modifies other processes is suspect by default, and every suspicion is
A/B tested before acting. → [Decisions 005](decisions/005-argocd-flux-argocd.md) and
[011](decisions/011-retrait-obi-sdk-otel.md)

## The managed control plane that was too slow

**Symptom.** On Scaleway Kapsule, GitOps converged poorly on a cluster of only 56 pods.

**Cause.** 0.73 s per etcd request on the free shared control plane. This limit appears
nowhere in the documentation.

**Lesson.** Measure `etcd_request_duration_seconds` before migrating, not after.
→ [Decision 004](decisions/004-retour-hetzner.md)

## ArgoCD cut itself off from the API {#argocd-coupe-api}

**Symptom.** At startup, ArgoCD stopped applying anything, and everything that talks to the Kubernetes API
went down with it.

**Cause.** At equal rank, ArgoCD applies resources in alphabetical order. A network
authorization policy came before the *default-deny* policy that contained the API access. ArgoCD
lost the API before it could apply the rule that gave it back.

**Fix.** The *default-deny* policy is applied in the very first wave.

**Lesson.** Flux applied everything in one pass and hid the flaw. A tool change reveals implicit
dependencies.

## Intermittent 404s on ArgoCD

**Cause.** Under a *wildcard* certificate, HTTP/2 reuses a connection opened for another host
(*connection coalescing*). Requests landed on a TLS chain that had no route to
ArgoCD.

**Fix.** ArgoCD went back to standard TLS termination, which a recent Cilium option
made possible.

## Velero backups refused by Backblaze B2

**Cause.** Velero's AWS plugin always sends a *tagging* header that B2 refuses. The header is
signed: it cannot be removed downstream without breaking the signature.

**Fix.** A signing proxy that removes the header and then re-signs the request. The upstream fix
is merged but not yet released. The exit condition is written down: remove the proxy as soon as a
release contains it.

**Lesson.** Every workaround is recorded in a register with its upstream cause, its link and its
exit condition. Without an exit condition, a workaround is not documented, it is buried.

## Other workarounds kept up to date

| Component | Problem | Workaround |
|---|---|---|
| Authentik | The default shared memory (64 Mi) fills up, causing 502s | 256 Mi memory volume mounted on `/dev/shm` |
| Backstage | OIDC discovery is only attempted at startup, with no retry | An init container waits until Authentik responds |
| n8n | OIDC SSO is reserved for the Enterprise edition | OIDC routes added through the supported external *hooks* |
| Gatus | The configuration written by the sidecar is only read on the second start | Restart at boot kept, upstream fix tracked |
| Let's Encrypt | Quota of 5 certificates per week, used up by rebuilds | Certificates backed up before destruction, re-injected on rebuild |
