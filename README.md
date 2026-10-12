# Taloslab : étude de cas

Plateforme Kubernetes de bout en bout, construite et opérée seul sur Hetzner : Talos Linux, Cilium,
ArgoCD, Crossplane, Backstage et un agent SRE piloté par LLM.

**Site : [naqa92-portfolio-projects.github.io/talos-docs](https://naqa92-portfolio-projects.github.io/talos-docs/)**
· [English](https://naqa92-portfolio-projects.github.io/talos-docs/en/)

Ce dépôt ne contient que la documentation publique : le pourquoi des choix, leurs coûts et ce qui a
cassé. Le code de la plateforme vit dans des dépôts privés.

## Prévisualiser en local

```bash
uv venv && uv pip install -r requirements.txt
uv run mkdocs serve
```

Chaque page existe en français (`page.md`) et en anglais (`page.en.md`).
