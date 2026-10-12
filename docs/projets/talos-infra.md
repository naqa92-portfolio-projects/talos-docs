# talos-infra : l'infrastructure

> **Un `task apply` crée les serveurs, installe l'OS et Kubernetes, puis passe la main à ArgoCD.**

| | |
|---|---|
| Rôle | Provisionner le cluster et l'amorcer en GitOps |
| Outils | Terraform, module `hcloud-k8s`, Talos Linux, Cilium, Taskfile, Devbox |
| Visibilité | Privé |

## Ce qu'il contient

- **Terraform** : serveurs Hetzner (1 control plane + 2 workers), réseau, pare-feu, load balancer,
  installation d'ArgoCD et de l'Application racine. L'état Terraform est stocké hors de Hetzner
  (Backblaze B2).
- **Le cycle de vie complet en tâches** : `apply` (création + convergence), `destroy` (destruction
  propre, DNS neutralisé), restauration PostgreSQL, vérification du quota TLS, tests des politiques
  réseau et Kyverno.
- **Les runbooks d'exploitation** : reprise après sinistre, sauvegarde et restauration, ajout d'un
  composant ou d'une route, politiques réseau, dimensionnement.
- **Les PRD** : près de 50 documents de conception clôturés, avec leurs décisions, leurs jalons et
  leurs résultats mesurés.

## Points à retenir

- **Le cluster est jetable.** Il est détruit et recréé régulièrement pour maîtriser le coût. Tout ce qui
  doit survivre (état Terraform, snapshots etcd, journaux PostgreSQL, sauvegardes Velero) vit sur
  Backblaze B2.
- **Aucun secret dans l'état Terraform** grâce aux attributs *write-only*.
- **Terraform amorce ArgoCD puis le lâche** : ArgoCD se gère ensuite lui-même depuis Git, sans conflit
  de propriété avec Terraform.
- **Un registre des contournements**, chacun avec sa cause amont et sa condition de sortie, relu à chaque
  mise à jour du composant concerné.

## Décisions liées

[001](../decisions/001-talos-hetzner-terraform.md) · [002](../decisions/002-control-plane-unique.md) ·
[003](../decisions/003-pas-de-haute-disponibilite.md) · [004](../decisions/004-retour-hetzner.md)
