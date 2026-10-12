# Lessons learned

Outages are the most instructive part of the project. Each one follows the same format: **symptom →
cause → fix → lesson**.

## A GitOps migration that triggered a deletion, restored with no loss {#prune-saveit}

**Context.** While removing Kustomize from the GitOps repo, ArgoCD deleted (*pruned*) the
resources of two applications, including their PostgreSQL database.

**Cause.** Between the merge of a commit and the application of an Application's new definition,
ArgoCD runs the **old** definition against the **new** files. The old one could only read the
folder through its Kustomize file, which had just been deleted. ArgoCD read the folder in
non-recursive mode, found almost nothing, and removed the rest. The `prune: false` added in the
same commit could not protect anything: it only took effect with the new definition.

**What worked.**

- **Guardrail**: the Kyverno policy that protects data refused the deletion of the Crossplane
  resources and stopped the cascade.
- **Restore**: point-in-time recovery (PITR) from the PostgreSQL logs continuously archived to B2,
  using the scripted procedure already exercised during cluster rebuilds.
  **No data loss**: the archive chain covered up to six seconds after the deletion began.

**What I changed afterwards.**

1. A change that touches both a definition and the content it reads is made in **two
   commits**: first the definition, verified on the cluster, then the content.
2. A CI check now verifies that every Application explicitly declares its read mode,
   and forbids Kustomize files from coming back.
3. A migration is judged on pod creation time, not only on `Healthy` / `Synced`.

**What it shows.** A protection written in the commit that creates the risk does not cover that commit,
and a render diff in CI cannot see the transition window, since it compares final states.
Defense in depth (admission guardrail + continuous backup + rehearsed restore) did
its job.

## A "Healthy" component rendered with its default values {#fenetre-migration}

**Symptom.** While evaluating another pattern, the Gatus availability probe was recreated
without its configuration, without its volumes and with the default service account, while ArgoCD
showed it as `Healthy` and `Synced` throughout.

**Cause.** The same transition window: the old definition read a restructured values file
and no longer recognized anything in it. **A render with default values is a valid render**, so
no error.

**Fix.** Rolled back in about a minute.

**Lesson.** Judge a migration on pod creation time, never on `Healthy` / `Synced` alone.

## The control plane saturated with no memory alert

**Symptom.** At cluster startup, the Gateway became unreachable, and the controller-manager, the
scheduler and the GitOps controllers restarted in a chain.

**Cause.** On a 2 vCPU server, the GitOps cascade pushed control plane CPU to **132%**.
Health probes timed out. No classic memory signal fired.

**Fix.** Move to 4 vCPU (`cx33`): 28% CPU, no more restarts, +€3.60/month.

**Lesson.** Size for the startup peak, and watch control plane CPU before
memory. → [Decision 002](decisions/002-control-plane-unique.md)

## An "impossible" bug, tracked down to its real cause

**Symptom.** Subprocesses read null bytes instead of their output: ArgoCD (`git
fetch`), PostgreSQL backups, Velero.

**Cause.** The OBI eBPF instrumentation, which places probes inside processes. I proved it with an A/B
test on PostgreSQL: excluding the pod from instrumentation was enough to restore backups. The symptom
first pointed to ArgoCD, the Talos kernel or the CloudNativePG plugin.

**Fix.** Remove OBI, instrument with the OpenTelemetry SDK instead.

**Lesson.** A component that modifies other processes is suspect by default, and every suspicion is
tested with an A/B before acting. → [Decisions 005](decisions/005-argocd-flux-argocd.md) and
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

**Cause.** Within the same wave, ArgoCD applies resources in alphabetical order. A network
allow policy came before the *default-deny* policy that contained the API access. ArgoCD
lost the API before it could apply the rule that gave it back.

**Fix.** The *default-deny* policy is applied in the very first wave.

**Lesson.** Flux applied everything in a single pass and hid the flaw. Changing tools reveals
implicit dependencies.

## Intermittent 404s on ArgoCD

**Cause.** Under a *wildcard* certificate, HTTP/2 reuses an open connection for another host
(*connection coalescing*). Requests landed on a TLS chain that had no route to
ArgoCD.

**Fix.** ArgoCD went back to standard TLS termination, which a recent Cilium option
made possible.

## Velero backups rejected by Backblaze B2

**Cause.** Velero's AWS plugin always sends a *tagging* header that B2 rejects. The header is
signed: it cannot be removed downstream without breaking the signature.

**Fix.** A signing proxy that strips the header and re-signs the request. The upstream fix
is merged but not yet released. The exit condition is written down: remove the proxy as soon as a
release contains it.

**Lesson.** Every workaround is logged in a register with its upstream cause, its link and its
exit condition. Without an exit condition, a workaround is not documented, it is buried.

## Other workarounds kept up to date

| Component | Problem | Workaround |
|---|---|---|
| Authentik | The default shared memory (64 Mi) saturates, causing 502s | 256 Mi memory volume mounted on `/dev/shm` |
| Backstage | OIDC discovery is only attempted at startup, with no retry | An init container waits for Authentik to respond |
| n8n | OIDC SSO is reserved for the Enterprise edition | OIDC routes added through the supported external *hooks* |
| Gatus | The configuration written by the sidecar is only read on the second start | Restart at boot kept, upstream fix tracked |
| Let's Encrypt | Quota of 5 certificates per week, exhausted by rebuilds | Certificates backed up before destruction, reinjected on rebuild |
