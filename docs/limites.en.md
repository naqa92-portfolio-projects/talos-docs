# Known limits

What the platform does not do, or does not do yet. Some limits are costed choices, others
are open work items.

## Deliberate choices

| Limit | Why it is acceptable here | What would lift it |
|---|---|---|
| No high availability, a single etcd member | Single-user service; rebuild and backups tested ([003](decisions/003-pas-de-haute-disponibilite.md)) | Third-party users or an availability commitment |
| Kyverno best practices in audit, not blocking | Only one person deploys ([010](decisions/010-kyverno-audit-garde-fous.md)) | A second team on the cluster |
| No permanent staging environment | The 24/7 cost of a second environment; a local Kind cluster replays the full GitOps chain to test Backstage | Several contributors, or risky production releases |
| Dependency on SaaS (Infisical, Cloudflare, Backblaze B2, GitHub) | Nothing to operate, free tiers are enough | A sovereignty or network isolation requirement |
| Terraform CI runs *plan* only, triggered by hand | The admin API is filtered to the administrator's IP: a CI runner would always show up as drift there | A list of allowed static IPs |
| Default alert rules follow their upstream repo | Freezing them would mean copying the chart's list, which would drift in turn | An alert threshold changed upstream that causes problems |

## Open work items {#chantiers-ouverts}

- **SRE agent: least privilege.** In production, the agent only notifies and proposes pull
  requests (manual mode). Its rights in the cluster remain broader than this mode requires, and there is
  no global LLM consumption budget yet. A full audit listed the fixes to
  make. This is the next step before any automatic mode.
- **CI lint for Backstage.** It is run locally before each push. The CI job exists as a comment,
  to be re-enabled as soon as there is a second contributor.
- **Application tests for the landing page.** Route tests only; reading the Kubernetes API
  is not covered.
- **A PostgreSQL backup plugin still installed from a raw manifest.** The official Helm chart
  now exists: the migration is ready to do.

## What I would do with a team

- Switch the Kyverno policies to blocking, rule by rule, with a prior audit period.
- Add a pre-production environment and an explicit promotion between environments.
- Put access to the SRE agent behind a per-user identity (OAuth) rather than a shared token.
- Federate LLM calls behind a gateway (budgets, switching between providers) as soon as a
  second consumer exists.
