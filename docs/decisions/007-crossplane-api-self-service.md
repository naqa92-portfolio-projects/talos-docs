# 007 · Une API maison `App` / `Database` avec Crossplane

> **En une phrase :** un développeur décrit son application en 15 lignes ; la plateforme génère les
> ressources Kubernetes qui vont avec et applique les bonnes pratiques à sa place.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-03 | `talos-argocd` |

## Contexte

Déployer correctement une application sur cette plateforme demande une dizaine d'objets : Deployment,
Service, route HTTPS, redirection HTTP, politique réseau, secrets d'accès au registre, monitoring. Les
écrire à la main pour chaque application, c'est recopier des erreurs.

## Décision

Deux types maison exposés par Crossplane v2 :

```yaml
apiVersion: taloslab.cc/v1alpha1
kind: App
metadata:
  name: saveit
spec:
  image: ghcr.io/…/saveit:0.2.11
  hostname: saveit.taloslab.cc
  size: medium          # small | medium | large → requests / limits
  migrate: true         # migrations de base avant le démarrage (initContainer)
  workerCommand: "…"    # second Deployment pour les tâches de fond
  metrics: true         # ServiceMonitor + dashboard
```

| Le développeur écrit | La plateforme génère |
|---|---|
| `kind: App` | ServiceAccount, Deployment (+ worker), Service, 2 HTTPRoutes (HTTPS + redirection), CiliumNetworkPolicy, ExternalSecrets, ConfigMap, ServiceMonitor |
| `kind: Database` | Cluster PostgreSQL CloudNativePG, PodMonitor, sauvegarde planifiée et archivage continu vers B2 |

- La logique des compositions est écrite **en Python** (`function-pythonic`) : lisible, testable, rendue
  hors cluster par `crossplane render`.
- **Tests de bout en bout Chainsaw** sur un cluster Kind éphémère en CI, plus une vérification que le
  code généré est à jour.
- Le statut de chaque ressource remonte dans Backstage avec des conditions lisibles.

## Alternatives écartées

| Alternative | Pourquoi non |
|---|---|
| Un chart Helm commun | Pas de statut ni de réconciliation continue ; chaque équipe peut surcharger n'importe quoi |
| Compositions en YAML pur (patch & transform) | Illisible dès qu'il y a des conditions |
| Opérations Crossplane (alpha), Usage / protection de suppression | Excessif pour deux applications ; une politique Kyverno fait le même travail plus simplement |
| RBAC multi-équipes Crossplane | Pensé pour plusieurs équipes, sans objet pour une personne |

## Compromis et coût

Une couche de plus à maintenir. Les valeurs par défaut vivent dans la définition de l'API (XRD) et ne
s'appliquent qu'à l'admission, donc les exemples de test doivent les écrire explicitement. Faire
évoluer le type d'un champ demande une procédure de migration.
