---
layout: page
title: Le module "Safe" et la gestion des "clé d'accès"
---

# Gestion des _credentials_ dans les applications

Une application `app1` met en jeu deux logiciels:
- `app1Srv` s'exécute sur un pool de serveurs au service de l'application gérant ses données centrales persistantes, ses opérations de mise à jour et ses extractions / synchronisations de données.
- `app1App` l'application terminale s'exécutant typiquement dans un browser et Web dont le source est lisible et délivré par un serveur statique / CDN.

Les opérations exécutées par le serveur comme les données qu'il peut retourner à l'application qui l'a sollicité, sont soumises, sauf exception, à des **droits d'accès**: d'une manière ou d'une autre l'utilisateur derrière l'application terminale doit exhiber qu'il possède effectivement les _droits requis_ par une opération pour la solliciter afin de mettre à jour et / ou obtenir des données centrales. Par exemple:
- `cpt` : droit à lire les documents d'un compte,
- `mbr` : droit de gestion des accès des membres d'un groupe,
- `trf` : droit de modification tarifaire ...

### Droit
Un _droit_ est matérialisé par les données suivantes:
- `type`, un code correspondant à sa _classe / catégorie_ `cpt, mbr, trf ...`
- `target` : l'identifiant (géré par l'application) de sa _cible_ qui peut comporter aussi bien des données lisibles (une adresse e-mail, un numéro de mobile ...) qu'être le résultat d'une génération aléatoire. Par exemple pour un droit `cpt` l'identifiant du compte.
- une liste de clés, en général d'un seul terme:
  - `variant` : un code (vide par défaut) permettant de caractériser plusieurs clés pour un même droit identifié par `type / target`.
  - `S` : clé privée de **signature** (environ 400 bytes).
  - `V` : clé publique de **vérification** (environ 100 bytes).

Le serveur d'une application gère les _credentials_ mais n'en connaît **QUE** `type target V` et n'a pas (sauf exception décrite ci-dessous) accès à le clé `S` correspondante. Le _serveur_ vérifie la validité d'un droit exhibé par _l'application terminale_ de la manière suivante:
- l'application terminale génère un texte _challenge_ qui n'a jamais été généré et ne sera jamais plus présenté.
- elle _signe_ ce _challenge_ par sa clé S et transmet au serveur le couple du challenge et de sa signature.
- le serveur utilise sa clé V correspondante à la clé S utilisée à la signature et peut vérifier que la signature reçue est bien celle du challenge transmis.

> Cette technique permet au serveur de s'assurer de la validité d'un droit sans jamais avoir eu en mémoire la clé S de signature: c'est un avantage déterminant de confiance par rapport aux solutions basées sur un mot de passe qui un moment ou à un autre a besoin d'être présent dans la mémoire du serveur. Même _haché_ le mot de passe permet une réutilisation éventuellement frauduleuse.

### Jetons d'accès
Quand l'application terminale soumet une opération au serveur, elle fournit dans sa requête un **jeton d'accès** qui réunit les preuves que son utilisateur dispose des _droits_ requis pour exécuter cette opération. Un jeton comporte:
- `devAppToken` : C'est 'identification d'UNE exécution l'application terminale sur UN _device_, à un instant donné une seule exécution terminale de l'application peut s'en prévaloir.
- `time` : date-heure en milliseconde de la génération du jeton d'accès. Cette donnée fait partie du _challenge_ des signatures du jeton.
- une liste de preuves de possession des droits constituée chacune d'un triplet:
  - `type` : du droit,
  - `target` : cible à laquelle le droit s'applique,
  - `signature` : signature du couple `devAppToken, time` par la clé `S` du droit correspondant.

**Remarques**:
- un _hacker_ un peu entraîné peut obtenir l'identifiant `devAppToken` en lançant en _debug_ l'application sur ce _device_.
- dans la liste des preuves, plusieurs pourraient concerner le même couple `type target`: il suffit que l'une d'elle soit reconnue comme valide pour que le droit le soit, une manière de traiter les situations, temporaires ou non, où un même droits peut avoir plusieurs _variantes_ de clés de signature / vérification (une _ancienne_ et une _nouvelle_ ...).
- un _jeton d'accès_ est crypté par la clé publique d'encryption du serveur applicatif ciblé de sorte seul celui-ci puisse le lire.

#### Solution simplifiée
Quand l'application terminale doit soumettre une opération requérant un _droit__ donné (par exemple `cpt` pour un compte `1234`), elle peut en obtenir la clé `S` par un module la demandant à l'utilisateur. 

Ce dernier effectue une forme ou une autre de _copier / coller_ depuis par exemple un fichier de texte externe détenu sur son _device_ et lui affichant sur une ligne la clé `S` pour le compte `1234`.

#### Solution par accès à un _safe_
Les _droits_ de l'utilisateur sont stockés dans une base de données dédiées en charge de conserver des _coffres forts / safes_. Chaque _safe_:
- est accessible par un utilisateur en ayant la preuve de propriété.
- dans ce _safe_, chaque application a sa collection _de clés_, chacun ayant les propriétés suivantes:
  - about : un texte libre permettant au propriétaire du _safe_ de se souvenir à quoi sert ce droit.
  - type du droit.
  - target du droit.
  - S : clé de signature (voire plusieurs clés séparées par un espace).

Dans la liste associée à une application, le couple type, target est identifiant.

En base de données, un droit est crypté,
- par la clé publique d'encryption de l'application afin qu'elle seule puisse le lire,
- par la clé de cryptage du _safe_ générée spécifiquement pour son propriétaire afin que la lecture directe de la base de données sans passer par le module _safe_ terminal soit impossible, lequel module devra avoir obtenu du propriétaire du _safe_ la clé pour en lire le contenu.

Quand l'application terminale doit soumettre une opération requérant un _droit_ donné (par exemple `cpt` pour un compte `1234`), elle peut en obtenir la clé `S` (ou liste de clés) en le demandant à son module _safe terminal_.

#### L'application terminale ne doit pas être _pirate_
Si l'application terminale est une application _pirate_ qui a été lancée en déjouant la vigilance de l'utilisateur (par exemple en le sollicitant d'appuyer sur un lien envoyé par un e-mail frauduleux), elle dispose des clés des droits auquel l'utilisateur lui a donné l'accès. 

Elle peut les envoyer sur un serveur pirate où elles seront à disposition de pirates pour utilisation dans la _vraie_ application et lui transmettre ces clés _usurpées_.

L'application _légitime_ a dans son code _sa clé de décryptage privée_ qui lui permet de décrypter les droits: mais en exécution en mode _debug_ un hacker un peu habile peut la retrouver. Cette _sécurité_ plus symbolique qu'effective.

Il n'existe aucun procédé logiciel qui permette de connaître l'origine d'une application, de quelle _source_ elle vient, si elle est _légitime_ ou _pirate_. Depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle de l'application pirate,
- avoir confiance dans le fait que le CDN distribue bien un code non piraté,
- avoir obtenu d'un expert indépendant l'assurance que ce code est bien légitime et ne redistribue pas les clés d'accès,
- avoir vérifié son certificat.

Ces conditions peuvent être réunies et sont vérifiables.

### Validation des _droits_ d'un jeton par le serveur de l'application
Quand le serveur de l'application traite une opération,
- il obtient de la requête le jeton d'accès et le décrypte par sa clé privée de décryptage.
- le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _droit_ de la liste du jeton,
  - il obtient depuis la base de données la clé `V` de vérification associée à sa cible,
  - il vérifie que la `signature` du couple `devAppToken, time` est bien validée par `V`, ce qui prouve que l'application terminale en détient effectivement la clé de signature `S` correspondante.

> `devAppToken, time` est utilisé comme _challenge_ cryptographique et n'est pas présenté plus d'une fois pour une application donnée.

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `target` d'un `type` de droit le dernier `V` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `V`, et cas de changement refait une vérification avant de valider / invalider le droit correspondant.
- le serveur conserve en cache pour chaque `devAppToken` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, leur _validation_ ne requiert qu'un calcul en mémoire sans accès à la base pour obtenir les clés de vérification.

## "Un" droit, "plusieurs" clés
Le serveur _peut_ mémoriser pour un droit non pas une clé `V` mais une liste de clés `[V1, v2 ...]`: pour être validé, un droit d'un jeton d'accès doit fournir une signature qui a été établie par **UNE DES** clés `Si` correspondantes.

La logique applicative peut ainsi prévoir de distribuer plusieurs _variantes_ de clés à des détenteurs différents selon ses propres critères: elle peut aussi au cours du temps en _désactiver_ certaines variantes dans le serveur et inhiber ainsi les détenteurs qui les possédaient.

Dans un jeton généré par l'application, s'il existait _plusieurs_ clés possibles, au lieu d'une signature pour un droit donné, c'est une liste de signatures (une par clé potentielle) qui est générée. Il suffit que l'une d'entre elles _matche_ avec la ou une des clés `Vi` de la liste détenue par le serveur pour que le droit soit validé.

> Ce dispositif permet aussi de gérer un _changement de clé_ pour un droit lorsqu'il a été attribué trop généreusement ou pour un temps limité. La création d'une nouvelle clé peut être faite et sa distribution restreinte aux seuls détenteurs souhaitables dans le futur: après un certain temps où les deux peuvent être admises, _l'ancienne_ peut être supprimée.

# Gestion par un utilisateur des droits qu'il a acquis
Quelqu'en soit le processus, à un moment donné un utilisateur a acquis un **droit** d'un type donné ce qui s'est manifesté par le fait qu'il connaisse le triplet `type, target, S` correspondant.
- L'identifiant `target` peut être abscons, difficile à mémoriser.
- `S` est _impossible_ à mémoriser, chacun étant constitué d'environ 400 lettres et chiffres d'apparence aléatoire.

## Fichier CSV des _droits_ d'un utilisateur
L'utilisateur doit a minima recourir à un fichier dans lequel il a enregistré ses droits. 

Cette liste étant en soi incompréhensible, pour chaque terme `type, target, S` l'utilisateur lui associe un _à propos_ parlant pour lui afin qu'il sache quelle ligne sélectionner quand l'application lui demandera un droit, par exemple `connexion de Bob`

Le fichier peut être un simple CSV ayant quatre colonnes: `application, type, about, target, S`.

> Remarque: `S` peut être en fait une liste de `S` séparés par un espace.

### Demande de droits d'une application
Lorsque l'application terminale souhaite avoir la clé S d'un _droit_, plusieurs situations se présentent:
- (1) pour le `type` fixé (par exemple `DRTARIF` : _droit à effectuer une modification tarifaire_), il n'existe qu'une clé unique, `target` est vide. Une ligne du fichier au plus est sélectionnable.
- (2) le `type` est fixé ET la valeur de `target` (par exemple `LOGIN, 1234` : _droit de l'utilisateur 1234 à se connecter_) est fixé par l'application. Une ligne du fichier au plus est sélectionnable.
- (3) le `type` est fixé MAIS PAS la valeur de `target` (par exemple `LOGIN`):  l'application recherche **à la fois une cible et sa clé**. Plusieurs lignes du fichier peuvent être sélectionnables et l'utilisateur devra **désigner** le login qu'il choisit: dans ce cas le commentaire `about` lui est utile (par exemple en donnant un nom en clair plutôt qu'un code).

L'application peut charger en mémoire le fichier CSV des credentials de l'utilisateur en lui demandant son _path_ sur le file-system local du _device_:
- dans les cas (1 2) l'application peut ensuite obtenir directement `S`,
- dans le cas (3) l'application peut présenter à l'utilisateur la liste des couples `about target` contenus dans le fichier. La sélection de l'utilisateur retourne le couple `target, S` correspondant à son choix.

> Remarque: le fichier CSV est une des solutions simplistes possibles. Une autre solution encore plus simpliste est que l'application présente une double zone de texte à saisir dans laquelle l'utilisateur _colle_ le couple `target, S` de son choix après l'avoir _copié_ de ... quelque part (un mail, un gestionnaire de clés ...).

# Les modules _safe_
La gestion d'un fichier CSV de ses droits par un utilisateur met en lumière le problème de sa gestion:
- le fichier CSV doit pouvoir être accessible depuis plusieurs _devices_, stocké sur le _cloud_ ...
- il doit être encrypté afin de ne pas exposer ses clés à des pirates et l'utilisateur doit se rappeler de sa clé de cryptage,
- la mise à jour du fichier suppose d'être vigilant: l'inscription d'un nouveau droit (voire le _changement_ d'une clé) est un risque de dégradation / perte du fichier et en conséquence de tous ses droits précieusement récupérés au fil du temps.

Pour résoudre ce problème, un couple de modules **safe**, un pour l'application terminale, un pour le serveur, propose à un utilisateur qui le souhaite une solution pour gérer avec une confidentialité rigoureuse un **coffre fort** où il peut stocker ses droits.

> _safe_ ne gère pas à proprement parler des _utilisateurs_ mais leurs coffres forts: rien n'empêche un _utilisateur_ (une personne) de posséder plus d'un coffre, mais absolument rien ne relient les coffres entre eux ni à un quelconque signifiant dans le monde réel.

## Organisation générale en base de données
La base de données a un _document_ par _safe_.

### Création d'un _safe_ par un utilisateur
Une clé AES `K` de 32 bytes est tirée aléatoirement: elle ne pourra pas changer.

L'utilisateur donne:
- une _phrase_ `p0` : un seul safe dans la base peut avoir cette phrase à un instant donné.
  - elle ne pourra plus être changée.
  - elle est stockée _cryptée par la clé K_ `p0_K` et son _hash fort_ `p0_SH` est l'identifiant du _safe_.
- une _phrase_ `p1`:
  - elle pourra être changée,
  - elle est stockée _cryptée par la clé K_ `p1_K` ainsi que son _hash fort_ `p1_SH`.
- une _phrase_ `p2`:
  - elle pourra être changée,
  - elle est stockée _cryptée par la clé K_ `p2_K` ainsi que son _hash fort_ `p2_SH`.
- un pseudo court comme `Alice`:
  - il ne pourra pas être changé,
  - il est stockée _crypté par la clé K_ `pseudo_K`

Les _phrases_ sont _longues_ d'au moins 16 signes pour p0 et 24 pour p1 et p2. Pour accéder à son _safe_ après création, son propriétaire devra fournir:
- sa phrase p0,
- l'une de ses deux phrases p1 OU p2.

La clé K du safe est stockée en base cryptée par (p0 + p1) K1 et par (p0 + p2) K2.

Après avoir identifier son _safe_ son propriétaire peut:
- voir _en clair_ ses phrase `p0, p1, p2` et son `pseudo` court.
- changer `p1` et / ou `p2`.

#### Remarques:
- l'existence de deux phrases `p1` et `p2` autorise l'amnésie d'une des deux. 
- en revanche le propriétaire ne doit pas oublier `p0`: rien ne l'empêche d'y mettre un identifiant usuel comme une adresse e-mail, son état civil, etc. Ce texte N'EST JAMAIS exposé extérieurement et même si un hacker essayait la bonne phrase en tablant sur la banalité d'une adresse email, il devra n'avoir aucune chance de trouver `p1` ou `p2` avant la fin du monde.
- le _pseudo_ ne sert à l'utilisateur qui aurait déclarer un _device_ de confiance d'y être repéré parmi les quelques autres utilisateurs ayant aussi déclaré ce même _device_ comme étant aussi _de confiance pour eux_. Le pseudo s'affiche toujours suivi des quelques premières lettres de l'identifiant du _safe_ (le _hash fort_ de `p0`).

### Structure du document _Safe_
Il comporte les parties suivantes:
- **entête** : ce sont les propriétés décrites ci-avant:
  - `p0_K p0_SH p1_K p1_SH p2_K p2_SH pseudo_K K1 K2`
- **droits des applications**. C'est une map avec une entrée par _code d'application_ donnant une _map de droits_. 
  - chaque _valeur_ est un objet ayant les propriétés `{appId, type, about, target, S}`. Cette valeur est cryptée,
    - par la clé publique de cryptage de l'application,
    - puis par la clé K du _safe_.
  - la clé est un hash de `appId, type, target`.
- **devises favoris**: cette section est décrite ci-après.

Cette organisation permet au module `Safe` serveur:
- de mettre à jour (créer, modifier, supprimer) UN droit d'une application.
- de retourner UN ou tous les droits d'une application.
- de supprimer une application complète.

> Le module _Safe_ serveur N'A JAMAIS ACCÈS à aucune des divers clés de cryptage _en clair_: c'est un module de _pur stockage opaque_ incapable d'interpréter son contenu.

> Le module _Safe_ terminal est en charge des cryptages / décryptages et d'interprétation: il est disponible en _source_ dans un browser en exécution et la non délinquance de son source est vérifiable publiquement.

### Devices _favoris_ d'un utilisateur

**TODO**

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

