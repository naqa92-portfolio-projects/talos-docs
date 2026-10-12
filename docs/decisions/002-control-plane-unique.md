# 002 · Un seul control plane, `cx33` minimum

> **En une phrase :** un control plane au lieu de trois divise le coût du poste le plus cher, et la
> sauvegarde etcd quotidienne couvre le vrai besoin de reprise.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-09 | `talos-infra` |

## Contexte

Un control plane hautement disponible demande trois nœuds (etcd a besoin d'une majorité). Le cluster
est mono-utilisateur, et il est régulièrement détruit et recréé.

## Décision

- **Un seul control plane**, dédié à Kubernetes : il refuse les applications (*taint* `NoSchedule`).
  Deux workers portent la charge.
- Taille plancher **`cx33`** (4 vCPU / 8 Go), fixée par la mesure.

## La mesure qui a fixé la taille

Sur un `cx23` (2 vCPU), la synchronisation GitOps initiale a saturé le control plane à **132 % de CPU**.
Aucune alerte mémoire ne s'est déclenchée : ni `OOMKilled`, ni `MemoryPressure`. Le symptôme réel était
une cascade de redémarrages (controller-manager, scheduler, contrôleurs GitOps) par expiration des
sondes de santé.

Après passage en `cx33` : 28 % de CPU, plus aucun redémarrage. Le redimensionnement s'est fait sur
place en 1 min 9 s, sans perte d'etcd. Coût de l'escalade : +3,60 €/mois.

**Leçon :** sur un control plane, le goulot est le CPU au démarrage, pas la mémoire en régime normal.
Il faut dimensionner sur le pic de la cascade, pas sur le repos.

## Alternatives écartées

| Alternative | Pourquoi non |
|---|---|
| 3 control planes | ×3 sur le poste le plus cher, pour un service sans utilisateur tiers |
| Control planes qui portent aussi les applications | C'était l'ancienne topologie : les applications concurrençaient etcd et l'API |

## Compromis et coût

**etcd n'a qu'un membre : aucune tolérance de panne.** Perdre le nœud impose de reconstruire le
cluster, pas de le réparer sur place. Le snapshot etcd quotidien vers Backblaze B2 (hors Hetzner) et la
reconstruction par Terraform + GitOps rendent ce risque acceptable.

Le redimensionnement est aussi **irréversible** : Hetzner agrandit le disque mais ne sait pas le réduire.

## Quand la revoir

Dès que le cluster porte un service avec des utilisateurs tiers, ou un engagement de disponibilité.
