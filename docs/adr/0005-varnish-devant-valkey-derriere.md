# ADR 0005, Varnish en cache de pages, Valkey en cache applicatif

- Statut : accepté
- Date : 2026-09-10

## Contexte

Le cadrage initial demandait « un cache Valkey devant la boutique ». Cette
formulation recouvrait deux architectures très différentes, dont les mécanismes
d'invalidation n'ont rien de commun.

Vérification faite, `FOSHttpCacheBundle` est pleinement fonctionnel avec
Varnish, tandis que Nginx et le `HttpCache` intégré de Symfony n'en supportent
qu'une partie. Un cache de pages stocké dans Valkey reste atteignable par un
store PSR-6 compatible étiquettes, mais par un chemin plus étroit.

## Décision

Varnish en cache de pages devant la boutique. Valkey en cache applicatif et en
stockage de sessions, derrière.

## Conséquences

- La documentation dira « Varnish devant, Valkey derrière », et non ce que
  disait le brief initial. La documentation décrit ce qui tourne.
- Un composant de plus à quota, à durcir et à superviser.
- L'interface de purge de Varnish ne doit être joignable que depuis l'intérieur
  du namespace.
- Le consommateur d'invalidation émet un appel HTTP interne vers Varnish. La
  NetworkPolicy l'autorise nommément, sans rouvrir de sortie vers Internet.
- L'invalidation par étiquette, via xkey, est plus riche que ce qu'offrirait le
  `HttpCache` intégré.

## Options écartées

**`HttpCache` de Symfony avec un store compatible étiquettes sur Valkey.** Tenait
l'énoncé initial au pied de la lettre, sans composant supplémentaire. Écarté par
préférence pour une configuration plus proche d'une production réelle.

**Pas de cache de pages du tout.** Le plus simple, mais on perdrait la
démonstration la plus parlante, celle du contenu qui ne se cache jamais.
