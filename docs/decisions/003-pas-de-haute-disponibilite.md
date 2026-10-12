# 003 · Pas de haute disponibilité : auto-réparation plutôt que continuité

> **En une phrase :** chaque composant est un point unique de défaillance, par choix économique écrit
> et chiffré ; la plateforme sait se réparer et se reconstruire, pas rester debout pendant une panne.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-09 | `talos-infra`, `talos-argocd` |

## Contexte

La haute disponibilité n'est pas un réglage, c'est une somme de coûts. Le service n'a qu'un utilisateur,
et une indisponibilité n'a aucun coût métier.

## Décision

Aucun dispositif de continuité : une seule réplique par application, une seule instance PostgreSQL, pas
de PodDisruptionBudget, pas d'autoscaling, une seule zone.

**Ce qui est couvert à la place :**

- ArgoCD réconcilie toutes les 1 à 2 minutes : toute modification hors Git est annulée.
- ExternalDNS recâble le DNS tout seul sur la nouvelle IP après une reconstruction complète.
- External Secrets + Reloader : la rotation d'un secret redémarre les pods concernés sans intervention.
- Gatus et des alertes Slack pour détecter, des sauvegardes B2 pour restaurer.

## Ce que chaque levier aurait coûté

| Levier | Coût |
|---|---|
| Plusieurs répliques | ~+0,5 Gi de RAM réservée par réplique et par application |
| PostgreSQL à 2 instances | Un pod de plus, stockage doublé |
| PodDisruptionBudget | Gratuit, mais inutile sans répliques ; il bloquerait même la maintenance des nœuds |
| Autoscaling (HPA) | Suppose de la capacité payée et inutilisée en permanence |
| Control plane HA | ×3 sur le poste le plus cher |
| Multi-zone | Workers dupliqués, trafic inter-zone, volumes non attachables d'une zone à l'autre |

## Compromis et coût

Une perte de nœud ou une mise à jour provoque une interruption, bornée au temps de redémarrage des pods
ailleurs.

## Quand la revoir

Dès qu'un service a des utilisateurs tiers ou qu'un engagement de disponibilité est pris.
