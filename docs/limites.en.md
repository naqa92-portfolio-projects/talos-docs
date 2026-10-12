# Accepted limits and next steps

What the platform does not do, or does not do yet. Most are costed choices; the rest form the
roadmap.

## Deliberate choices

| Limit | Why it is acceptable here | What would lift it |
|---|---|---|
| No high availability, a single etcd member | Single-user service; rebuild and backups tested ([003](decisions/003-pas-de-haute-disponibilite.md)) | Third-party users or an availability commitment |
| Kyverno best practices in audit, not enforcement | A single person deploys ([010](decisions/010-kyverno-audit-garde-fous.md)) | A second team on the cluster |
| No permanent staging environment | The 24/7 cost of a second environment; a local Kind cluster replays the full GitOps chain to test Backstage | Several contributors, or risky production releases |
| Dependence on SaaS (Infisical, Cloudflare, Backblaze B2, GitHub) | Nothing to operate, free tiers are enough | A sovereignty or network isolation requirement |
| Terraform CI runs *plan* only, triggered manually | The admin API is filtered to the administrator's IP: a CI runner would always show up as drift | A list of static allowed IPs |
| Default alert rules follow their upstream repo | Freezing them would mean copying the chart's list, which would drift in turn | An upstream-modified alert threshold that causes problems |

## Roadmap {#chantiers-ouverts}

- **SRE agent (in development).** It runs in manual mode: it investigates, notifies and proposes
  pull requests, and a human decides. Before considering an automatic mode: tool-by-tool least-privilege
  hardening, and a global LLM consumption budget.
- **Lint in CI for Backstage.** Run locally before every push; the CI job is ready, to be enabled as soon
  as a second contributor arrives.
- **Move the PostgreSQL backup plugin to its official Helm chart**, now available.

## What I would do with a team

- Switch the Kyverno policies to enforcement, rule by rule, with a prior audit period.
- Add a pre-production environment and an explicit promotion between environments.
- Put access to the SRE agent behind a per-user identity (OAuth) rather than a shared token.
- Federate LLM calls behind a gateway (budgets, provider failover) as soon as a
  second consumer exists.
