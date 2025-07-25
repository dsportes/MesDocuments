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

> L'application terminale à obtenu la clé `ks` en la demandant à l'utilisateur: celle-ci n'est restée disponible en mémoire que le temps d'effectuer l'opération de signature. Elle est certes lisible en _debug_ mais puisque c'est l'utilisateur qui la fournit on ne voit pas pourquoi il utiliserait un procédé compliqué pour lire une donnée qu'il a le droit de connaître et connaît.

Quand le serveur traite une opération il va commencer par établir la liste des _droits d'accès_ validés depuis le jeton d'accès attaché à la requête. Le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _preuve_ de la liste du jeton,
- obtention depuis la base de données de la clé `kv` associée à son id,
- vérification que la `signature` (par `ks` das l'application terminale) du couple `devAppId, time` (utilisé comme _challenge_ cryptographique ne devant pas être présenté plus d'une fois) est bien validée par `kv`.

> La _vérification cryptographique_ répond ok si le texte `t` signé par `ks` est bien égal à la signature transmise.

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `id` d'un droit le dernier `ks` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `ks`, et cas de changement refait une vérification avant de valider / invalider le droit correspondant.
- le serveur conserve en cache pour chaque `devAppId` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, la _validation_ des droits ne requiert qu'un calcul en mémoire sans accès à la base.

> Ce procédé signature / vérification assure que les serveurs n'ont PAS besoin de détenir la clé `ks` d'un droit pour en vérifier la validité et que celle-ci a pu rester cantonnée dans la mémoire d'une application terminale s'exécutant pour un utilisateur en ayant le droit.

## "Un" droit, "plusieurs" clés
Le serveur _peut_ mémoriser pour un droit non pas une clé `kv` mais une liste de clés `[kv1, kv2 ...]`: pour être validé, un jeton d'accès doit fournir une signature qui a été établie par une des clés `ks` correspondantes.

La logique applicative peut ainsi prévoir de distribuer plusieurs clés à des détenteurs différents selon ses propres critères: elle peut aussi au cours du temps en _désactiver_ certaines dans le serveur et inhiber ainsi les détenteurs qui les possédaient.

Dans un jeton généré par l'application, s'il existait _plusieurs_ clés possibles, au lieu d'une signature dans le jeton pour un droit donné, c'est une liste de signatures (une par clé potentielle) qui est générée. Il suffit que l'une d'entre elles _matche_ avec la ou une des clés kv détenues par le serveur pour que le droit soit validé.

> Ce dispositif permet de gérer un _changement de clé_ pour un droit lorsqu'il a été attribué trop généreusement. La création d'une nouvelle clé peut être faite et sa distribution restreinte aux seuls détenteurs souhaitables dans le futur: après un certain temps où les deux sont admises, _l'ancienne_ peut être supprimée.

# Gestion par un utilisateur des couples id / ks des droits qu'il a acquis
Quelqu'en soit le processus, à un moment donné un utilisateur a acquis un **droit** ce qui s'est manifesté par le fait qu'il connaisse le couple `id / ks` correspondant.

> Il peut aussi en connaître la clé `kv`: celle-ci étant _publique_, n'importe qui peut lire la clé `kv` d'un droit d'id donné.

Le terme `id` peut être abscons, difficile à mémoriser.

Le terme `ks` est _impossible_ à mémoriser, constitué d'environ 400 lettres et chiffres d'apparence aléatoire.

## Fichier CSV des _droits_ d'un utilisateur
L'utilisateur doit a minima recourir à un fichier dans lequel il a enregistré ses droits. 

Cette liste est en soi incompréhensible  et pour chaque terme `id / ks` il lui faut associer un libellé parlant pour l'utilisateur afin qu'il sache quel droit sélectionner quand l'application lui demandera d'un choisir un. 

Le fichier peut être un simple CSV ayant quatre colonnes: `application, libellé, id, ks`.

> Remarque: `ks` peut être en fait une liste de `ks` séparés par un espace.

### Demande de droits d'une application
Lorsque l'application souhaite avoir la preuve d'un droit de l'utilisateur, deux situations se présentent:
- (1) **l'application connaît l'id exacte d'un droit** (par exemple `DRTARIF` : droit à effectuer une modification tarifaire) et demande à l'utilisateur sa clé `ks` correspondant à ce droit.
- (2) **l'application ne connaît qu'un préfixe d'un droit** (par exemple `LOGIN_...` : droit d'un utilisateur ... à se connecter) et demande à l'utilisateur,
  - de fixer _quel_ login  en donnant l'id complète : `LOGIN_me@example.com`.
  - de donner le `ks` correspondant.

L'application peut charger en mémoire le fichier CSV des droits de l'utilisateur lui demandant son _path_ sur le file-system local du _device_:
- dans le cas (1) l'application peut obtenir directement le `ks`,
- dans le cas (2) l'application peut présenter à l'utilisateur la liste des droits possibles contenus dans le fichier. La sélection de l'utilisateur dans la liste retourne le couple `id, ks` correspondant à son choix.

> Remarque: le fichier CSV est une des solutions simplistes possibles. Une autre solution encore plus simpliste est que l'application présente une double zone de texte à saisir dans laquelle l'utilisateur _colle_ le couple `id ks` de son choix après l'avoir _copié_ quelque part (un mail, un gestionnaire de clés ...).

## L'application _KeyRing_
La gestion d'un fichier CSV de leurs droits par les utilisateurs met en lumière le problème de gérer ses droits d'accès par un utilisateur:
- le fichier CSV doit pouvoir être accessible depuis plusieurs _devices_, stocké sur le _cloud_ ...
- il doit être encrypté afin de ne pas exposer ses clés à des pirates et l'utilisateur doit se rappeler de sa clé de cryptage,
- la mise à jour du fichier suppose d'être vigilant: l'inscription d'une nouvelle clé (voire le _changement_ d'une clé) est un risque de dégradation / perte du fichier et en conséquence de tous ses droits précieusement récupérés au fil du temps.

Pour résoudre ce problème, l'application **KeyRing** (un couple _application / serveur_), propose aux utilisateurs qui le souhaitent une solution pour gérer leur **anneau de clés** avec une confidentialité rigoureuse et ce pour toutes les applications qui ont implémenté son protocole.

Dans cette application un utilisateur déclare un _anneau_ identifié et sécurisé par une _phrase secrète_: il pourra ainsi accrocher ses propres clés à cet _anneau_.

> Un utilisateur peut se déclarer plusieurs anneaux pour convenances personnelles mais devra identifier lequel il utilise quand il ouvrira une application, exactement comme il l'aurait fait s'il avait plus d'un fichier CSV contenant ses clés.

## Acquisition de droits par un utilisateur
Il existe plusieurs processus pour acquérir un droit selon le protocole applicatif choisi.

### Acquisition directe dans l'application terminale
Dans cette situation c'est l'application terminale qui génère le _droit_.

L'exemple pris ici est _la création d'un compte dans l'application_:
- l'application a généré l'id du compte ou l'a obtenu du serveur.
- l'application a généré un couple de clés ks / kv.
- elle fait enregistrer par exemple par KeyRing le couple id, ks (ou affiche ce couple à l'utilisateur pour qu'il l'inscrive dans son fichier CSV de droits).
- l'application _valide_ la création du compte auprès du serveur pour qu'il enregistre en base le couple id / kv.

### Obtention par l'application terminale d'un droit stocké dans le serveur
Un certain nombre de droits peuvent être _configurés_ dans le serveur: en d'autres termes ils sont inscrits, soit _en dur_ (et chargés à l'initialisation), soit _en base_ une autorité _supérieure_ ayant le droit de créer des droits identifiés.

> Les ks de ces droits ne doivent être lisibles directement en base afin qu'un détournement de celle-ci ne donne pas accès à ces clés. 

Dans ce cas, l'application terminale récupère depuis le serveur le couple `id, ks` du droit attribué à  l'utilisateur et fait enregistrer ce couple par KeyRing (ou le présente à l'utilisateur pour incorporation dans son fichier CSV ou autre). 

### Attachement à un autre droit
Le serveur dispose pour un droit `dx` non pas d'une clé `ks` mais d'une liste de droits `d1, d2 ...` Par exemple:
- `dx` est un droit de gestion d'un tarif,
- `d1 d2 ...` sont les logins à qui ce droit a été attribué.

Pour exécuter une opération requérant le droit `dx`, il suffit que le _jeton_ ait un des droits `d1 d2 ...`

### Transmission explicite entre détenteurs de droits
Si une personne P1 dispose d'un droit c'est matérialisé par l'existence _quelque part_ de son couple `id, ks` associé.

Basiquement P1 peut par exemple transmettre à une personne P2 ce droit par un simple e-mail à l'adresse de P2 qui l'ajoutera à son anneau KeyRing (ou à son fichier CSV).

Ce procédé général _d'export / import_ demande à résoudre les points suivants:
- comment P1 connaît-elle P2, est-ce que P1 a accès à un transport (e-mail, SMS, ...) par lequel P2 est joignable ?
- quelle sécurité cet échange a-t-il, peut-il être _écouté / intercepté_ au milieu, comment s'assurer que seule P2 pourra en faire usage ?

L'application KeyRing propose un protocole sécurisé à cet effet, pour autant que,
- P1 et P2 y soient enregistrés,
- que P1 y connaisse un identifiant de P2.

> Il n'y a pas de _solution générale_ à l'identification des _personnes_ dans une application, certaines pouvant d'ailleurs explicitement exclure toute référence _personnalisé_ dans le _vrai_ monde. KeyRing suppose que P1 connaît UNE identification textuelle de P2 joignable a minima temporairement sous ce _texte_. 

