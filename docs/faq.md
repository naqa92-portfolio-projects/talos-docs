# Questions d'entretien

Les questions qu'on me pose, ou qu'on devrait me poser, avec la réponse courte. Le détail est dans
chaque page de décision.

## Vue d'ensemble

??? question "En une phrase, c'est quoi Taloslab ?"
    Une plateforme Kubernetes complète que j'opère seul sur Hetzner pour 41 €/mois. Un développeur
    remplit un formulaire Backstage, et son application est en ligne avec sa base, son DNS, son TLS, ses
    secrets et son monitoring, entièrement en GitOps.

??? question "Pourquoi construire tout ça pour un seul utilisateur ?"
    Pour pratiquer le métier de platform engineer sur un vrai système, pas sur un tutoriel. Chaque
    choix a un coût réel en euros, et chaque panne doit être réparée par moi. La contrainte
    mono-utilisateur est assumée : elle explique plusieurs choix, comme l'absence de haute disponibilité
    ou Kyverno en audit, et je sais dire ce qui changerait avec une équipe.

??? question "Qu'est-ce que tu referais différemment ?"
    Mesurer avant d'attribuer. J'ai changé d'outil GitOps à cause d'un bug que j'avais attribué à
    ArgoCD, alors qu'il venait de l'instrumentation eBPF. Et sur un Kubernetes managé, mesurer la
    latence etcd avant de migrer, pas après.

## Infrastructure

??? question "Pourquoi Talos plutôt qu'Ubuntu + kubeadm ?"
    Talos est immuable, sans SSH ni shell, piloté par API. Il n'y a rien à patcher à la main ni de
    dérive de configuration, et la surface d'attaque est minimale. Le cluster est recréé souvent : un OS
    déclaratif s'y prête mieux.

??? question "Pourquoi pas un Kubernetes managé ?"
    J'en ai essayé deux. Infomaniak avait des timeouts d'API récurrents ; Scaleway, 0,73 s par requête
    etcd sur son control plane mutualisé. Je suis revenu sur Hetzner, où je contrôle le réseau, Cilium
    avec Gateway API compris. → [004](decisions/004-retour-hetzner.md)

??? question "Un seul control plane, ce n'est pas risqué ?"
    Si : etcd n'a aucune tolérance de panne. Mais trois control planes triplent le poste le plus cher,
    pour un service sans utilisateur tiers. En échange : snapshot etcd quotidien hors Hetzner et
    reconstruction complète par Terraform + GitOps. → [002](decisions/002-control-plane-unique.md)

??? question "Comment as-tu choisi la taille des machines ?"
    Par la mesure. Sur 2 vCPU, le control plane montait à 132 % de CPU pendant la synchronisation
    initiale, sans aucune alerte mémoire : les contrôleurs redémarraient en chaîne. Passé à 4 vCPU, il
    tombe à 28 %, pour 3,60 € de plus par mois.

## GitOps

??? question "Pourquoi ArgoCD ? Tu étais passé sur Flux."
    Je suis passé sur Flux pour fuir un bug. Quand la même signature est apparue sur d'autres composants,
    prouvée comme venant de l'instrumentation eBPF, la raison du départ est tombée. J'ai refait une
    preuve A/B (334 `git fetch`, 0 erreur) avant de revenir. → [005](decisions/005-argocd-flux-argocd.md)

??? question "Comment gères-tu l'ordre de déploiement sans `dependsOn` ?"
    Des vagues : la vague suivante attend que toute la précédente soit synchronisée et en bonne santé.
    ArgoCD désactive par défaut la santé des Applications enfants, donc je la réactive par un script
    Lua, sans quoi les vagues n'attendent rien. → [006](decisions/006-app-of-apps-sync-waves.md)

??? question "Raconte-moi un incident."
    Pendant une migration du dépôt GitOps, ArgoCD a fait tourner l'ancienne définition d'une Application
    sur les nouveaux fichiers et a supprimé une base PostgreSQL. Je l'ai restaurée sans perte grâce aux
    journaux archivés sur B2. Une politique Kyverno a arrêté la cascade. Leçon : séparer en deux commits
    le changement de définition et le changement de contenu. → [Incident](incidents.md#prune-saveit)

??? question "Pourquoi pas le Rendered Manifests Pattern ?"
    L'implémentation native d'ArgoCD ne sait pas exprimer 23 de mes 37 Applications. Rendu en CI, le
    résultat diverge du cluster pour 17 charts sur 22. Un diff de rendu en commentaire de chaque PR me
    donne la même revue sans artefact à maintenir. → [012](decisions/012-pas-de-rendered-manifests.md)

## Plateforme

??? question "Pourquoi Crossplane plutôt qu'un chart Helm commun ?"
    Crossplane réconcilie en continu et remonte un statut lisible, et le développeur ne voit qu'une API
    de 15 lignes au lieu de toutes les valeurs d'un chart. La logique est en Python, testée de bout en
    bout sur Kind en CI. → [007](decisions/007-crossplane-api-self-service.md)

??? question "Où sont les secrets ?"
    Dans Infisical. External Secrets les synchronise dans le cluster, et Reloader redémarre les pods à la
    rotation. Il n'y a rien dans Git, et rien dans l'état Terraform grâce aux attributs *write-only*.
    → [009](decisions/009-secrets-infisical-eso.md)

??? question "Kyverno en audit, ce n'est pas un peu léger ?"
    Pour les bonnes pratiques, oui, et c'est voulu : une seule personne déploie. La seule règle
    bloquante protège ce qui ne se reconstruit pas, les données, et elle a déjà servi en incident. Avec
    une équipe, je passerais en blocage règle par règle. → [010](decisions/010-kyverno-audit-garde-fous.md)

??? question "Comment l'observabilité est-elle organisée ?"
    Stack Victoria (métriques, logs, traces) avec Grafana et des alertes Slack, plus Gatus pour la
    disponibilité vue de l'extérieur. L'application est instrumentée par le SDK OpenTelemetry, après le
    retrait d'une instrumentation eBPF qui corrompait des processus. → [011](decisions/011-retrait-obi-sdk-otel.md)

## Agent SRE

??? question "Tu laisses un LLM modifier ta production ?"
    Non. En production, l'agent enquête en lecture et propose. Un humain décide, et une ressource gérée
    en GitOps n'est corrigée que par pull request. Le prochain chantier, avant tout mode automatique, est
    de resserrer ses droits au strict nécessaire. → [sre-agent](projets/sre-agent.md)

??? question "Pourquoi ne pas utiliser kagent ou une passerelle LLM ?"
    Une plateforme partagée ne se rentabilise qu'avec plusieurs consommateurs, et il n'y en a qu'un.
    kagent remplacerait du code déterministe et testé par du prompt. Je le réévaluerais derrière un test
    de fiabilité, une fois plusieurs agents en place.
