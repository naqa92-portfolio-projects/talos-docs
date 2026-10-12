# 009 · Secrets dans Infisical, synchronisés par External Secrets

> **En une phrase :** aucun secret dans Git ni dans l'état Terraform ; le cluster va chercher ses
> secrets dans un coffre externe et redémarre les pods quand ils changent.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu | 2026-02 | `talos-infra`, `talos-argocd` |

## Décision

- **Infisical** (SaaS, région UE) est le coffre. **External Secrets Operator** crée les secrets
  Kubernetes à partir du coffre. **Reloader** redémarre les pods quand un secret change.
- Terraform ne dépose qu'**un seul secret d'amorçage** (l'identité machine Infisical), via des attributs
  *write-only* : la valeur ne se retrouve jamais dans l'état Terraform.
- Les compositions Crossplane créent elles-mêmes les ExternalSecrets d'une application.

## Alternatives écartées

| Alternative | Pourquoi non |
|---|---|
| SOPS / Sealed Secrets | Des secrets chiffrés dans Git restent dans l'historique, et la rotation de clé est manuelle |
| Infisical auto-hébergé | Un service critique de plus à opérer et à sauvegarder ; le palier gratuit suffit |
| Vault / OpenBao | Surdimensionné pour une personne |

## Compromis et coût

Le démarrage du cluster dépend d'un SaaS externe : si Infisical est indisponible, les nouveaux secrets
ne sont pas créés (ceux déjà présents restent en place).
