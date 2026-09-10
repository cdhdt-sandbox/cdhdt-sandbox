# ADR 0008, checkout invité avec adresse électronique verrouillée

- Statut : accepté
- Date : 2026-09-10

## Contexte

Le critère de sortie de la phase 1 exige qu'une commande passée dans la boutique
arrive dans l'ERP. Passer une commande, c'est écrire. Or une règle du projet dit
qu'aucun compte en écriture n'est public.

La contradiction n'est qu'apparente : la règle vise l'administration, pas le
tunnel de commande. Restait un problème réel. Une adresse électronique saisie
par un visiteur inconnu deviendrait publiquement lisible dans Mailpit, ce qui
serait une fuite de donnée personnelle causée par la démonstration elle-même.

## Décision

Ouvrir le tunnel de commande aux visiteurs anonymes, et préremplir le champ
d'adresse électronique avec une adresse générée par session, verrouillée, sur un
domaine de premier niveau `.invalid`.

## Conséquences

- Le domaine `.invalid` est réservé par la RFC 2606 et n'est jamais routable.
  Même si un envoi sortant échappait à la NetworkPolicy, il ne pourrait
  atteindre personne.
- Aucune donnée personnelle de tiers n'entre dans le système. Mailpit peut donc
  rester publiquement consultable, sans masquage ni purge agressive.
- La surcharge du formulaire Sylius est un développement à part entière, avec
  ses tests. Elle constitue un lot.
- La démonstration est légèrement moins réaliste qu'un formulaire libre. C'est
  le prix, et il est faible.

## Options écartées

**Adresse libre avec masquage dans Mailpit.** Plus réaliste. Écarté parce que la
surface de fuite dépendrait alors de la correction du masquage, et qu'une
mention de confidentialité serait nécessaire.

**Vitrine en lecture seule avec commandes injectées.** Surface minimale, mais on
ne démontrerait plus Sylius en conditions réelles, et le faux prestataire de
paiement perdrait son objet.
