# ADR 0003, modèle pivot et adaptateurs

- Statut : accepté
- Date : 2026-09-10

## Contexte

Trois systèmes doivent échanger : Pimcore, Sylius et ERPNext. Trois câblages
étaient possibles. Des connecteurs point à point, un modèle pivot avec des
adaptateurs, ou un service d'intégration unique.

## Décision

Définir nos propres schémas d'événements, versionnés dans le dépôt, et donner à
chaque système un adaptateur Go qui traduit dans les deux sens. Aucun système ne
connaît le format d'un autre.

## Conséquences

- La conception initiale est plus longue qu'avec des connecteurs point à point.
- Les tests d'idempotence portent sur le pivot, une fois, au lieu d'être
  réécrits pour chaque connecteur.
- Le versionnement de schéma devient une contrainte explicite : ascendant
  uniquement à l'intérieur d'une version majeure.
- Ajouter un quatrième système coûte un adaptateur, pas une refonte des
  mappings existants.

## Options écartées

**Connecteurs point à point.** Plus rapides à livrer et suffisants pour trois
systèmes. Écartés parce qu'un changement du modèle produit toucherait plusieurs
connecteurs, avec des mappings dupliqués que rien ne garde cohérents.

**Service d'intégration unique.** Le moins de déploiements, un seul endroit à
lire. Écarté parce qu'il devient un point de défaillance central, qu'il grossit
vite, et qu'il se prête mal aux tests en isolation posés comme règle du projet.
