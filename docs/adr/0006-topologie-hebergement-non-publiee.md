# ADR 0006, la topologie d'hébergement n'est pas publiée

- Statut : accepté
- Date : 2026-09-10

## Contexte

Ce dépôt est public. La démonstration qu'il décrit est un honeypot assumé.

Documenter publiquement où et sur quoi tourne l'instance revient à indiquer à un
attaquant ce qu'il gagne s'il sort du conteneur. C'est précisément le
renseignement qu'un honeypot ne doit pas fournir. Le rayon de souffle réel fait
partie de ce qui se protège, au même titre qu'un secret.

Il y a une tension avec l'esprit du projet, qui est de tout documenter. Elle se
résout en distinguant deux choses. La transparence sur l'architecture et sur ses
propriétés vérifiables est un atout. La transparence sur le rayon de souffle est
un cadeau fait à l'attaquant.

## Décision

La topologie d'hébergement n'est pas documentée dans ce dépôt : ni l'hébergeur,
ni le pays, ni le matériel, ni le plan d'adressage, ni la structure réseau située
derrière le tunnel.

Le raisonnement complet, les options écartées et les contrôles compensatoires
sont consignés hors de ce dépôt.

Reste public ce qui est nécessaire pour comprendre l'architecture et vérifier ses
propriétés :

- L'entrée se fait uniquement par un tunnel sortant. Aucun port n'est ouvert en
  entrée.
- La terminaison TLS est déportée, donc aucun secret de zone DNS ne réside dans
  le cluster.
- La sortie réseau est en refus par défaut, avec une exception unique et nommée.
- L'infrastructure est dédiée à ce seul projet et isolée de tout autre système.
  L'isolement est un contrôle testé, pas seulement configuré.

## Conséquences

- Un lecteur ne peut pas reproduire à l'identique le déploiement public. Il peut
  reproduire la pile applicative, qui est l'objet de ce dépôt.
- La revue de sécurité de la phase 2 existe en deux versions. Une publique, qui
  traite les propriétés vérifiables. Une privée, qui traite la topologie.
- **La règle s'applique aux messages de commit et aux corps de pull request.**
  Ils sont une surface publique au même titre que les fichiers versionnés.
- Toute contribution future doit être relue sous l'angle « qu'est-ce que ceci
  révèle », et pas seulement « est-ce que ceci est correct ».

## Options écartées

**Publier la topologie complète.** Cohérent avec l'esprit de transparence du
projet et avec la valeur pédagogique recherchée. Écarté : dans une vitrine dont
l'objet est de résister à des attaques réelles, publier ce qui se trouve derrière
la porte annule le bénéfice de l'avoir verrouillée.

**Ne rien dire du tout de l'exposition réseau.** Écarté également. Les propriétés
listées ci-dessus sont vérifiables de l'extérieur et constituent l'essentiel de
la démonstration. Les taire n'apporterait aucune sécurité et retirerait sa valeur
au document.
