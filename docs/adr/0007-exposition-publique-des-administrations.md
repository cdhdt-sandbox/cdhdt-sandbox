# ADR 0007, exposition publique de toutes les interfaces

- Statut : accepté
- Date : 2026-09-10

## Contexte

Deux modèles d'exposition étaient possibles. Un DNS public minimal, où seules la
boutique, Grafana, Mailpit et le faux prestataire de paiement existent sur
Internet, les administrations n'étant joignables que par tunnel privé. Ou une
exposition complète.

L'auteur du projet a choisi l'exposition complète, contre l'avis rendu pendant
le cadrage.

## Décision

Toutes les interfaces sont publiquement accessibles, y compris les
administrations de Pimcore, d'ERPNext et de RabbitMQ.

## Conséquences

- Trois interfaces d'administration supplémentaires sont offertes aux scanners,
  avec leurs flux de CVE. Cela renforce la criticité de l'engagement pris dans
  l'ADR 0002.
- Aucun identifiant en écriture n'est publié, quelle que soit l'interface. Le
  public reçoit uniquement un compte Sylius en lecture seule.
- La limitation de débit à l'entrée protège autant la ressource processeur que
  la sécurité.
- Aucune conception ne doit reposer sur la discrétion d'un nom d'hôte. Tout
  certificat émis par une autorité publique est publié dans les journaux de
  Certificate Transparency.
- Un pare-feu applicatif filtre du HTTP. Ce n'est pas une frontière. Les
  frontières réelles restent la NetworkPolicy, les quotas, l'absence de compte
  en écriture public et l'isolement réseau de l'infrastructure.

## Options écartées

**DNS public minimal.** Réduisait la surface d'un cran sans rien coûter en
valeur de démonstration, les administrations n'étant de toute façon pas
utilisables par un visiteur. Écarté par préférence de l'auteur du projet.
