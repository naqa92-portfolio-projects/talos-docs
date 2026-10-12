# talos-backstage : le portail développeur

> **Un formulaire Backstage devient une pull request GitOps : un dépôt, une application ou une base de
> données en self-service, avec SSO et un serveur MCP pour piloter le portail depuis un assistant IA.**

| | |
|---|---|
| Rôle | Interface self-service au-dessus de l'API Crossplane |
| Outils | Backstage (nouveau système de plugins), TypeScript, Authentik OIDC, PostgreSQL, Kind |
| Visibilité | Privé |

## Ce que voit un développeur

- **Créer un dépôt** à partir d'un squelette Python/Flask ou Go, avec CI et Dockerfile prêts.
- **Créer une application ou une base de données.** Les formulaires sont générés automatiquement à partir
  des API Crossplane : ajouter un champ à l'API l'ajoute au formulaire.
- **Modifier ou supprimer** un composant, toujours par pull request.
- **Voir l'état réel** : synchronisation ArgoCD, pods, politiques Kyverno, ressources Crossplane,
  dashboards Grafana, métriques DORA.

## Ce que j'ai développé

- **Un module scaffolder** avec trois actions maison et un processeur qui nettoie les formulaires
  générés : il retire les réglages techniques et reporte les champs obligatoires de l'API.
- **Un plugin publié sur npm** qui synchronise les utilisateurs et groupes d'Authentik dans le catalogue
  ([open source](https://github.com/naqa92/authentik-backstage-plugin)).
- **Un environnement local fidèle** : un cluster Kind avec Cilium et Gateway API, qui rejoue la chaîne
  GitOps complète pour tester un formulaire de bout en bout.

## Choix notables

| Choix | Raison |
|---|---|
| Plugins déclarés explicitement | Le chargement automatique ne prend pas les plugins frontend dans la version utilisée |
| Plugin ArgoCD épinglé, exclu de Renovate | Les versions suivantes exigent un Backstage plus récent et dupliqueraient les paquets du framework. La condition de sortie est écrite |
| Image d'exécution non *distroless* | TechDocs génère la documentation dans le conteneur (Python + MkDocs) ; Alpine est incompatible avec un module natif |
| Scan de vulnérabilités bloquant avant publication | Seul contrôle de qualité exécuté en CI : le build d'image est le job qui compte |
| PostgreSQL sans sauvegarde en production | Tout l'état du portail se reconstruit depuis GitHub et la configuration d'Authentik |
| Lecture Kubernetes par rôle agrégé, pas de `*` | Une règle `*` donne aussi accès aux Secrets |
