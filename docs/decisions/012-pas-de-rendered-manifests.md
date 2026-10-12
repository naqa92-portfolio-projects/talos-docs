# 012 · Pas de *Rendered Manifests Pattern*, un diff de rendu en CI à la place

> **En une phrase :** committer les manifestes rendus coûtait plus qu'il ne rapportait à cette
> échelle ; un diff du rendu en commentaire de chaque PR donne la même revue sans l'artefact.

| Statut | Date | Dépôt |
|---|---|---|
| Écarté deux fois, définitivement le 2026-09-15 | | `talos-argocd` |

## Contexte

Le *Rendered Manifests Pattern* consiste à committer le YAML final rendu (Helm, Kustomize) pour que Git
montre exactement ce qui tourne. Le pattern est très recommandé dans la littérature GitOps.

## Pourquoi non

1. **L'implémentation native d'ArgoCD (Source Hydrator) ne sait pas exprimer ce dépôt** : 23 Applications
   sur 37 combinent un chart distant, des values et des manifestes locaux. Les deux manques sont des
   demandes ouvertes en amont.
2. **Rendre en CI ne reproduit pas fidèlement le cluster** : 17 charts sur 22 lisent les capacités réelles
   de l'API (`.Capabilities`), qu'une CI ne connaît pas, et 7 émettent des hooks Helm que le pattern perd.
3. **La tentative a cassé la production.** Pendant la migration, un composant a été rendu avec ses
   valeurs par défaut tout en restant affiché `Healthy` (voir [incidents](../incidents.md#fenetre-migration)).

## Décision

`argocd-diff-preview` en CI : chaque PR reçoit en commentaire le diff des manifestes rendus des deux
côtés. C'est la même information pour la revue, sans artefact à maintenir.

## Compromis et coût

Pas d'historique Git des manifestes rendus : impossible de faire un `git blame` sur un objet final.

## Quand la revoir

S'il faut un jour remonter l'historique des manifestes rendus pour trouver une régression, si un
démarrage à froid échoue faute d'accès aux dépôts de charts, ou si les demandes amont sont livrées.
