# cdhdt-sandbox, spécification de cadrage

- Statut : proposée
- Date : 2026-09-10
- Auteur : cdhdt
- Phase : 0, cadrage

Ce document fixe le périmètre, l'architecture et les règles de la démonstration
avant toute ligne de code applicatif. Il est le contrat entre les phases. Chaque
décision structurante renvoie à un ADR dans `docs/adr/`.

---

## 1. Objet et non-objectifs

### 1.1 Objet

`cdhdt-sandbox` est une infrastructure e-commerce publique, complète et
reproductible, présentée comme une fiction d'un environnement réel. Elle sert de
vitrine sur trois compétences : le développement, l'intégration de flux entre
progiciels, et la sécurité d'un système exposé sur Internet.

Le durcissement fait partie du produit. Ce n'est pas une étape finale.

### 1.2 Ce que la démonstration doit prouver

1. Une boutique, un PIM et un ERP réels échangent par événements, sans qu'aucun
   ne connaisse le format d'un autre.
2. Les flux sont idempotents et reprennent après panne, et cela se démontre par
   des tests, pas par une affirmation.
3. Un système complet peut être exposé publiquement sans être exploitable, et la
   preuve est écrite, point par point.
4. Tout se reconstruit en une commande, et se remet à zéro en une commande.

### 1.3 Non-objectifs

- **Le mode bac à sable est hors périmètre.** Aucun visiteur ne modifie le
  catalogue, les prix ou le stock. La question sera rouverte seulement quand les
  phases 2 et 3 auront tenu un mois sans incident.
- Aucune transaction financière réelle, à aucun moment.
- Aucune performance de production. La démonstration vise la justesse, pas le
  débit.
- Aucune donnée personnelle de tiers n'entre dans le système, par construction
  et non par politique de confidentialité.

### 1.4 Attente créée par le nom

Le domaine contient le mot « sandbox ». Un visiteur supposera donc qu'il peut
écrire. La page d'accueil doit dire immédiatement ce qui est possible et ce qui
ne l'est pas, sans quoi le nom crée une promesse que le produit ne tient pas.

---

## 2. Frontières et rayon de souffle

Cette section est volontairement franche sur les propriétés du système. Elle est
volontairement muette sur ce qui se trouve derrière.

### 2.1 Ce qui n'est jamais publié

Ce dépôt est public et la démonstration est un honeypot assumé. Certaines
informations ne sont donc pas publiables, non par discrétion mais parce que les
publier réduirait la sécurité du dispositif.

Ne sont jamais publiés, dans ce dépôt ni ailleurs :

- L'hébergeur, le pays, le matériel, le fournisseur d'accès.
- Le plan d'adressage et la structure réseau situés derrière le tunnel.
- Toute information répondant à la question « qu'est-ce que l'attaquant gagne
  s'il sort du conteneur ».

**Cette règle s'applique aux fichiers versionnés, aux messages de commit et aux
corps de pull request.** Ces trois surfaces sont publiques au même titre.

Toute contribution est relue sous l'angle « qu'est-ce que ceci révèle », en plus
de « est-ce que ceci est correct ». Voir l'ADR 0006.

### 2.2 Hébergement

La topologie d'hébergement n'est pas documentée ici. C'est une décision
délibérée, consignée dans l'ADR 0006.

Ce qui est public parce que vérifiable de l'extérieur, et suffisant pour
comprendre l'architecture :

- L'infrastructure est dédiée à ce seul projet et isolée de tout autre système.
- L'entrée se fait uniquement par un tunnel sortant. **Aucun port n'est ouvert en
  entrée.**
- La sortie réseau est en refus par défaut, avec une exception unique et nommée.

### 2.3 Rayon de souffle assumé

Un échappement de conteneur donne à l'attaquant la machine hôte. Les contrôles
qui subsistent alors sont l'isolement réseau de l'infrastructure et le refus de
sortie. Ce sont donc les deux contrôles qui doivent être **testés**, pas
seulement configurés.

La remise à zéro toutes les vingt-quatre heures efface la démonstration. Elle
n'efface pas ce qu'un attaquant aurait installé ailleurs. Cette limite est
assumée et documentée, elle n'est pas une omission.

### 2.4 Sortie réseau

Refus par défaut sur tous les namespaces applicatifs. **Une seule exception
nommée : le pod du tunnel.** Aucun autre pod n'a de sortie vers Internet.

Le critère de sortie de la phase 2 n'est donc pas « aucune sortie » mais
« aucune sortie hors de cette liste d'un élément », ce qui est vérifiable par un
test automatisé.

Le DNS interne et les communications entre pods du même namespace restent
autorisés, nommément et non par défaut.

### 2.5 TLS et certificats

La terminaison TLS est déportée en amont du tunnel. Conséquence directe et
souhaitable : aucun défi ACME à l'origine, donc pas de `cert-manager`, pas de
webhook DNS tiers, et surtout **aucun secret de zone DNS dans le cluster**.

C'est le pire secret à laisser dans un système conçu pour être attaqué, et il
n'y sera pas.

### 2.6 Ce qui est public

Toutes les interfaces applicatives sont publiquement accessibles, administrations
comprises. C'est une décision assumée, prise contre l'avis rendu pendant le
cadrage. Elle est documentée dans l'ADR 0007 avec ses contreparties, notamment
l'engagement de suivi des CVE de l'ADR 0002.

Aucun identifiant en écriture n'est publié, quelle que soit l'interface.

### 2.7 Certificate Transparency

Tout certificat émis par une autorité publique est publié dans les journaux de
Certificate Transparency. Aucun sous-domaine n'est donc discret. La conception ne
doit à aucun moment reposer sur le fait qu'un nom d'hôte serait inconnu.

---

## 3. Le modèle pivot

### 3.1 Principe

Aucun système ne connaît le format d'un autre. Nous définissons nos propres
schémas d'événements, versionnés dans le dépôt. Chaque système reçoit un
adaptateur Go qui traduit dans les deux sens entre son modèle et le pivot.

### 3.2 Les entités du pivot

| Entité | Source de vérité | Consommateurs |
|---|---|---|
| Produit | Pimcore | Boutique |
| Prix | ERPNext | Boutique |
| Stock | ERPNext | Boutique |
| Commande | Boutique | ERPNext |

### 3.3 Règles de schéma

- Chaque événement porte un identifiant stable, une version de schéma, un
  horodatage d'émission et une **clé d'idempotence**.
- La clé d'idempotence est déterministe et dérivée du contenu métier, jamais
  d'un identifiant technique de message. Rejouer le même événement doit être un
  non-événement.
- Les évolutions de schéma sont **ascendantes uniquement** à l'intérieur d'une
  version majeure. Ajouter un champ optionnel est permis, retirer ou renommer ne
  l'est pas.
- Les schémas vivent dans le dépôt et sont la seule référence. Un adaptateur qui
  ne valide pas contre le schéma rejette le message vers la file d'échec.

### 3.4 Ordre et concurrence

Les événements de prix et de stock portent un numéro de version croissant par
référence produit. Un consommateur ignore silencieusement un événement dont la
version est inférieure ou égale à celle qu'il détient déjà. Cela rend le
traitement correct même si l'ordre de livraison n'est pas garanti.

---

## 4. Les quatre flux

Pour chacun : le contrat, la garantie de livraison, le comportement en panne.

### 4.1 Pimcore vers boutique, modifications produit

- Déclencheur : modification d'un produit dans Pimcore.
- Contrat : événement `product.updated` au format pivot.
- Garantie : au moins une fois. L'idempotence est portée par la clé et par le
  numéro de version.
- En panne de la boutique : les messages s'accumulent dans la file. À la reprise,
  ils sont consommés dans l'ordre, les versions périmées sont ignorées.

### 4.2 Boutique vers ERPNext, commandes

- Déclencheur : commande validée dans la boutique.
- Fiabilité : **outbox transactionnel.** Symfony Messenger écrit l'événement
  dans la base de la boutique via le transport `doctrine://`, dans la même
  transaction que la commande. Un relais l'expédie ensuite vers AMQP.
- Conséquence recherchée : aucune commande ne peut exister sans son événement,
  ni l'inverse. C'est ce qui rend le test de reprise après panne démontrable.
- En panne d'ERPNext : les commandes continuent d'être acceptées et
  s'accumulent. Aucune perte, aucun refus côté visiteur.

### 4.3 ERPNext vers boutique, prix et stock

- Déclencheur : modification de prix ou de mouvement de stock dans ERPNext.
- Modèle : **poussé, jamais interrogé à la lecture.** La boutique lit sa propre
  base.
- Motif : une panne d'ERPNext ne doit pas rendre la vitrine inutilisable, et une
  page produit ne doit pas dépendre d'un appel synchrone vers un progiciel.

### 4.4 Invalidation de cache

- Déclencheur : tout événement qui rend une page obsolète.
- Mécanisme : un consommateur dédié traduit l'événement en purge par étiquette
  vers Varnish.
- Contrainte réseau : cet appel est interne au namespace. Il est autorisé
  nommément dans la NetworkPolicy, sans rouvrir de sortie vers Internet.
- L'interface de purge de Varnish n'est joignable que depuis l'intérieur.

---

## 5. Stratégie de cache

### 5.1 Répartition

- **Varnish** en cache de pages, devant la boutique.
- **Valkey** en cache applicatif et en stockage de sessions, derrière.

Le brief initial parlait d'un cache Valkey devant la boutique. La documentation
décrira ce qui tourne réellement. Voir l'ADR 0005.

### 5.2 Ce qui se cache

| Contenu | Mise en cache | Durée |
|---|---|---|
| Page d'accueil et pages de contenu | Oui | longue |
| Listes de catégories | Oui | moyenne |
| Fiche produit, partie descriptive | Oui | moyenne |
| Prix | **Jamais** | sans objet |
| Stock | **Jamais** | sans objet |
| Panier, tunnel de commande | **Jamais** | sans objet |
| Interfaces d'administration | **Jamais** | sans objet |

Les durées exactes sont fixées à l'implémentation et documentées dans le README.

### 5.3 Pourquoi les prix ne se cachent jamais

Le prix dépend d'un groupe tarifaire, professionnel ou particulier. Le visiteur
peut choisir son groupe tarifaire, sinon la règle resterait une affirmation
invérifiable. Les prix et le stock sont donc servis hors de la page mise en
cache, et l'invalidation par notification porte sur la partie descriptive.

---

## 6. Surface publique et durcissement

Revue point par point contre les règles non négociables du projet. La revue de
sécurité complète est le livrable de sortie de la phase 2. Cette section fixe
les exigences.

| Règle | Traitement |
|---|---|
| Aucun compte en écriture public | Compte admin Sylius en lecture seule, construit et testé. Les comptes réels restent dans un gestionnaire de secrets. |
| Aucun trafic sortant | Refus par défaut, une exception nommée, `cloudflared`. Test automatisé de preuve. |
| Aucun upload public | Désactivé. Si un flux le rendait nécessaire, quarantaine sans exécution ni service direct. |
| Quotas et limitation de débit | Quotas CPU et mémoire sur chaque charge. Limitation de débit à l'entrée, qui protège autant la ressource que la sécurité. |
| Faux PSP | Liste fermée de cartes de test. Bandeau affiché avant tout formulaire. Tout autre numéro est refusé. |
| Aucun secret dans le dépôt | Protection au push GitHub, analyse de secrets, et vérification de l'historique. |
| Journalisation intégrale | Voir section 7. |
| Portfolio non impliqué | Aucun sous-domaine, aucune redirection, aucun cookie partagé avec le domaine du portfolio. |

### 6.1 Le compte admin en lecture seule

Sylius ne fournit pas nativement un rôle en lecture seule. Il faut le construire
en refusant toute méthode HTTP non sûre pour cet utilisateur sur le préfixe
d'administration. C'est un développement à part entière, avec ses tests, pas une
case à cocher.

### 6.2 Le checkout invité

Le tunnel de commande est ouvert aux visiteurs anonymes. Le champ d'adresse
électronique est **prérempli et verrouillé** avec une adresse générée par
session, sur un domaine en `.invalid`.

Motif : le domaine de premier niveau `.invalid` est réservé par la RFC 2606 et
n'est jamais routable. Aucune adresse saisie par un tiers n'entre donc dans le
système, et Mailpit peut rester public sans exposer de donnée personnelle.

### 6.3 Mailpit

Consultable publiquement en lecture. Les méthodes d'écriture de son API, dont la
suppression, sont bloquées à l'entrée.

### 6.4 Isolement identitaire

Le portfolio parlera publiquement de la démonstration. L'isolement entre les
deux domaines est donc **technique, pas identitaire** : un visiteur de la
démonstration peut remonter à son auteur. Ce point doit figurer dans la revue de
sécurité plutôt que d'être découvert. Le lien depuis le portfolio portera
`rel="noopener noreferrer"` et ne devra pas fuiter de `Referer`.

---

## 7. Honeypot et journalisation

### 7.1 Principe

Tout ce que fait un visiteur est journalisé et conservé. La démonstration est un
honeypot assumé, et les observations alimenteront des billets de blog.

### 7.2 L'adresse IP réelle

Derrière un tunnel, l'adresse réelle du visiteur arrive par en-tête. Deux règles :

1. La journalisation la capture.
2. **Cet en-tête n'est jamais accepté s'il provient d'autre chose que du
   tunnel.** Sinon n'importe qui falsifie les journaux, ce qui ruine l'intérêt
   du dispositif.

### 7.3 Rétention

| Catégorie | Durée | Dans l'instantané |
|---|---|---|
| Journaux applicatifs | 30 jours | non |
| Journaux d'observation, pseudonymisés | 12 mois | non |
| Métriques Prometheus | 15 jours | non |
| Données de démonstration | jusqu'au prochain instantané | oui |

### 7.4 Mention publique

La page d'accueil indique que la navigation est journalisée et à quelle fin. La
formulation retenue sera une proposition à valider par l'auteur du projet. Ce
document ne constitue pas un avis juridique.

---

## 8. Remise à zéro par instantané

- Déclenchement planifié toutes les vingt-quatre heures, à heure fixe.
- Déclenchement manuel par une commande unique.
- Aucune commande de remise à zéro n'est exécutée sans validation préalable
  explicite de l'auteur du projet.

### 8.1 Ce qui est effacé

Les bases de la boutique, de Pimcore et d'ERPNext, le contenu de Mailpit, l'état
des files, le cache Varnish et Valkey.

### 8.2 Ce qui survit

Les journaux d'observation et les métriques. Ils sont **hors instantané**, sur
un volume distinct. Sans cette séparation, la remise à zéro détruirait l'objet
même du honeypot toutes les vingt-quatre heures.

### 8.3 Méthode

Restauration d'un instantané de volume, jamais un rejeu d'installation. Le
premier démarrage d'ERPNext prend plusieurs minutes, ce qui est acceptable une
fois mais pas quotidiennement.

---

## 9. Tests

Développement piloté par les tests sur tout ce que nous écrivons nous-mêmes :
les adaptateurs, le faux PSP, les consommateurs de files, la stratégie de cache
et le compte en lecture seule.

### 9.1 Tests unitaires

Sur chaque adaptateur : traduction dans les deux sens, rejet d'un message
invalide, calcul de la clé d'idempotence.

### 9.2 Tests d'intégration

Ils portent sur des propriétés, pas sur des scénarios heureux.

1. **Idempotence** : le même événement livré deux fois produit exactement un
   effet.
2. **Reprise après panne** : un consommateur arrêté en cours de traitement, puis
   redémarré, ne perd ni ne duplique aucun message.
3. **Ordre** : un événement de version inférieure arrivé après un plus récent est
   ignoré.
4. **File d'échec** : un message invalide finit dans la file d'échec, il ne
   bloque pas la file principale.
5. **Invalidation ciblée** : une modification de prix invalide la page concernée
   et **uniquement** celle-là.

### 9.3 Test de preuve d'isolement réseau

Un test automatisé tente une sortie vers Internet depuis chaque namespace
applicatif et échoue si l'une aboutit. La liste d'exception attendue contient un
seul élément, `cloudflared`.

### 9.4 Test d'isolement de l'infrastructure

Une tentative d'accès depuis l'infrastructure de la démonstration vers tout autre
réseau doit échouer. Ce contrôle est testé, pas seulement configuré.

Sa mise en oeuvre exacte relève de la topologie et n'est pas documentée ici. Le
résultat du test, lui, figure dans la revue de sécurité publique.

---

## 10. Dépôt, intégration continue et lots

### 10.1 Dépôt

Dépôt unique et public, `cdhdt-sandbox`, sous l'organisation GitHub du même nom.

Motif du dépôt unique : un lot touchera souvent à la fois un adaptateur Go, un
manifeste Kubernetes et un ADR. En dépôts séparés, le critère « rien n'est
fusionné sans build vert » perd son sens.

### 10.2 Identité

- Auteur des commits : `cdhdt`, `cdhoudetot@ik.me`.
- Une seule adresse électronique apparaît dans tout le dépôt.
- Aucune information sensible : ni adresse IP, ni nom d'hôte réel autre que le
  domaine du projet, ni chemin local, ni nom d'employeur.

### 10.3 Les trois déclencheurs d'intégration continue

Ce découpage vient d'un constat : une CI qui ne se déclenche que sur les pull
requests n'alertera jamais d'une CVE apparue sans commit.

1. **Sur pull request** : construction, tests, linters, revue de dépendances.
   C'est le critère de fusion.
2. **Planifiée, quotidienne** : analyse des images réellement déployées, Pimcore
   et ERPNext compris.
3. **En continu** : Dependabot sur `composer.json`, `go.mod`, les Dockerfiles et
   les actions GitHub.

Sur un dépôt public, GitHub fournit sans surcoût l'analyse de secrets, la
protection au push, les alertes et mises à jour Dependabot, l'analyse de code
CodeQL et la revue de dépendances.

### 10.4 Politique de correctifs

Les délais d'application et la conduite à tenir quand aucun correctif n'existe
encore sont définis dans l'ADR 0002, qui porte l'engagement de suivi des CVE.
Le principe directeur : une interface pour laquelle aucun correctif n'existe est
retirée de l'exposition publique, elle n'est pas laissée en ligne avec une
mitigation partielle.

### 10.5 Langue

Spécification, ADR et README en français. Code, identifiants et messages de
commit en anglais.

### 10.6 Découpage en lots

Une branche et une pull request par lot. Rien n'est fusionné sans build vert et
sans relecture de l'auteur du projet.

| Lot | Contenu | Phase |
|---|---|---|
| 0 | Cadrage : cette spécification et les ADR | 0 |
| 1 | Socle local : k3d, manifestes, Sylius, Valkey, Varnish, Mailpit | 1 |
| 2 | RabbitMQ, schémas pivot, squelette des adaptateurs | 1 |
| 3 | Pimcore et son adaptateur, flux produit | 1 |
| 4 | ERPNext et son adaptateur, flux prix et stock | 1 |
| 5 | Outbox et flux commande vers ERPNext | 1 |
| 6 | Faux PSP et checkout invité à adresse verrouillée | 1 |
| 7 | Invalidation de cache par notification | 1 |
| 8 | Observabilité, Prometheus, Grafana, journalisation | 1 |
| 9 | Durcissement, NetworkPolicy, quotas, compte lecture seule | 2 |
| 10 | Instantané et remise à zéro | 2 |
| 11 | Revue de sécurité écrite et tests de preuve | 2 |
| 12 | Déploiement public, tunnel, page d'accueil | 3 |
| 13 | Documentation | 4 |

---

## 11. Points à vérifier avant la phase 3

Éléments non vérifiés à ce jour, à confirmer avant engagement.

- Compatibilité exacte de la chaîne de cache retenue avec Sylius 2.2.
- Ce que le plan Cloudflare retenu inclut en règles WAF gérées et en limitation
  de débit. Cela conditionne le budget et ce qui peut être annoncé.
- Empreinte mémoire et processeur réelle de la pile complète, mesurée en phase 1.
  Les estimations du cadrage ne sont pas des mesures.
- Gabarit final de la machine, déduit de cette mesure et non l'inverse.

---

## 12. Références vérifiées au 2026-09-10

- Cycle de publication Sylius : 2.2 publiée le 2025-12-17, maintenue jusqu'en
  novembre 2026, supportée jusqu'en mars 2027. Dernier correctif 2.2.9 le
  2026-09-02. 2.3.0-ALPHA.1 le 2026-08-10. 1.14 LTS en sécurité seule jusqu'en
  décembre 2026.
- Symfony Messenger fournit les transports `amqp://`, `doctrine://` et
  `redis://`, une stratégie de reprise paramétrable et un `failure_transport`.
- ERPNext, recommandation de Frappe : au moins 4 Go de mémoire, 2 processeurs et
  40 Go de stockage. Prérequis v15 : Python 3.10.12 ou plus, Node.js 18 ou plus,
  MariaDB 10.6 ou plus, Redis.
- Pimcore : InnoDB et utf8mb4 requis, `memory_limit` PHP d'au moins 150M en
  fonctionnement et 512M pour l'installation. Versionnage calendaire, ligne
  `2026.x`.
- `cert-manager` prend nativement en charge huit fournisseurs DNS-01. Les autres
  passent par un webhook hors arbre, ce qui suppose une clé d'API de zone dans le
  cluster. Sans objet ici, la terminaison TLS étant déportée.
- RFC 2606 : le domaine de premier niveau `.invalid` est réservé et non routable.
- Sur un dépôt public, GitHub fournit sans GitHub Advanced Security l'analyse de
  secrets, la protection au push, Dependabot, CodeQL et la revue de dépendances.
