# Limites assumées et prochaines étapes

Ce que la plateforme ne fait pas, ou pas encore. La plupart sont des choix chiffrés ; le reste forme la
feuille de route.

## Choix assumés

| Limite | Pourquoi c'est acceptable ici | Ce qui la ferait lever |
|---|---|---|
| Aucune haute disponibilité, un seul membre etcd | Service mono-utilisateur ; reconstruction et sauvegardes testées ([003](decisions/003-pas-de-haute-disponibilite.md)) | Des utilisateurs tiers ou un engagement de disponibilité |
| Bonnes pratiques Kyverno en audit, pas en blocage | Une seule personne déploie ([010](decisions/010-kyverno-audit-garde-fous.md)) | Une deuxième équipe sur le cluster |
| Pas d'environnement de staging permanent | Le coût 24 h/24 d'un second environnement ; un cluster Kind local rejoue la chaîne GitOps complète pour tester Backstage | Plusieurs contributeurs, ou des mises en production à risque |
| Dépendance à des SaaS (Infisical, Cloudflare, Backblaze B2, GitHub) | Rien à opérer, des paliers gratuits suffisants | Une exigence de souveraineté ou d'isolation réseau |
| CI Terraform en *plan* seul, déclenchée à la main | L'API d'administration est filtrée sur l'IP de l'administrateur : un runner CI y apparaîtrait toujours en dérive | Une liste d'IP statiques autorisées |
| Les règles d'alerte par défaut suivent leur dépôt amont | Les figer reviendrait à recopier la liste du chart, qui dériverait à son tour | Un seuil d'alerte modifié en amont qui pose problème |

## Feuille de route {#chantiers-ouverts}

- **Agent SRE (en cours de développement).** Il tourne en mode manuel : il enquête, notifie et propose
  des pull requests, un humain décide. Avant d'envisager un mode automatique : durcissement du moindre
  privilège outil par outil, et budget global de consommation LLM.
- **Lint en CI pour Backstage.** Lancé en local avant chaque push ; le job CI est prêt, à activer dès
  l'arrivée d'un second contributeur.
- **Passage du plugin de sauvegarde PostgreSQL sur son chart Helm officiel**, désormais disponible.

## Ce que je ferais avec une équipe

- Passer les politiques Kyverno en blocage, règle par règle, avec une période d'audit préalable.
- Ajouter un environnement de préproduction et une promotion explicite entre environnements.
- Mettre l'accès à l'agent SRE derrière une identité par utilisateur (OAuth) plutôt qu'un jeton partagé.
- Fédérer les appels LLM derrière une passerelle (budgets, bascule entre fournisseurs) dès qu'un
  deuxième consommateur existe.
