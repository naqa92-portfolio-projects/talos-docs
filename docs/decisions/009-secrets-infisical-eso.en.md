# 009 · Secrets in Infisical, synced by External Secrets

> **In one sentence:** no secret in Git or in the Terraform state; the cluster fetches its
> secrets from an external vault and restarts pods when they change.

| Status | Date | Repo |
|---|---|---|
| Adopted | 2026-02 | `talos-infra`, `talos-argocd` |

## Decision

- **Infisical** (SaaS, EU region) is the vault. **External Secrets Operator** creates the Kubernetes
  secrets from the vault. **Reloader** restarts pods when a secret changes.
- Terraform deposits only **one bootstrap secret** (the Infisical machine identity), via
  *write-only* attributes: the value never ends up in the Terraform state.
- Crossplane compositions create an application's ExternalSecrets themselves.

## Alternatives rejected

| Alternative | Why not |
|---|---|
| SOPS / Sealed Secrets | Encrypted secrets in Git stay in the history, and key rotation is manual |
| Self-hosted Infisical | One more critical service to operate and back up; the free tier is enough |
| Vault / OpenBao | Oversized for one person |

## Trade-offs and cost

Cluster startup depends on an external SaaS: if Infisical is unavailable, new secrets
are not created (those already present stay in place).
