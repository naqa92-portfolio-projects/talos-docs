# 010 · Kyverno : audit pour les bonnes pratiques, blocage pour les données

> **En une phrase :** les règles de bonnes pratiques signalent sans bloquer, parce qu'il n'y a qu'un
> utilisateur ; la seule règle bloquante protège ce qui est irrécupérable, les données.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-04 | `talos-argocd` |

## Décision

| Politique | Mode | Pourquoi |
|---|---|---|
| Conteneurs non privilégiés, non root, sans namespaces hôte, avec requests / limits, registres autorisés | **Audit** | La visibilité suffit en mono-utilisateur ; le blocage a du sens quand plusieurs équipes déploient |
| Validation des claims `App` / `Database`, unicité des noms d'hôte | **Audit** | Même raison |
| Suppression d'une ressource Crossplane sans annotation `allow-delete` | **Blocage** | Une base supprimée par erreur ne se reconstruit pas depuis Git |

Les politiques sont écrites en **CEL** (le langage d'expression natif de Kubernetes), l'ancien format
`ClusterPolicy` étant en voie de retrait.

## Ce que la règle bloquante a déjà évité

Lors de l'[incident du 2026-09-15](../incidents.md#prune-saveit), c'est cette politique qui a refusé
la suppression des ressources Crossplane et arrêté la cascade.

## Compromis et coût

Les mauvaises pratiques sont visibles dans les rapports, pas empêchées. Un déploiement non conforme
passe.

## Quand la revoir

Dès qu'une deuxième personne ou équipe déploie sur le cluster : passage en blocage, règle par règle.
