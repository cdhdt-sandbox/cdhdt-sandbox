# ADR 0004, RabbitMQ et outbox transactionnel

- Statut : accepté
- Date : 2026-09-10

## Contexte

Le choix d'un bus de messages est souvent présenté comme un choix de broker. Le
coût réel se situait ailleurs : dans l'existence ou non d'un transport de
première main du côté PHP.

Symfony Messenger fournit nativement les transports `amqp://`, `doctrine://` et
`redis://`, une stratégie de reprise paramétrable et un `failure_transport`.

Par ailleurs, publier un événement de commande après avoir validé la commande
laisse une fenêtre où la commande existe sans son événement.

## Décision

Retenir RabbitMQ.

Publier les commandes depuis la boutique par outbox transactionnel : Messenger
écrit l'événement dans la base de la boutique via le transport `doctrine://`,
dans la même transaction que la commande, et un relais l'expédie ensuite vers
AMQP.

## Conséquences

- Aucune commande ne peut exister sans son événement, ni l'inverse. Le test de
  reprise après panne devient démontrable au lieu d'être déclaratif.
- Le relais est un composant supplémentaire à superviser. Son retard est une
  métrique à exposer sur le tableau de bord.
- L'interface de gestion de RabbitMQ rend les files, les rejeux et la file
  d'échec visibles, ce qui a une valeur de démonstration directe.
- RabbitMQ est le plus lourd des candidats. Ce n'est plus un problème compte
  tenu du gabarit retenu.

## Options écartées

**NATS JetStream.** Plus léger et agréable côté Go. Écarté parce qu'il n'existe
pas de transport Messenger de première main : il faudrait auditer un transport
communautaire ou l'écrire, et le risque se déplacerait sur la brique Sylius,
celle où l'on veut le moins de code maison.

**Valkey Streams.** Transport de première main côté PHP, mais routage pauvre, et
une seconde instance Valkey serait nécessaire pour ne pas mélanger cache et
messagerie. Le gain d'un composant en moins est illusoire.
