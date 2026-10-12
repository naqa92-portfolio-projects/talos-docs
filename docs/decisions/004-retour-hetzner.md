# 004 · Hetzner → Infomaniak → Scaleway → Hetzner

> **En une phrase :** j'ai quitté Hetzner pour un Kubernetes managé afin de réduire le coût et d'éliminer
> un bug, puis j'y suis revenu parce que les offres managées gratuites ont cédé sur la fiabilité de leur
> control plane.

| Statut | Date | Dépôt |
|---|---|---|
| Retenu : Hetzner, control plane et workers séparés | 2026-02 → 2026-09-12 | `talos-infra` |

## Chronologie

| Quand | Où | Pourquoi on part / on arrive |
|---|---|---|
| 2026-02 | **Hetzner** + Talos, 3 control planes qui portent aussi les applications | Point de départ |
| 2026-07 | **Infomaniak KaaS** (managé) | Le control plane managé est gratuit, donc de vrais workers pour le même budget ; et un bug de pipe corrompu semblait venir de Talos (~33-38 €/mois chez Hetzner à l'époque) |
| 2026-09 | **Scaleway Kapsule** (managé), tentative | L'API Infomaniak tombait très souvent en timeout, alors que le cluster est recréé fréquemment |
| 2026-09-12 | **Retour Hetzner**, 1 control plane + 2 workers | Scaleway disqualifié en un jour : **0,73 s par requête etcd** sur le control plane mutualisé, avec seulement 56 pods. Le type `cx33` était de nouveau disponible chez Hetzner |

## Décision

Retour au « fait maison » sur Hetzner, cette fois avec un control plane dédié et deux workers. Je garde
la maîtrise de l'OS et du réseau (Cilium avec Gateway API, qu'un managé ne laissait pas configurer
entièrement).

## Ce que ces migrations m'ont appris

- **Sur un control plane managé, mesurer la latence etcd avant de migrer le GitOps.** Les limites
  annoncées (RAM, nombre de nœuds) n'ont jamais été atteintes : c'est la latence, non documentée, qui a
  cédé.
- **La disponibilité des types de serveur change sans préavis.** Chez Hetzner, `cx43`/`cx53` ont disparu
  et `cx33` est passé par trois états en une heure. La disponibilité se vérifie à chaque `apply`.
- **Le « bug Talos » n'en était pas un.** Il venait de l'instrumentation eBPF (voir
  [décision 011](011-retrait-obi-sdk-otel.md)). Une attribution non mesurée a pesé dans une migration.

## Alternatives écartées

| Alternative | Pourquoi non |
|---|---|
| Rester sur Infomaniak | Timeouts API récurrents, incompatibles avec un cluster recréé souvent |
| Scaleway Kapsule | Latence etcd du control plane mutualisé ; Gateway API de Cilium non activable (configuration de l'opérateur hors d'atteinte) |
| ARM chez Hetzner (`cax31`) | Plus cher par Go de RAM, et applications publiées en amd64 seulement |

## Compromis et coût

Trois migrations en deux mois. Chacune a été menée comme un projet, avec un PRD, des jalons et une
mesure de fin, et le GitOps a permis de reconstruire la plateforme à chaque fois. Coût actuel :
**41,36 €/mois**.
