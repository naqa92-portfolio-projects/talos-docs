# 006 · App-of-apps découverte par dossier, ordonnée par vagues

> **En une phrase :** ajouter un composant revient à déposer un fichier dans un dossier, et l'ordre de
> démarrage est garanti par des vagues qui attendent la bonne santé de la vague précédente.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-09 | `talos-argocd` |

## Contexte

Une plateforme a des dépendances strictes : les types (CRD) d'un opérateur doivent exister avant les
ressources qui les utilisent, et un certificat a besoin du secret Cloudflare, lui-même fourni par
External Secrets. Lors du premier passage sur ArgoCD, 26 synchronisations lancées en même temps avaient
mis un cluster à genoux.

## Décision

- **Une Application racine** découvre récursivement chaque `application.yaml`. Aucun registre central.
- **Des vagues** (`sync-wave`) : la vague N+1 ne démarre qu'une fois la vague N synchronisée **et**
  en bonne santé. ArgoCD a désactivé par défaut la santé des Applications enfants : je la restaure par un
  script Lua, sans quoi les vagues n'attendent rien.
- La racine **se gère elle-même** : elle est l'une de ses propres sources.
- **Une convention unique** : `<catégorie>/<composant>/application.yaml` + `values.yaml` + `resources/`.
  Pas d'exception, pas de dossier parapluie.

## Alternatives écartées

| Alternative | Pourquoi non |
|---|---|
| ApplicationSets | Plus de concurrence sur les opérations Git (c'était un facteur du bug de la [décision 005](005-argocd-flux-argocd.md)), et plus indirect à lire |
| Progressive Syncs (RollingSync) | Fonction encore derrière un flag, et elle **désactive l'auto-sync** des Applications générées : on perdrait l'auto-réparation |
| Kustomize | Retiré : chaque fichier était déclaré deux ou trois fois (le fichier, l'index du dossier, l'index parent) |

## Compromis et coût

- Le démarrage à froid est sérialisé, donc plus lent qu'un démarrage parallèle, mais il converge seul
  (33/33 Applications au dernier démarrage complet).
- La découverte automatique a un revers : une erreur de chemin peut déclencher une suppression
  (*prune*). C'est contenu à trois niveaux : un contrôle CI sur chaque Application, une politique
  Kyverno qui bloque la suppression des ressources portant des données, et des sauvegardes continues
  ([incident maîtrisé](../incidents.md#prune-saveit)).
