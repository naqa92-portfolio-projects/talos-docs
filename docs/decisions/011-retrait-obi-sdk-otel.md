# 011 · Retirer l'instrumentation eBPF au profit du SDK OpenTelemetry

> **En une phrase :** l'instrumentation « sans toucher au code » corrompait des processus depuis sept
> mois ; pour une seule application dont je contrôle le code, le SDK fait mieux, sans privilège.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-09-12 | `talos-argocd`, application saveit |

## Contexte

OBI (OpenTelemetry eBPF Instrumentation) trace les applications sans modifier leurs images : un
DaemonSet privilégié pose des sondes dans les processus. Adopté en février 2026, il couvrait au départ
tous les namespaces.

## Ce qu'il a cassé

- Les lectures de pipe des processus instrumentés étaient corrompues. Effet **prouvé** sur les
  sauvegardes PostgreSQL : exclure le pod suffisait à les rétablir.
- La même signature a pesé dans deux migrations ([005](005-argocd-flux-argocd.md),
  [004](004-retour-hetzner.md)), après avoir été attribuée tour à tour à barman, au noyau Talos et au
  plugin CloudNativePG.
- Environ 350 Mi de mémoire par nœud pour tracer une seule application Django.

## Décision

Retrait d'OBI, instrumentation de l'application avec le **SDK OpenTelemetry** : spans ORM, HTTP et
worker, et `trace_id` dans chaque log émis sous un span. Les traces partent vers VictoriaTraces, les
métriques vers VictoriaMetrics.

## Compromis et coût

Il faut modifier le code pour tracer. L'argument « zéro modification » ne vaut que pour des applications
tierces, et il n'y en a pas ici.

## Ce que j'en retiens

Avant de retirer OBI, chaque suspicion a été testée en A/B : une prouvée (PostgreSQL), deux non
reproduites (ArgoCD, Authentik). Un contournement de 863 lignes posé pour Authentik s'est révélé
inutile et a été retiré, preuve à l'appui.
