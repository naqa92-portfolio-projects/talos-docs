# Retours d'expérience

Les pannes sont la partie la plus instructive du projet. Chacune suit le même format : **symptôme →
cause → correctif → leçon**.

## Une migration GitOps qui a déclenché une suppression, restaurée sans perte {#prune-saveit}

**Contexte.** Pendant le retrait de Kustomize du dépôt GitOps, ArgoCD a supprimé (*prune*) les
ressources de deux applications, dont leur base PostgreSQL.

**Cause.** Entre le merge d'un commit et l'application de la nouvelle définition d'une Application,
ArgoCD fait tourner l'**ancienne** définition sur les **nouveaux** fichiers. L'ancienne ne savait lire
le dossier qu'à travers son fichier Kustomize, qui venait d'être supprimé. ArgoCD a lu le dossier en
mode non récursif, n'y a trouvé presque rien, et a retiré le reste. Le `prune: false` ajouté dans le
même commit ne pouvait pas protéger : il ne prenait effet qu'avec la nouvelle définition.

**Ce qui a fonctionné.**

- **Garde-fou** : la politique Kyverno qui protège les données a refusé la suppression des ressources
  Crossplane et arrêté la cascade.
- **Restauration** : restauration à un instant donné (PITR) depuis les journaux PostgreSQL archivés en
  continu sur B2, par la procédure scriptée déjà utilisée lors des reconstructions de cluster.
  **Aucune perte de données** : la chaîne d'archives couvrait jusqu'à six secondes après le début de
  la suppression.

**Ce que j'ai changé ensuite.**

1. Un changement qui touche à la fois une définition et le contenu qu'elle lit se fait en **deux
   commits** : d'abord la définition, vérifiée sur le cluster, ensuite le contenu.
2. Un contrôle CI vérifie désormais que chaque Application déclare explicitement son mode de lecture,
   et interdit le retour de fichiers Kustomize.
3. Une migration se juge sur la date de création des pods, pas seulement sur `Healthy` / `Synced`.

**Ce que ça montre.** Une protection écrite dans le commit qui crée le risque ne couvre pas ce commit,
et un diff de rendu en CI ne voit pas la fenêtre de transition, puisqu'il compare des états finaux.
La défense en profondeur (garde-fou d'admission + sauvegarde continue + restauration répétée) a fait
son travail.

## Un composant « Healthy » rendu avec ses valeurs par défaut {#fenetre-migration}

**Symptôme.** Pendant l'évaluation d'un autre pattern, la sonde de disponibilité Gatus a été recréée
sans sa configuration, sans ses volumes et avec le compte de service par défaut, alors qu'ArgoCD
l'affichait `Healthy` et `Synced` du début à la fin.

**Cause.** La même fenêtre de transition : l'ancienne définition lisait un fichier de valeurs restructuré
et n'y reconnaissait plus rien. **Un rendu avec les valeurs par défaut est un rendu valide**, donc
aucune erreur.

**Correctif.** Annulé en environ une minute.

**Leçon.** Juger une migration sur la date de création des pods, jamais sur `Healthy` / `Synced` seuls.

## Le control plane saturé sans aucune alerte mémoire

**Symptôme.** Au démarrage du cluster, le Gateway devenait injoignable, et le controller-manager, le
scheduler et les contrôleurs GitOps redémarraient en chaîne.

**Cause.** Sur un serveur à 2 vCPU, la cascade GitOps poussait le CPU du control plane à **132 %**.
Les sondes de santé expiraient. Aucun signal mémoire classique ne s'est déclenché.

**Correctif.** Passage à 4 vCPU (`cx33`) : 28 % de CPU, plus aucun redémarrage, +3,60 €/mois.

**Leçon.** Dimensionner sur le pic de démarrage, et surveiller le CPU du control plane avant la
mémoire. → [Décision 002](decisions/002-control-plane-unique.md)

## Un bug « impossible », traqué jusqu'à sa vraie cause

**Symptôme.** Des sous-processus lisaient des octets nuls à la place de leur sortie : ArgoCD (`git
fetch`), les sauvegardes PostgreSQL, Velero.

**Cause.** L'instrumentation eBPF OBI, qui pose des sondes dans les processus. Je l'ai prouvé par A/B
sur PostgreSQL : exclure le pod de l'instrumentation suffisait à rétablir les sauvegardes. Le symptôme
pointait d'abord vers ArgoCD, le noyau Talos ou le plugin CloudNativePG.

**Correctif.** Retrait d'OBI, instrumentation par le SDK OpenTelemetry.

**Leçon.** Un composant qui modifie d'autres processus est suspect par défaut, et chaque suspicion se
teste en A/B avant d'agir. → [Décisions 005](decisions/005-argocd-flux-argocd.md) et
[011](decisions/011-retrait-obi-sdk-otel.md)

## Le control plane managé trop lent

**Symptôme.** Sur Scaleway Kapsule, le GitOps convergeait mal sur un cluster de seulement 56 pods.

**Cause.** 0,73 s par requête etcd sur le control plane mutualisé gratuit. Cette limite n'apparaît
nulle part dans la documentation.

**Leçon.** Mesurer `etcd_request_duration_seconds` avant de migrer, pas après.
→ [Décision 004](decisions/004-retour-hetzner.md)

## ArgoCD s'est coupé lui-même de l'API {#argocd-coupe-api}

**Symptôme.** Au démarrage, ArgoCD n'appliquait plus rien, et tout ce qui parle à l'API Kubernetes
tombait avec lui.

**Cause.** À rang égal, ArgoCD applique les ressources par ordre alphabétique. Une politique
d'autorisation réseau passait avant la politique *default-deny* qui contenait l'accès à l'API. ArgoCD
perdait l'API avant d'avoir pu appliquer la règle qui la lui rendait.

**Correctif.** La politique *default-deny* est appliquée dans la toute première vague.

**Leçon.** Flux appliquait tout en une passe et masquait le défaut. Un changement d'outil révèle des
dépendances implicites.

## 404 intermittents sur ArgoCD

**Cause.** Sous un certificat *wildcard*, HTTP/2 réutilise une connexion ouverte pour un autre hôte
(*connection coalescing*). Les requêtes arrivaient sur une chaîne TLS qui n'avait pas de route vers
ArgoCD.

**Correctif.** ArgoCD est repassé en terminaison TLS standard, ce qu'une option récente de Cilium
rendait possible.

## Les sauvegardes Velero refusées par Backblaze B2

**Cause.** Le plugin AWS de Velero envoie toujours un en-tête de *tagging* que B2 refuse. L'en-tête est
signé : impossible de le retirer en aval sans casser la signature.

**Correctif.** Un proxy de signature qui retire l'en-tête puis re-signe la requête. La correction amont
est mergée mais pas encore publiée. La condition de sortie est écrite : retirer le proxy dès qu'une
release la contient.

**Leçon.** Chaque contournement est consigné dans un registre avec sa cause amont, son lien et sa
condition de sortie. Sans condition de sortie, un contournement n'est pas documenté, il est enterré.

## Autres contournements tenus à jour

| Composant | Problème | Contournement |
|---|---|---|
| Authentik | La mémoire partagée par défaut (64 Mi) sature, d'où des 502 | Volume mémoire de 256 Mi monté sur `/dev/shm` |
| Backstage | La découverte OIDC n'est tentée qu'au démarrage, sans retry | Un conteneur d'init attend qu'Authentik réponde |
| n8n | Le SSO OIDC est réservé à l'édition Enterprise | Routes OIDC ajoutées par les *hooks* externes supportés |
| Gatus | La configuration écrite par le sidecar n'est lue qu'au second démarrage | Redémarrage au boot conservé, correction amont suivie |
| Let's Encrypt | Quota de 5 certificats par semaine, épuisé par les reconstructions | Certificats sauvegardés avant destruction, réinjectés à la reconstruction |
