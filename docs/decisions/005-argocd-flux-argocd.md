# 005 · ArgoCD → Flux → ArgoCD

> **En une phrase :** j'ai testé Flux en conditions réelles pendant quatre mois. Son architecture est
> meilleure, mais ArgoCD est le standard du marché, et la panne qui m'avait poussé à partir s'est
> révélée étrangère à ArgoCD : je suis revenu.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu : ArgoCD | Flux le 2026-05-30, retour ArgoCD le 2026-09-13 | `talos-argocd` (ex-`talos-fluxcd`) |

## Pourquoi j'ai testé Flux

Deux raisons, arrivées en même temps :

1. **La sortie de la [Web UI de Flux Operator](https://fluxoperator.dev/web-ui/).** L'absence
   d'interface était le principal reproche fait à Flux face à ArgoCD. Une UI intégrée, avec SSO OIDC
   et un graphe du pipeline de livraison, levait cet argument. C'était le moment de comparer les deux
   outils sur un vrai cluster.
2. **Une panne récurrente côté ArgoCD.** Le composant qui lit Git (`repo-server`) échouait au
   démarrage du cluster (`git fetch exit 128`). Un PoC Flux sur le même cluster donnait 0 erreur,
   contre 16 côté ArgoCD.

## Ce que quatre mois de Flux m'ont appris

**Flux a la meilleure architecture.** Des contrôleurs spécialisés (sources, Kustomize, Helm) plutôt
qu'un serveur central. Des dépendances natives entre composants (`dependsOn` + `wait`), des contrôles
de santé en CEL, et l'application d'un composant en une seule passe.

**Mais il est plus difficile à prendre en main.** Il y a plus d'objets à comprendre, l'état est réparti
entre plusieurs contrôleurs, et le debug passe davantage par la CLI que par une vue d'ensemble.

## Pourquoi je suis revenu sur ArgoCD

1. **Le marché.** ArgoCD est le moteur GitOps le plus demandé dans les équipes plateforme. Le maîtriser
   en profondeur, ordonnancement, santé des Applications, diff de rendu en CI, a plus de valeur pour mes
   missions.
2. **La panne n'était pas d'ArgoCD.** La même signature (octets nuls dans la sortie d'un sous-processus)
   est apparue sur d'autres composants. Je l'ai isolée par A/B : elle venait de l'instrumentation eBPF
   OBI ([décision 011](011-retrait-obi-sdk-otel.md)). Avant de revenir, j'ai refait une preuve sur un banc
   dédié : **334 `git fetch` instrumentés, 0 erreur**. Puis, en production, 0 occurrence sur plus de
   40 000 lignes de logs, sur cinq démarrages complets.

## Décision

Retour sur ArgoCD. Ce n'est pas un *revert* : le dépôt a été réécrit, et il reprend ce que Flux faisait
mieux. L'ordonnancement du démarrage devient explicite ([décision 006](006-app-of-apps-sync-waves.md)),
et des contrôles de santé écrits en Lua remplacent ceux de Flux en CEL.

## Compromis et coût

- ArgoCD n'a pas de `dependsOn` entre Applications : il a fallu des vagues et un contrôle de santé Lua.
- Deux migrations complètes du GitOps. Les deux ont été absorbées sans toucher aux applications.

## Ce que j'en retiens

- Comparer deux outils sur un vrai cluster apprend plus qu'une matrice de fonctionnalités.
- Un composant qui instrumente d'autres processus est un suspect par défaut dans tout diagnostic
  « impossible ». Chaque suspicion se vérifie par A/B avant d'agir.
