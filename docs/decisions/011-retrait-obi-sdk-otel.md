# 011 · Retirer l'instrumentation eBPF au profit du SDK OpenTelemetry

> **En une phrase :** l'instrumentation eBPF « sans toucher au code » s'est révélée intrusive ; pour une
> application dont je contrôle le code, le SDK OpenTelemetry fait mieux, sans privilège.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-09-12 | `talos-argocd`, application saveit |

## Contexte

OBI (OpenTelemetry eBPF Instrumentation) trace les applications sans modifier leurs images : un
DaemonSet privilégié pose des sondes dans les processus. Adopté en février 2026, il couvrait au départ
tous les namespaces.

## Ce que j'ai mesuré

- Les sondes eBPF perturbaient les lectures de pipe des processus instrumentés. Je l'ai **prouvé** par
  A/B sur les sauvegardes PostgreSQL : exclure le pod de l'instrumentation suffisait à les rétablir.
- Le symptôme ressemblait à des bugs d'autres composants (ArgoCD, noyau Talos, plugin CloudNativePG) :
  un DaemonSet privilégié qui écrit dans la mémoire des processus brouille tous les diagnostics.
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
