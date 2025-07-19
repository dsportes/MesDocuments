---
layout: page
title: Clés d'accès, l'application "KeyRing"
---

# Principe d'authentification dans les applications

Une application `app1` met en jeu deux parties:
- `app1Srv` : un pool de serveurs au service de l'application et gérant ses données centrales persistantes.
- `app1App` : une application terminale, typiquement une application Web dont le source est lisible et délivrée par un serveur statique / CDN.

Les opérations exécutées par le serveur comme les données qu'il peut retourner à l'application qui l'a sollicité, sont soumise, sauf exception, à des **droits d'accès**: d'une manière ou d'une autre l'utilisateur derrière l'application terminale doit exhiber qu'il possède effectivement le droit pour lancer une opération ou obtenir des données centrales. Par exemple:
- droit à lire les documents d'une personne ou d'un compte,
- droit de gestion des accès à un groupe de comptes,
- droit de modification tarifaire ...

### Droit d'accès
Un droit d'accès est matérialisé par trois données:
- son `id` : son identifiant unique est géré par l'application et peut comporter aussi bien des données lisibles (une adresse e-mail, un numéro de mobile ...) qu'être le résultat d'une génération aléatoire.
- son couple de clés:
  - `kv` : clé publique de **vérification**,
  - `ks` : clé privée de **signature**.

Le serveur d'une application gère les droits et n'en connaît **QUE** le couple id `kv`, sous aucune condition il n'a jamais accès même temporaire à le clé `ks` correspondante.

### Jetons d'accès
Quand l'application terminale soumet une opération au serveur, elle fournit dans sa requête un jeton d'accès qui va réunir toutes les preuves que son utilisateur dispose des droits d'accès requis pour exécuter cette opération. Un jeton comporte:
- `devAppId` : l'identification de l'application terminale s'exécutant sur un _device_. Cet id est unique, ne désigne qu'un seul couple _device / application_, à un instant donné une seule exécution peut s'en prévaloir.
  - Remarque: un _hacker_ un peu entraîné peut obtenir cet identifiant en lançant en _debug_ l'application sur ce _device_.
- `time` : date-heure en milliseconde de la génération du jeton d'accès.
- une liste de preuves de possession d'un droit d'accès constituée chacune d'un couple:
  - `id` du droit,
  - `signature` : signature du couple `devAppId, time` par la clé `ks` du droit correspondant.

> L'application terminale à obtenu la clé ks en la demandant à l'utilisateur: celle-ci n'est restée disponible en mémoire que le temps d'effectuer l'opération de signature. Elle est certes lisible en _debug_ mais puisque c'est l'utilisateur qui la fournit on ne voit pas pourquoi il utiliserait un procédé compliqué pour lire une donnée qu'il a le droit de connaître et connaît.

Quand le serveur traite une opération il va commencer par établir la liste des _droits d'accès_ validés depuis le jeton d'accès attaché à la requête. Le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _preuve_ de la liste du jeton,
- obtention depuis la base de données de la clé `kv` associée à son id,
- vérification que la `signature` (par `ks` das l'application terminale) du couple `devAppId, time` (utilisé comme _challenge_ cryptographique ne devant pas être présenté plus d'une fois) est bien validée par `kv`.

> La _vérification cryptographique_ répond ok si le texte `t` signé par `ks` est bien égal à la signature transmise.

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `id` d'un droit le dernier `ks` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `ks`, et cas de changement refait une vérification avant de valider / invalider le droit correspondant.
- le serveur conserve en cache pour chaque `devAppId` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, la _validation_ des droits ne requiert qu'un calcul en mémoire sans accès à la base.

> Ce procédé signature / vérification assure que les serveurs n'ont **JAMAIS** eu besoin de détenir la clé `ks` d'un droit pour en vérifier la validité et que celle-ci a pu rester cantonnée dans la mémoire d'une application terminale s'exécutant pour un utilisateur en ayant le droit.

# Gestion par un utilisateur des couples id / ks des droits qu'il a
Quelqu'en soit le processus, à un moment donné un utilisateur a acquis un **droit** ce qui s'est manifesté par le fait qu'il connaisse le couple `id / ks` correspondant.

> Il peut aussi en connaître la clé `kv`: celle-ci étant _publique_, n'importe qui peut lire la clé `kv` d'un droit d'id donné.

Le terme `id` peut être abscons, difficile à mémoriser.

Le terme `ks` est _impossible_ à mémoriser, constitué d'environ 400 lettres et chiffres d'apparence aléatoire.

### Fichier de texte des _droits_ d'un utilisateur
L'utilisateur doit a minima recourir à un fichier de texte dans lequel il a enregistré ceux-ci. De plus cette liste est imbuvable et pour chaque terme `id / ks` il lui faut associer un libellé parlant pour lui pour savoir lequel sélectionner quand l'application lui demandera d'un choisir un. 

### Demande de droits d'une application
Deux situations se présente:
- (1) **l'application connaît l'id exacte d'un droit** (par exemple `DRTARIF` : droit à effectuer une modification tarifaire) et demande à l'utilisateur sa clé ks correspondant à ce droit.
- (2) **l'application ne connaît qu'un préfixe d'un droit** (par exemple `LOGIN_...` : droit d'un utilisateur à se connecter) et demande à l'utilisateur,
  - de fixer _quel_ login  en donnant l'id complète : `LOGIN_me@example.com`.
  - de donner le `ks` correspondant.

L'application peut charger en mémoire le fichier de texte de l'utilisateur contenant ses droits et pour autant que la syntaxe en soit fixée,
- dans le cas (1) peut obtenir le ks,
- dans le cas (2) peut présenter à l'utilisateur la liste des droits possibles contenus dans le fichier et sa sélection retourne le couple `id, ks` correspondant au choix de l'utilisateur.
