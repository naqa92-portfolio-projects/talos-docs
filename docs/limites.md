# Limites connues

Ce que la plateforme ne fait pas, ou pas encore. Certaines limites sont des choix chiffrés, d'autres
sont des chantiers ouverts.

## Choix assumés

| Limite | Pourquoi c'est acceptable ici | Ce qui la ferait lever |
|---|---|---|
| Aucune haute disponibilité, un seul membre etcd | Service mono-utilisateur ; reconstruction et sauvegardes testées ([003](decisions/003-pas-de-haute-disponibilite.md)) | Des utilisateurs tiers ou un engagement de disponibilité |
| Bonnes pratiques Kyverno en audit, pas en blocage | Une seule personne déploie ([010](decisions/010-kyverno-audit-garde-fous.md)) | Une deuxième équipe sur le cluster |
| Pas d'environnement de staging permanent | Le coût 24 h/24 d'un second environnement ; un cluster Kind local rejoue la chaîne GitOps complète pour tester Backstage | Plusieurs contributeurs, ou des mises en production à risque |
| Dépendance à des SaaS (Infisical, Cloudflare, Backblaze B2, GitHub) | Rien à opérer, des paliers gratuits suffisants | Une exigence de souveraineté ou d'isolation réseau |
| CI Terraform en *plan* seul, déclenchée à la main | L'API d'administration est filtrée sur l'IP de l'administrateur : un runner CI y apparaîtrait toujours en dérive | Une liste d'IP statiques autorisées |
| Les règles d'alerte par défaut suivent leur dépôt amont | Les figer reviendrait à recopier la liste du chart, qui dériverait à son tour | Un seuil d'alerte modifié en amont qui pose problème |

## Chantiers ouverts {#chantiers-ouverts}

- **Agent SRE : moindre privilège.** En production, l'agent ne fait que notifier et proposer des pull
  requests (mode manuel). Ses droits dans le cluster restent plus larges que ce mode n'exige, et il
  n'existe pas encore de budget global de consommation LLM. Un audit complet a listé les corrections à
  faire. C'est la prochaine étape avant tout mode automatique.
- **Lint en CI pour Backstage.** Il est lancé en local avant chaque push. Le job CI existe en commentaire,
  à réactiver dès qu'il y aura un second contributeur.
- **Tests applicatifs de la page vitrine.** Tests de routes seulement ; la lecture de l'API Kubernetes
  n'est pas couverte.
- **Un plugin de sauvegarde PostgreSQL encore installé par manifeste brut.** Le chart Helm officiel
  existe désormais : la migration est prête à faire.

## Ce que je ferais avec une équipe

- Passer les politiques Kyverno en blocage, règle par règle, avec une période d'audit préalable.
- Ajouter un environnement de préproduction et une promotion explicite entre environnements.
- Mettre l'accès à l'agent SRE derrière une identité par utilisateur (OAuth) plutôt qu'un jeton partagé.
- Fédérer les appels LLM derrière une passerelle (budgets, bascule entre fournisseurs) dès qu'un
  deuxième consommateur existe.
