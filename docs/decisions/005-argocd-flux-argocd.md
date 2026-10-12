# 005 · ArgoCD → Flux → ArgoCD

> **En une phrase :** j'ai quitté ArgoCD à cause d'un bug que j'avais attribué à ArgoCD ; quatre mois
> plus tard, la même signature est apparue ailleurs, prouvée comme venant d'un autre composant, et je
> suis revenu sur ArgoCD après une preuve A/B.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu : ArgoCD | Flux le 2026-05-30, retour ArgoCD le 2026-09-13 | `talos-argocd` (ex-`talos-fluxcd`) |

## Contexte

Au printemps 2026, le composant d'ArgoCD qui lit Git (`repo-server`) échouait de façon chronique au
démarrage du cluster (`git fetch exit 128`, sortie remplie d'octets nuls). Quatre contournements
empilés ne suffisaient pas. Un PoC Flux sur le même cluster donnait **0 erreur**, contre 16 côté
ArgoCD. J'ai migré.

## Ce qui a changé le diagnostic

1. La même signature (octets nuls dans la sortie d'un sous-processus) a été **prouvée** sur les
   sauvegardes PostgreSQL comme un effet de l'instrumentation eBPF OBI. Exclure le pod d'OBI
   suffisait à faire passer la sauvegarde. Or OBI instrumentait aussi le namespace d'ArgoCD pendant
   toute la vie du bug.
2. Sous Flux, Velero a présenté la même signature : changer d'outil GitOps n'avait pas supprimé le
   problème.

## La preuve avant de revenir

Banc dédié, uprobes vérifiées avant chaque fenêtre de mesure : **334 `git fetch` instrumentés,
0 erreur**. Puis, en production, 0 occurrence sur plus de 40 000 lignes de logs, sur cinq démarrages
complets.

**Portée du résultat, sans la surestimer :** le banc n'avait ni la même version d'ArgoCD ni le même
noyau que l'incident. La cause OBI est très probable, mais ni prouvée ni écartée pour l'incident
historique.

## Décision

Retour sur ArgoCD. Ce n'est pas un *revert* : le dépôt a été réécrit, avec cette fois un ordonnancement
explicite du démarrage ([décision 006](006-app-of-apps-sync-waves.md)). Son absence expliquait aussi
une partie de la charge initiale.

Pourquoi revenir plutôt que rester sur Flux : **ce bug était la seule raison du départ**. L'argument
« Flux ne lance aucun sous-processus » n'avait jamais été un besoin, seulement une propriété constatée
après coup. Le motif tombé, la décision tombait aussi. Les briques ArgoCD déjà construites étaient
réutilisables : SSO, diff de rendu en CI (`argocd-diff-preview`), plugin Backstage.

## Compromis et coût

- ArgoCD n'a pas de `dependsOn` entre Applications, contrairement à Flux. Il a fallu des vagues et un
  contrôle de santé écrit en Lua.
- Quatre mois sur un outil choisi pour une mauvaise raison, et deux migrations complètes.

## Ce que j'en retiens

- **Un composant eBPF est un suspect par défaut** dans tout diagnostic « impossible » sur un pod qu'il
  instrumente. Il se limite par workload, jamais à un namespace entier.
- **Un suspect n'est pas un coupable.** Une attribution non mesurée coûte aussi cher que le bug.
- J'ai prévu un repli gradué si le motif revenait, et je me suis interdit de remettre les
  contournements d'avant.
