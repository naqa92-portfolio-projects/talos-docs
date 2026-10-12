# talos-backstage: the developer portal

> **A Backstage form becomes a GitOps pull request: a repo, an application or a database
> in self-service, with SSO and an MCP server to drive the portal from an AI assistant.**

| | |
|---|---|
| Role | Self-service interface on top of the Crossplane API |
| Tools | Backstage (new plugin system), TypeScript, Authentik OIDC, PostgreSQL, Kind |
| Visibility | Private |

## What a developer sees

- **Create a repo** from a Python/Flask or Go skeleton, with CI and Dockerfile ready.
- **Create an application or a database.** The forms are generated automatically from
  the Crossplane APIs: adding a field to the API adds it to the form.
- **Modify or delete** a component, always by pull request.
- **See the real state**: ArgoCD sync, pods, Kyverno policies, Crossplane resources,
  Grafana dashboards, DORA metrics.

## What I built

- **A scaffolder module** with three homemade actions and a processor that cleans the generated
  forms: it removes technical settings and carries over the API's required fields.
- **A plugin published on npm** that syncs Authentik users and groups into the catalog
  ([open source](https://github.com/naqa92/authentik-backstage-plugin)).
- **A faithful local environment**: a Kind cluster with Cilium and Gateway API, which replays the full
  GitOps chain to test a form end to end.

## Notable choices

| Choice | Reason |
|---|---|
| Plugins declared explicitly | Automatic loading does not pick up frontend plugins in the version used |
| ArgoCD plugin pinned, excluded from Renovate | Later versions require a newer Backstage and would duplicate the framework's packages. The exit condition is written down |
| Runtime image not *distroless* | TechDocs generates the documentation in the container (Python + MkDocs); Alpine is incompatible with a native module |
| Blocking vulnerability scan before publishing | The only quality check run in CI: the image build is the job that counts |
| PostgreSQL without backup in production | All the portal's state can be rebuilt from GitHub and Authentik's configuration |
| Kubernetes read access through an aggregated role, no `*` | A `*` rule also gives access to Secrets |
