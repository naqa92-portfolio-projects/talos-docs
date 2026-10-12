# 008 · Gateway API servie par Cilium, sans contrôleur d'entrée séparé

> **En une phrase :** Cilium fait déjà le réseau et les politiques ; lui confier aussi l'entrée HTTP
> retire un composant à opérer, au prix d'un couplage assumé.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-09 | `talos-infra`, `talos-argocd` |

## Contexte

Il faut une entrée HTTPS pour les services publics. L'Ingress historique est remplacé par la Gateway API,
le standard actuel de Kubernetes. Sur le passage par Infomaniak, j'utilisais Envoy Gateway.

## Décision

- **Cilium** sert la Gateway API, en remplacement de kube-proxy, avec le load balancer Hetzner piloté
  par annotations.
- **Un listener HTTPS générique** sur le certificat *wildcard* : une nouvelle application est
  joignable sans modifier la Gateway.
- ArgoCD est exposé comme les autres services (terminaison TLS), plus en *passthrough*.

## Alternatives écartées

| Alternative | Pourquoi non |
|---|---|
| Envoy Gateway / ingress-nginx | Un composant de plus à opérer et à mettre à jour |
| TLS passthrough pour ArgoCD | Sous un certificat *wildcard*, HTTP/2 réutilise la connexion d'un autre hôte (*connection coalescing*, RFC 7540) : 404 intermittents. Cilium 1.20 permet de désactiver la traduction gRPC-web qui imposait le passthrough |
| Un certificat par hôte | Quota Let's Encrypt (5 par semaine), et un fichier à éditer à chaque nouvel hôte |

## Compromis et coût

**Couplage au CNI.** Un passage sur un Kubernetes managé, où le CNI n'est pas le mien, obligera à
réintroduire un contrôleur d'entrée. C'est précisément ce qui a bloqué sur Scaleway
([décision 004](004-retour-hetzner.md)).
