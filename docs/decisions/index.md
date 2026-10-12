# Décisions

Chaque page suit le même plan : **contexte → décision → alternatives écartées → compromis et coût →
quand la revoir**. Les compromis sont écrits noir sur blanc : un choix sans coût n'est pas un choix.

| # | Décision | Le compromis accepté |
|---|---|---|
| [001](001-talos-hetzner-terraform.md) | Talos Linux sur Hetzner, provisionné par un module Terraform maintenu | Dépendre des choix par défaut du module, en partie inutilisables tels quels |
| [002](002-control-plane-unique.md) | Un seul control plane, `cx33` minimum | Aucune tolérance de panne sur etcd : perdre le nœud impose de reconstruire |
| [003](003-pas-de-haute-disponibilite.md) | Pas de haute disponibilité : auto-réparation plutôt que continuité | Chaque composant est un point unique de défaillance, assumé |
| [004](004-retour-hetzner.md) | Hetzner → Infomaniak → Scaleway → Hetzner | Trois bascules d'hébergeur, absorbées par l'IaC et le GitOps |
| [005](005-argocd-flux-argocd.md) | ArgoCD → Flux → ArgoCD | Flux a la meilleure architecture, ArgoCD domine le marché : j'ai choisi le marché |
| [006](006-app-of-apps-sync-waves.md) | App-of-apps découverte par dossier, ordonnée par vagues | Un démarrage à froid sérialisé, plus lent |
| [007](007-crossplane-api-self-service.md) | Une API maison `App` / `Database` avec Crossplane | Une couche d'abstraction de plus à maintenir et à tester |
| [008](008-gateway-api-cilium.md) | Gateway API servie par Cilium, sans contrôleur d'entrée séparé | Couplage au CNI, à repayer en cas de passage sur un Kubernetes managé |
| [009](009-secrets-infisical-eso.md) | Secrets dans Infisical, synchronisés par External Secrets | Dépendance à un SaaS pour démarrer le cluster |
| [010](010-kyverno-audit-garde-fous.md) | Kyverno : audit pour les bonnes pratiques, blocage pour les données | Les violations de bonnes pratiques sont signalées, pas empêchées |
| [011](011-retrait-obi-sdk-otel.md) | Retirer l'instrumentation eBPF au profit du SDK OpenTelemetry | Modifier le code de l'application pour la tracer |
| [012](012-pas-de-rendered-manifests.md) | Pas de *Rendered Manifests Pattern*, un diff de rendu en CI à la place | Pas d'historique Git des manifestes rendus |

## Ce qui a été écarté sans être adopté

Liste courte des outils évalués puis refusés, avec la raison en une ligne :

| Outil | Pourquoi non |
|---|---|
| Terragrunt | Un seul cluster, aucun besoin multi-environnement |
| Kargo, promotion par dossiers d'environnement | Pas d'environnement persistant à promouvoir : un staging permanent coûterait 24 h/24 pour une seule personne |
| Environnements de preview par PR | Plan complet chiffré (~+15 €/mois, 17 fichiers sur 3 dépôts), écarté : le GitOps ajoute de la latence là où l'éphémère doit être jetable |
| vCluster | Crossplane est global au cluster : il faudrait tout recréer dans chaque vCluster |
| ARM (`cax31`) | Plus cher par Go de RAM (1,57 € contre 1,27 €) et images applicatives amd64 uniquement |
| Chiffrement des sauvegardes (AGE) | Bucket déjà privé ; la gestion des clés coûte plus que le risque couvert |
| Infisical auto-hébergé | Le palier gratuit couvre le besoin sans rien à opérer |
| kagent, LiteLLM, agentgateway | Une passerelle partagée ne se rentabilise qu'à partir de deux consommateurs ; il n'y en a qu'un |
| Lecture seule « tout sauf les Secrets » pour Backstage | Impossible en RBAC Kubernetes : une règle `*` inclut les Secrets, et aucun webhook ne filtre les lectures |
