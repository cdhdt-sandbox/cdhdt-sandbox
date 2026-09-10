# ADR 0001, version de Sylius et politique de montée

- Statut : accepté
- Date : 2026-09-10

## Contexte

Au 2026-09-10, l'état des versions Sylius est le suivant. La 2.2 est la dernière
stable, publiée le 2025-12-17, dernier correctif 2.2.9 le 2026-09-02, maintenue
jusqu'en novembre 2026 et supportée jusqu'en mars 2027. La 2.3.0-ALPHA.1 date du
2026-08-10. La 2.1 sort de support en septembre 2026. La 1.14 LTS ne reçoit plus
que des correctifs de sécurité, jusqu'en décembre 2026.

La démonstration prétend prouver qu'un système exposé publiquement n'est pas
exploitable. Faire tourner une boutique publique sur une base dont le support
s'arrête dans trois mois contredirait directement cet objectif.

## Décision

Épingler Sylius 2.2.x. Monter vers la 2.3 dès sa publication stable. Ne jamais
déployer une version alpha ou bêta.

## Conséquences

- Un suivi de dépendances en intégration continue est nécessaire dès le premier
  lot, il n'est pas reporté à la phase de durcissement.
- La montée vers 2.3 devra faire l'objet de son propre lot, avec sa pull request
  et son build vert, comme n'importe quel changement.
- Une échéance existe : la maintenance de la 2.2 s'arrête en novembre 2026. La
  montée ne peut pas être repoussée indéfiniment.

## Options écartées

**1.14 LTS.** Défendable seulement pour démontrer une compétence sur la ligne
1.x. Le support s'arrête en décembre 2026, ce qui obligerait soit à éteindre la
démonstration, soit à migrer, et imposerait de l'annoncer sur la page d'accueil.

**Suivre la dernière version disponible, alpha comprise.** Incompatible avec une
démonstration laissée en fonctionnement sans surveillance permanente.
