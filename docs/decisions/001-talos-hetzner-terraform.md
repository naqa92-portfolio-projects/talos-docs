# 001 · Talos Linux sur Hetzner, provisionné par un module Terraform maintenu

> **En une phrase :** j'utilise un module Terraform communautaire maintenu plutôt que d'écrire ou de
> forker mon propre provisioning, et je ne surcharge que ce qui est cassé.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-02, reconduit en 2026-09 | `talos-infra` |

## Contexte

Il faut un cluster Kubernetes reproductible, qu'on peut détruire le soir et recréer le lendemain pour
maîtriser le coût. Kubernetes « à la main » sur des VM demande de gérer l'OS, les mises à jour, SSH et
le durcissement.

## Décision

- **Talos Linux** comme OS : immuable, sans SSH ni shell, entièrement piloté par API. Il n'y a rien à
  patcher à la main, et la surface d'attaque est minimale.
- **Module Terraform `hcloud-k8s/kubernetes/hcloud`** (version épinglée) : il installe Talos, Cilium,
  les intégrations Hetzner (load balancer, volumes chiffrés) et la sauvegarde etcd.
- Principes posés dès le premier PRD : **pas de Terragrunt, pas de fork, pas de SOPS**.

## Alternatives écartées

| Alternative | Pourquoi non |
|---|---|
| Écrire mes propres ressources Terraform | Réinventer ce qu'un module maintenu fait déjà, et le maintenir seul |
| Forker le module | Perdre les mises à jour amont ; chaque bump devient une fusion manuelle |
| Kubernetes managé | Essayé deux fois (Infomaniak, Scaleway), voir [décision 004](004-retour-hetzner.md) |

## Compromis et coût

Les valeurs par défaut du module n'étaient pas toutes utilisables. Le type de serveur du builder
d'image (`cpx11`) n'était plus commandable en Europe, et la version de Talos par défaut dépassait le
client `talosctl` disponible. Les deux sont surchargés explicitement, avec la raison en commentaire.

## Quand la revoir

Si le module n'est plus maintenu, ou si un besoin multi-cluster apparaît (Terragrunt redeviendrait
pertinent).
