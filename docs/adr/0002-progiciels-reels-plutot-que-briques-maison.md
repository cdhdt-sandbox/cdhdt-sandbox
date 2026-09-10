# ADR 0002, Pimcore et ERPNext plutôt que des briques écrites par nous

- Statut : accepté
- Date : 2026-09-10

## Contexte

Le cadrage initial prévoyait d'écrire nous-mêmes un faux ERP et un faux PIM en
Go. Deux arguments ont fait basculer la décision.

Le premier tient à ce que la démonstration vend. « J'ai intégré Sylius et
ERPNext » décrit le travail réel d'un intégrateur. « J'ai écrit un faux ERP »
décrit un exercice où l'on conçoit soi-même les deux côtés du contrat, ce qui
est nettement plus facile et moins convaincant.

Le second est que la mémoire n'est pas la contrainte que l'on croyait. Les
recommandations de dimensionnement des éditeurs, 4 Go pour ERPNext selon Frappe,
sont des dimensionnements pour un usage réel, pas des empreintes au repos.

## Décision

Utiliser Pimcore comme PIM et ERPNext comme ERP, tous deux en produits réels. Le
code Go se déplace vers les adaptateurs du modèle pivot et vers le faux
prestataire de paiement.

## Conséquences

- **Un engagement de suivi des CVE, indéfini, sur une machine publique.** C'est
  le coût principal, et il est permanent. Il est tenu par l'intégration continue
  décrite en section 10.3 de la spécification, dont l'exécution planifiée
  quotidienne, seule capable de signaler une CVE apparue sans commit.
- Le premier démarrage d'ERPNext prend plusieurs minutes. La remise à zéro passe
  donc par la restauration d'un instantané de volume, jamais par un rejeu
  d'installation.
- Le gabarit de la machine augmente. Il sera déduit d'une mesure en phase 1, pas
  d'une estimation.
- Ni Pimcore ni ERPNext ne parlent AMQP nativement. Leurs adaptateurs Go font le
  pont entre leur API et RabbitMQ.

## Politique de correctifs

Cet ADR porte l'engagement de suivi. Il en fixe donc les délais.

| Gravité | Délai d'application |
|---|---|
| Critique, ou exploitation active constatée | 24 heures |
| Élevée | 7 jours |
| Moyenne ou basse | au prochain lot |

Quand aucun correctif n'existe encore, l'interface concernée est **retirée de
l'exposition publique** jusqu'à publication du correctif. Elle n'est pas laissée
en ligne avec une mitigation partielle. Ce point est la contrepartie directe de
l'ADR 0007, qui expose publiquement les administrations.

## Options écartées

**Tout écrire en Go.** Le plus léger, la plus faible surface d'attaque, la
remise à zéro la plus rapide. Écarté parce que la valeur de vitrine est moindre.

**Un seul progiciel réel.** Position intermédiaire cohérente, écartée par
préférence explicite pour la version complète.
