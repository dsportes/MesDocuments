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
  - elle est stockée _cryptée par la clé K_ `p0_K`.
  - `p0_H` est le SHA du `SH(p0, p0)`. Identifiant du _safe_.
- une _phrase_ `p1`:
  - elle pourra être changée,
  - elle est stockée _cryptée par la clé K_ `p1_K`.
  - `p1_H` est le SHA du `SH(p0, p1)`. Index unique du _safe_.
- une _phrase_ `p2`:
  - elle pourra être changée,
  - elle est stockée _cryptée par la clé K_ `p2_K`.
  - `p2_H` est le SHA du `SH(p0, p2)`. Index unique du _safe_.
- un pseudo court comme `Alice`:
  - il ne pourra pas être changé,
  - il est stocké _crypté par la clé K_ `pseudo_K`

Les _phrases_ sont _longues_ d'au moins 16 signes pour p0 et 24 pour p1 et p2. Pour accéder à son _safe_ après création, son propriétaire devra fournir:
- sa phrase p0,
- l'une de ses deux phrases p1 OU p2.

La clé `K` du safe est stockée en base en `K1` et `K2` cryptages respectifs par  `SH(p0, p1, SEP)` et `SH(p0, p2, SEP)`.

Après avoir identifié son _safe_ son propriétaire peut:
- voir _en clair_ ses phrase `p0, p1, p2` et son `pseudo` court.
- changer `p1` et / ou `p2`.

#### Remarques:
- l'existence de deux phrases `p1` et `p2` autorise l'amnésie d'une des deux. 
- en revanche le propriétaire ne doit pas oublier `p0`: rien ne l'empêche d'y mettre un identifiant usuel comme une adresse e-mail, son état civil, etc. Ce texte N'EST JAMAIS exposé extérieurement et même si un hacker essayait la bonne phrase en tablant sur la banalité d'une adresse email, il devra n'avoir aucune chance de trouver `p1` ou `p2` avant la fin du monde.
- le _pseudo_ ne sert à l'utilisateur qui aurait déclarer un _device_ de confiance d'y être repéré parmi les quelques autres utilisateurs ayant aussi déclaré ce même _device_ comme étant aussi _de confiance pour eux_. Le pseudo s'affiche toujours suivi des quelques premières lettres de l'identifiant du _safe_ (le _hash fort_ de `p0`).

### Structure du document _Safe_
Il comporte les parties suivantes:
- **entête** : ce sont les propriétés décrites ci-avant:
  - `p0_K p0_H p1_K p1_H p2_K p2_H pseudo_K K1 K2`
- **droits des applications**. C'est une map avec une entrée par _code d'application_ donnant une _map de droits_. 
  - chaque _valeur_ est un objet ayant les propriétés `{appId, type, about, target, S}`. Cette valeur est cryptée,
    - par la clé publique de cryptage de l'application,
    - puis par la clé K du _safe_.
  - la clé est un hash de `appId, type, target`.
- **devices de confiance**: cette section est décrite ci-après.

### Opérations du module `Safe` serveur

#### Création d'un nouveau _Safe_
Arguments:
- `K1, K2` 
- `SH(p0, p0), SH(p0, p1), SH(p0, p2)`
- `p0_K, p1_K, p2_K, pseudo_K`

Exceptions:
- `p0` déjà utilisée pour un Safe existant.

#### Suppression d'un _Safe_
Arguments:
- `SH(p0, p1 OU p2)`

Exceptions:
- Safe inexistant

#### Mise à jour dun _Safe_ identifié par `SH(p0, p1 OU p2)`
- créer, modifier, supprimer UN droit d'une application.
- supprimer tous les droits d'une application.

#### Droits d'un _Safe identifié par `SH(p0, p1 OU p2)`
- retourner UN ou tous les droits d'une application.

> Le module _Safe_ serveur N'A JAMAIS ACCÈS à aucune aux clés de cryptage _en clair_: c'est un module de _stockage opaque_ incapable d'interpréter son contenu.

> Le module _Safe_ terminal est en charge des cryptages / décryptages et des interprétations: il est disponible en _source_ dans un browser en exécution et la validité de son source est vérifiable publiquement.

## Appareils _de confiance_ d'un utilisateur
Un utilisateur qui veut utiliser une application depuis un _device_ est placé devant deux cas de figure:
- **soit il juge l'appareil _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - il peut y laisser quelques informations cryptées et espérer raisonnablement les retrouver plus tard.
- **soit il n'a pas confiance dans cet appareil** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver des données.

Au lancement d'une application il doit être en mesure de lui communiquer ses droits d'accès, en particulier lui donner accès à son _safe_.
- il peut toujours le faire en saisissant ses phrases `p0` et `p1` ou `p2`.
- si c'est un appareil _de confiance_ il peut le faire en fournissant un simple `code PIN` (d'au moins 8 signes) ce qui est beaucoup plus rapide.

Pour un utilisateur lancer une application depuis un appareil _de confiance_ a plusieurs autres avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant une petite base de données locale (cryptée) comme _cache_ de documents de l'application: ceux connus et à jour n'auront pas besoin d'être demandés au serveur.
- **possibilité d'accéder à l'application en mode _avion_** sans accès au réseau en utilisant les documents et les droits en _cache_.

> Même _de confiance_ un appareil _peut_ être utilisé par d'autres que soi-même, même dans un cadre familial ou de couple, l'appareil n'est pas strictement _personnel_.

#### Accès au _Safe_ de l'utilisateur
Le _login_ à un appareil de confiance étant _protégé_ par un mot de passe connu seulement de quelques personnes de confiance (ou seulement soi-même), l'accès à un _Safe_ d'un utilisateur peut être allégé:
- l'utilisateur peut se désigner lui-même dans la la courte liste des _pseudos_ des utilisateurs ayant déclaré cet appareil de confiance.
- l'authentification par un code PIN court (d'au moins 8 signes) est jugée suffisante,
  - parce que le _login_ de l'appareil a, normalement, déjà écarté l'essentiels des personnes indésirables,
  - parce que le droit à l'erreur sur la saisie du code PIN est limité à 1 ou 2 échecs: au delà l'authentification par code PIN est invalidée et le couple `p0` et (`p1` ou `p2`) devient requis.

### Storage local d'un appareil _de confiance_
Certaines données sont mémorisées dans le _localStorage_ de l'appareil pour chaque utilisateur ayant déclaré l'appareil de confiance:
- un item de clé `$Ktux...@Bob` contient un objet avec les données spécifiques au _Safe_ dont l'identifiant est `Ktux...` et dont le `pseudo` est `Bob`.
- pour chacune des applications `app1` ayant été accédée par Bob, une base de données IDB nommée `$Ktux...@appi.idb` est la mémoire _cache_ (cryptée par la clé K du _Safe_) des documents de l'application `app1` pour `Bob`.

> En _debug_ il est simple d'effacer ces données sélectivement: en revanche la lecture des contenus des items et des bases IDB est plus problématique. Les données sont soit codées, soit cryptées.

#### Structure d'un item `$Ktux...@Bob` du _localStorage_
C'est un objet ayant les propriétés suivantes:
- `devId` : code aléatoire long identifiant le device pour le _Safe_ de `Bob`.
- `K1 K2 Kp` : cryptages de la clé `K` du _Safe_ respectivement par `SH(p0, p1, SEP)`, `SH(p0, p2, SEP)` et `SH(PIN, cy + cz, SEP)`.
- `cx` : challenge x aléatoire généré à la déclaration de confiance.
- `cy` : challenge y aléatoire généré à la déclaration de confiance.

En désérialisant un tel item (ce qui est techniquement simple), un hacker n'obtient rien d'utilisable. Il ne peut pas obtenir la clé `K` du _Safe_ sans connaître `p0` et (`p1` ou `p2`), ou `PIN` et `cz`:
- à la limite il peut _deviner_ `p0`, mais ni `p1` ni `p2` ne sont accessibles par force brute en raison de leur longueur.
- il peut tenter par force brute de cracker `PIN` (c'est déjà long), mais `cz` est très long et non accessible par force brute.

### Section _devices de confiance_ d'un _Safe_
Cet objet a les propriétés suivantes:
- `ldev` : liste des appareils déclarés _de confiance_ pour ce _Safe_. Couples [devId, about] ou about est un commentaire parlant pour le propriétaire du _Safe_. Par exemple `['QxtU...', 'mobile de Alice']`
  - cette liste facilite la suppression d'un appareil de confiance par le propriétaire du _Safe_.
- `cx` : challenge x aléatoire généré à la déclaration de confiance.
- `cz` : challenge z aléatoire généré à la déclaration de confiance.
- `sign` : signature par une clé `Sa` de la chaîne `sha(PIN + cy)`.
- `Va` : clé de validation correspondante au `Sa` utilisé ci-dessus.
- `Kp` : clé `K` du _Safe_ cryptée par `SH(PIN, cy + cz, SEP)`.
- `nbe` : nombre d'échecs de proposition de code PIN.

#### Accès au _Safe_ par code PIN depuis un appareil déjà déclaré de confiance par Bob
L'objectif est d'obtenir l'accès au _Safe_ de Bob à l'ouverture d'une application.
- accès à l'item `$Ktux...@Bob`: l'utilisateur a sélectionné `Bob` dans la liste proposée ce qui lui donne `$Ktux...` l'identifiant de son _Safe_.
- l'utilisateur saisit un PIN et le module _Safe_ terminal interroge le serveur (module _Safe_) en lui donnant en paramètres:
  - `appi` le code de l'application.
  - `devId` le code de l'appareil généré à sa déclaration de confiance.
  - `$Ktux...` l'identifiant de son _Safe_.
  - `cx` challenge x trouvé dans l'item (_version_ de la déclaration du code PIN).
  - `sha(PIN + cy)` où cy est le challenge y trouvé dans l'item et PIN le code PIN saisi par l'utilisateur.
- le module _Safe_ du serveur:
  - accède au _Safe_ pour l'identifiant `$Ktux...`.
  - vérifie la signature `sign` du _challenge_ `sha(PIN + cy)` en utilisant la clé de vérification `Va`.
  - en cas de succès, il met à 0 nbe s'il ne l'tait pas déjà et retourne `cz, K1, K2, Kp `et la liste des _droits_ pour l'application de code `appi`.

Le module Safe de l'application terminale peut décrypter `Kp` et en obtenir la clé `K` du _Safe_ de `Bob`: il a le code `PIN`, `cy` dans l'item lu du _localStorage_ et `cz` retourné par le serveur.
- **il peut décrypter les droits reçus**, chacun étant doublement crypté par,
  - la clé `K` qu'il vient d'obtenir,
  - la clé de cryptage de `appi` dont l'application a dans son code la clé privée de décryptage.

##### Échecs
- (1) si le devId reçu du _Safe_ terminal n'est pas dans la liste détenue dans le _Safe_ serveur, c'est que cet appareil N'EST PAS / PLUS de confiance.
- (2) si le challenge cx détenu en localStorage ne correspond pas au cx enregistré dans le _Safe_ le module _Safe_ serveur considère le localStorage de l'appareil pour Bob est basé sur une déclaration _ancienne_ du code PIN.
- (3) si la signature sign n'est pas vérifiée par Va, c'est que le code PIN a été redéfini (Va ne correspond plus au Sa qui a été utilisé à sa signature), OU que le code PIN fourni n'est pas le bon. Le nombre d'erreurs nbe est incrémenté.
  - si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais. Les données de la section _appareils de confiance_ sont supprimées à l'exception de la liste ldev. L'utilisateur devra redéclarer un code PIN (ce qui exigera une authentification _forte_ par p0 et (p1 ou p2)).

En cas de réussite, le nombre d'échecs nbe est remis à 0 s'il ne l'était pas.

### Remarques sur la _sécurité_ du protocole
- le code PIN n'est jamais passé en clair au module _Safe_ serveur: il ne peut pas être détourné ou être lu depuis la base de données.
- pour tenter depuis les données du _Safe_ serveur d'obtenir le code PIN par force brute, il faut effectuer une vérification de `sha(PIN + cy)`.

**BUG** : cy est lisible dans un localStorage.

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

