---
layout: page
title: L'application "Safe", la gestion des "droits d'accès"
---

# Gestion des _credentials / droits d'accès_ dans les applications

Une application `app1` met en jeu deux logiciels:
- `app1Srv` s'exécute sur un pool de serveurs au service de l'application gérant ses données centrales persistantes, ses opérations de mise à jour et ses extractions / synchronisations de données.
- `app1App` l'application terminale s'exécutant typiquement dans un browser et Web dont le source est lisible et délivré par un serveur statique / CDN.

Les opérations exécutées par le serveur comme les données qu'il peut retourner à l'application qui l'a sollicité, sont soumises, sauf exception, à des **droits d'accès**: d'une manière ou d'une autre l'utilisateur derrière l'application terminale doit prouver qu'il possède effectivement les _droits requis_ pour solliciter une opération afin de mettre à jour et / ou obtenir des données centrales. Par exemple:
- `cpt` : droit à lire les documents d'un compte,
- `mbr` : droit de gestion des accès des membres d'un groupe,
- `trf` : droit de modification tarifaire ...

### Droit d'une application
Un _droit_ d'une application est matérialisé par les données suivantes:
- `type`, un code correspondant à sa _classe / catégorie_ `cpt, mbr, trf ...`
- `target` : l'identifiant dans l'application de sa _cible_ qui peut comporter aussi bien des données lisibles (une adresse e-mail, un numéro de mobile ...) qu'être le résultat d'une génération aléatoire. Par exemple pour un droit `cpt` l'identifiant du compte.
- une _liste de clés_, souvent d'un seul terme:
  - `variant` : un code (vide par défaut) permettant de caractériser plusieurs clés pour un même droit identifié par `type / target`.
  - `S` : clé privée de **signature** (environ 400 bytes).
  - `V` : clé publique de **vérification** (environ 100 bytes).

Le serveur d'une application gère les _droits d'accès_ mais n'en connaît **QUE** `type target V` et n'a pas (sauf exception décrite ci-dessous) accès à le clé `S` correspondante. Le _serveur_ vérifie la validité d'un droit transmis par _l'application terminale_ de la manière suivante:
- l'application terminale génère un texte _challenge_ qui n'a jamais été généré et ne sera jamais plus présenté à l'application serveur.
- elle _signe_ ce _challenge_ par sa clé `S` et transmet au serveur le couple du challenge et de sa signature.
- le serveur utilise sa clé `V` correspondante à la clé `S` utilisée à la signature et peut vérifier que la signature reçue est bien celle du challenge transmis.

> Cette technique permet au serveur de s'assurer de la validité d'un droit sans jamais avoir eu en mémoire la clé `S` de signature: c'est un avantage de confiance par rapport aux solutions basées sur un mot de passe qui, à un moment ou à un autre, a besoin d'être présent dans la mémoire du serveur, même si un hachage fort des mots de passe limite le risque.

### Jetons d'accès
Quand l'application terminale soumet une opération au serveur, elle fournit dans sa requête un **jeton d'accès** qui réunit les preuves que son utilisateur dispose des _droits_ requis pour exécuter cette opération. Un jeton comporte:
- `devAppToken` : C'est 'identification d'UNE exécution l'application terminale sur UN _device_, à un instant donné une seule exécution terminale de l'application peut s'en prévaloir.
- `time` : date-heure en milliseconde de la génération du jeton d'accès. Cette donnée fait partie du _challenge_ des signatures du jeton.
- une liste de preuves de possession des droits constituée chacune d'un triplet:
  - `type` : du droit,
  - `target` : cible à laquelle le droit s'applique,
  - `signature` : signature du couple `hash(devAppToken), time` par la clé `S` du droit correspondant.

**Remarques**:
- un _hacker_ un peu entraîné peut obtenir l'identifiant `devAppToken` en lançant en _debug_ l'application sur ce _device_.
- dans la liste des preuves, plusieurs peuvent concerner le même couple `type target`: il suffit que l'une d'elle soit reconnue comme valide pour que le droit le soit. C'est une manière de traiter les situations, temporaires ou non, où un même droit peut avoir plusieurs _variantes_ de clés de signature / vérification (une _ancienne_ et une _nouvelle_, une variante par _pseudo_ ...).
- un _jeton d'accès_ est crypté par la clé publique d'encryption du serveur applicatif ciblé de sorte que seul celui-ci puisse le lire. Cette clé fait partie de la _configuration_ du serveur que son administrateur technique délivre lors du déploiement et qu'il doit conserver confidentiellement.

#### Solutions: simpliste versus _Safe_
Quand l'application terminale doit soumettre une opération requérant un _droit_ donné (par exemple `cpt` pour un compte `1234`), elle pourrait en obtenir la clé `S` par un module la demandant à l'utilisateur. 

Ce dernier effectue une forme ou une autre de _copier / coller_ depuis par exemple un fichier de texte externe détenu sur son _device_ et lui affichant sur une ligne la clé `S` pour le compte `1234`.

Cette solution _simpliste_ est difficile à gérer dans la pratique:
- comment l'utilisateur gère la disponibilité sur plusieurs appareils ?
- risque d'écrasement partiel ou perte du fichier lors d'une mise à jour ?
- comment _sécuriser_ un tel fichier contre le vol ?

Un autre solution est de faire gérer le stockage de ces _droits_ par une application _Safe_ en charge de résoudre les questions ci-dessus et de rendre accessible les _droits_ aux applications qu'ils concernent.

#### Les applications terminales ne doivent pas être _piratées_
Si l'application terminale est une application _pirate_ qui a été lancée en déjouant la vigilance de l'utilisateur (par exemple en le sollicitant d'appuyer sur un lien envoyé par un e-mail frauduleux), elle dispose des clés des droits auquel l'utilisateur lui a donné l'accès. 

Elle peut les envoyer sur un serveur pirate où elles seront à disposition de pirates pour utilisation dans la _vraie_ application et lui transmettre ces clés _usurpées_.

L'application _légitime_ a dans son code _sa clé de décryptage privée_ qui lui permet de décrypter les droits: mais en exécution en mode _debug_ un hacker un peu habile peut la retrouver. Cette _sécurité_ plus symbolique qu'effective.

Il n'existe aucun procédé logiciel qui permette de connaître l'origine d'une application, de quelle _source_ elle vient, si elle est _légitime_ ou _pirate_. Depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle de l'application pirate,
- avoir confiance dans le fait que le CDN distribue bien un code non piraté,
- avoir obtenu d'un expert indépendant l'assurance que ce code est bien légitime et ne redistribue pas les clés d'accès,
- avoir vérifié son certificat.

L'utilisation de l'application _Safe_ pour lancer les applications _légitimes_ sans risque d'usage d'une URL _pirate_ réduit les risques: l'utilisateur n'a plus qu'é s'assurer qu'une seule application (8safe_) est légitime au lieu de surveiller _toutes_ les URLs des applications.

### Demandes de droits par une application terminale
En supposant que l'application terminale ait obtenu, en sécurité, la **liste des droits** de l'utilisateur, lorsque celle-ci recherche la clé `S` d'un _droit_, plusieurs situations se présentent:
- (1) pour le `type` fixé (par exemple `DRTARIF` : _droit à effectuer une modification tarifaire_), il n'existe qu'une clé unique, `target` est vide. Un seul terme de la liste des droits peut s'appliquer.
- (2) le `type` est fixé ET la valeur de `target` (par exemple `LOGIN, 1234` : _droit de l'utilisateur 1234 à se connecter_) est fixé par l'application. Une seul terme _au plus_ de la liste des droits peut s'appliquer.
- (3) le `type` est fixé MAIS PAS la valeur de `target` (par exemple `LOGIN`):  l'application recherche **à la fois une cible et sa clé**. Plusieurs droits de la liste peuvent être candidats et l'utilisateur devra **désigner** le login qu'il choisit: dans ce cas un commentaire `about` attaché à chaque droit lui est utile (par exemple en donnant un nom en clair plutôt qu'un code).

### Validation des _droits_ d'un jeton par le serveur de l'application
Quand le serveur de l'application traite une opération,
- il obtient de la requête le jeton d'accès et le décrypte par sa clé privée de décryptage.
- le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _droit_ de la liste du jeton,
  - il obtient depuis la base de données la clé `V` de vérification associée à sa cible,
  - il vérifie que la `signature` du couple `hash(devAppToken), time` est bien validée par `V`, ce qui prouve que l'application terminale en détient effectivement la clé de signature `S` correspondante.

> `hash(devAppToken), time` est utilisé comme _challenge_ cryptographique et n'est pas présenté plus d'une fois pour une application donnée.

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `target` d'un `type` de droit le dernier `V` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `V`, et cas de changement refait une vérification avant de valider / invalider le droit correspondant.
- le serveur conserve en cache pour chaque `hash(devAppToken)` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, leur _validation_ ne requiert qu'un calcul en mémoire sans accès à la base pour obtenir les clés de vérification.

### "Un" droit, "plusieurs" clés
Le serveur _peut_ mémoriser pour un droit non pas une clé `V` mais une liste de clés `[V1, v2 ...]`: pour être validé, un droit d'un jeton d'accès doit fournir une signature qui a été établie par **UNE DES** clés `Si` correspondantes.

La logique applicative peut ainsi prévoir de distribuer plusieurs _variantes_ de clés à des détenteurs différents selon ses propres critères (par exemple un _pseudo court_ du détenteur): elle peut aussi au cours du temps en _désactiver_ certaines variantes dans le serveur et inhiber ainsi les détenteurs qui les possédaient.

Dans un jeton généré par l'application, s'il existe pour un droit donné _plusieurs_ clés possibles, au lieu d'une signature c'est une liste de signatures (une par clé potentielle) qui est générée. Il suffit que l'une d'entre elles _matche_ avec la ou une des clés `Vi` de la liste détenue par le serveur pour que le droit soit validé.

> Ce dispositif permet aussi de gérer un _changement de clé_ pour un droit lorsqu'il a été attribué trop généreusement ou pour un temps limité. La création d'une nouvelle clé peut être faite et sa distribution restreinte aux seuls détenteurs souhaitables dans le futur: après un certain temps où les deux peuvent être admises, _l'ancienne_ peut être supprimée.

# L'application _Safe_

Cette application a pour objet de gérer des _coffres forts_ pour des utilisateurs.

Après avoir lancé l'application _Safe terminale_ depuis son appareil, un utilisateur lui indique quel est son _coffre fort_ de manière à ce que les applications lancées ultérieurement sur cet appareil puissent y trouver diverses données _sensibles_ de l'utilisateur dont ses _droits d'accès_.

Une fois ainsi initialisée, l'application _Safe terminale_ peut:
- gérer ses appareils _de confiance_.
- gérer la liste des applications auxquelles l'utilisateur peut accéder,
- lancer une de celles-ci.

Pour chaque application lancée, l'application _Safe terminale_ peut:
- stocker `ses droits` et les mettre à jour.
- stocker _divers objets_, dont des objets transmis par un autre utilisateur disposant lui aussi d'un _safe_. Certains de ces objets peuvent être interprétés par l'application comme des _préférences / options_.

> Le lancement d'applications par le _Safe_ évite le risque de lancement d'une application _piratée_ et permet de choisir le cas échéant une des des _options_ au lancement ouvrant la session dans un contexte déjà pré-fixé.

> L'URL de lancement de _Safe_ doit être soigneusement vérifiée: le code de cette application est lisible dans le browser et il est possible de vérifier auprès de sites certificateurs que l'application est _fair_. Ceci évite d'avoir à le faire pour chaque application gérée par _Safe_, quoi que ce soit toujours possible.

## Stockage des _safes_

### Le serveur _Safe_
Ce serveur et sa base de données sont le lieu de stockage _par défaut_ des _safes_.

L'accès par l'application terminale à un _safe_ se fait par deux procédés:
- la donnée du SH(p0, (p1 ou p2)) où p0 p1 p2 sont des _phrases_ longues connues uniquement du détenteur du _safe_.
- la donnée d'un triplet `(devName, E, cx SH(PIN, cy))` si l'appareil est _de confiance_, où:
  - `devName` : nom identifiant le device pour le _Safe_.
  - `E` est l'identifiant du _safe_,
  - `cx` et `cy` deux challenges obtenus lors de la déclaration de confiance d'appareil,
  - `PIN` est un code PIN court (au moins 8 signes) enregistré à la déclaration de confiance de l'appareil.
  - `devName, E, cx cy` étant mémorisé sur l'appareil _de confiance_ lui-même lors de la déclaration de confiance, E peut être cliqué dans une liste courte et PIN doit être saisi.

Après avoir identifié en sécurité le _safe_ de l'utilisateur, l'application terminale _Safe_ peut disposer du contenu du _safe_ en mémoire et communiquer les données souhaitées aux applications lancées sur l'appareil.

La mise à jour du _safe_ se répercute directement sur le serveur.

### Un _fichier_ JSON crypté
Au lieu d'obtenir le contenu du _safe_ par l'application _Safe serveur_, l'application _Safe terminale_ peut demander à l'utilisateur de lui fournir un fichier `safe.json` (gzippé et crypté).

Le cryptage est assurée par une _phrase longue_ connue de l'utilisateur.

Après lecture et décryptage, l'application fonctionne comme avec son serveur avec les différences suivantes:
- les mises à jour sont cumulées en mémoire mais pas transmises au serveur.
- c'est à l'utilisateur de demander la _sauvegarde_ (après compression et cryptage) de ce fichier et de gérer sa communication éventuelle par le cloud ou des clés USB ...

> L'application _Safe_ ne gère pas à proprement parler des _utilisateurs_ mais leurs coffres forts: rien n'empêche un _utilisateur_ (une personne) de posséder plus d'un coffre, rien ne relient les coffres entre eux ni à un quelconque signifiant dans le monde réel.

## Organisation générale en base de données
La base de données a un _document_ par _safe_.

### Création d'un _safe_ par un utilisateur
Une clé AES `K` de 32 bytes est tirée aléatoirement: elle ne pourra pas changer.

Une couple de clés `C / D` asymétriques:
- `C` est en clair : c'est l'identifiant du _safe_
- `D_K` est le cryptage de `D` par `K`.
- `C_KH` est le SHA du cryptage de `C` par `K`. Cette donnée permet au _safe_ serveur de s'assurer qu'une requête a été issue d'une application terminale détenant la clé `K`.

L'utilisateur donne:
- une _phrase_ `p0` qui de facto ne pourra plus être changée.
- une _phrase_ `p1`:
  - elle pourra être changée,
  - `p1_H` est le SHA du `SH(p0, p1)`. Index unique du _safe_.
- une _phrase_ `p2`:
  - elle pourra être changée,
  - `p2_H` est le SHA du `SH(p0, p2)`. Index unique du _safe_.
- un pseudo court comme `Alice` facultatif à ce moment et pourra être donné plus tard lors de la première déclaration d'un appareil de confiance.
  - il est immuable, ne pourra plus être changé après avoir été fixé,
  - il est stocké _crypté par la clé K_ `pseudo_K`.

Les _phrases_ sont _longues_ d'au moins 16 signes pour `p0` et 24 pour `p1` et `p2`. Pour accéder à son _safe_ après création, son propriétaire doit fournir:
- sa phrase `p0`,
- l'une de ses deux phrases `p1` OU `p2`.

La clé `K` du safe est stockée en base en `K1` et `K2` cryptages respectifs par  `SH(p0, p1, SEP)` et `SH(p0, p2, SEP)`.

Après avoir identifié son _safe_ son propriétaire peut:
- voir _en clair_ son `pseudo` court.
- changer `p1` ou `p2` à condition de fournir `p1` ou `p2` actuel.
- de facto il n'est pas possible de changer p0, bien que non stocké dans le _safe_ sous aucune forme.

> A aucun moment `p0 p1 p2` ne sont stockés nulle part ni transmis _en clair_. Ils ne figurent _en clair_ que très temporairement à la saisie par l'utilisateur dans le module _Safe_ terminal et y sont effacés dès la fin de la saisie.

#### Remarques:
- l'existence de deux phrases `p1` et `p2` autorise l'oubli de l'une des deux. 
- en revanche le propriétaire ne doit pas oublier `p0`: il peut donner une adresse _e-mail, son état civil, etc._ Ce texte N'EST JAMAIS exposé extérieurement et même si un hacker essayait la bonne phrase en tablant sur la banalité d'une adresse email, il n'aurait aucune chance de trouver `p1` ou `p2` avant la fin du monde (si un minimum de règles de choix est respecté).
- le _pseudo_ sert à l'utilisateur déclarant un _device_ de confiance d'y être repéré parmi les quelques autres utilisateurs ayant déclaré ce même _device_ comme étant aussi _de confiance pour eux_. Le pseudo s'affiche suivi du début de l'identifiant du _safe_.

### Structure du document _Safe_
Il comporte les parties suivantes:
- **entête** : ce sont les propriétés décrites ci-avant:
  - `E D_K C_KH K1 K2 p1_H p2_H pseudo_K`
  - `lastAccess` : numéro du dernier mois d'utilisation du _safe_ afin de pouvoir procéder périodiquement à une _purge_ des _safe_ obsolètes / fantômes.
- **droits des applications**. C'est une map avec une entrée par _code d'application_ donnant une _map de droits_. 
  - chaque _valeur_ est un objet ayant les propriétés `{appId, type, about, target, S}`. Cette valeur est cryptée,
    - par la clé publique de cryptage de l'application,
    - puis par la clé K du _safe_.
  - la clé est un hash de `appId, type, target`.
- **devices de confiance**: cette section est décrite ci-après.
- **objets reçus** : pour chaque application, c'est une map d'items de clés `hash(obj)` et de valeurs `{ exp_C, obj }` où,
  - `exp_C` est la clé publique de cryptage de l'expéditeur.
  - `type` : code du _type_ de l'objet (par exemple `DROIT`).
  - `obj` est un objet crypté par la clé `C` du _safe_ (son identifiant).
    - `obj` a une propriété `about` donnant une explication _humaine_ à propos de l'objet transmis.
- **liste noire**: c'est une map d'items:
  - _clé_: hash de la clé publique d'un _safe_ dont le _safe_ n'accepte plus de recevoir d'objets.
  - _valeur_: texte crypté par la clé K du _safe_ rappelant à son propriétaire qui il a mis en liste noire et pourquoi.

Le propriétaire d'un _safe_ A peut envoyer un _objet_ confidentiel au propriétaire d'un _safe_ B:
- la structure de l'objet dépend de son objectif, typiquement ce peut être,
  - un simple message textuel,
  - un _droit_ transmis de A à B.
- l'objet a toujours une propriété `about` fournissant un commentaire textuel court de A à l'intention de B.
- A peut s'envoyer un objet à lui-même, comme dans un _presse-papier_, confidentiel et persistant.

### Opérations du module `Safe` serveur

#### Création d'un nouveau _Safe_
Afin d'éviter une inflation incontrôlable de création de _safes_ fantômes, un utilisateur ne peut créer un _safe_ qu'après avoir reçu de la part d'une application un _code_ de validité limitée.

La base de données détient une liste des `SH(c)` des codes `c` en cours de validité avec leur date-heure limite de validité.

L'utilisateur est invité à saisir `code, p0, p1, p2, pseudo`.

Le module _Safe_ terminal génère les clé `K C D` et calcule les arguments suivants:
- `SH(code)` : code d'autorisation délivré par une application.
- `K1, K2, D_K, C_KH` 
- `SH(p0, p1), SH(p0, p2)`
- `pseudo_K`

Exceptions:
- `p0` déjà utilisée pour un Safe existant.

#### Suppression d'un _Safe_
Arguments:
- soit `SH(p0, p1 OU p2)`
- soit `E E_K` : l'identifiant du _safe_ et son cryptage par la clé K comme preuve de la détention de K acquis par exemple par le PIN.

Exceptions:
- Safe inexistant

#### Mise à jour / consultation d'un _Safe_
Le _safe_ est identifié, soit par `SH(p0, p1 OU p2)`, soit par son `E E_K`. Les actions possibles sont:
- lecture du pseudo.
- obtention d'UN ou de tous les droits d'une application (export).
- création, modification, suppression d'UN ou plusieurs droits d'une application (import).
- suppression de tous les droits d'une application.
- modification de `p1` ou `p2` en donnant la valeur actuelle de `p0` et (`p1` ou `p2`).

> Le module _Safe_ serveur N'A JAMAIS ACCÈS à aucune des clés de cryptage _en clair_: c'est un module de _stockage opaque_ incapable d'interpréter son contenu.

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
- il peut toujours le faire en saisissant ses phrases `p0` et, `p1` ou `p2`.
- si c'est un appareil _de confiance_ il peut le faire en saisissant seulement un `code PIN` (d'au moins 8 signes) ce qui est plus rapide. L'identifiant du _Safe_ est stocké localement dans l'appareil.

Pour un utilisateur lancer une application depuis un appareil _de confiance_ a plusieurs autres avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant une petite base de données locale (cryptée) pour chaque application comme _cache_ de ses document: ceux qui y figurent et à jour n'auront pas besoin d'être demandés au serveur de l'application.
- **disponibilité des droits** de l'utilisateur pour chaque application le dispensant de s'en souvenir ou de les copier / coller d'un support externe.
- **possibilité d'accéder à l'application en mode _avion_** sans accès au réseau en utilisant les documents et les droits en _cache_.

> Même _de confiance_ un appareil _peut_ être utilisé par d'autres que soi-même, même dans un cadre familial ou de couple, l'appareil n'est pas strictement _personnel_.

#### Accès au _Safe_ de l'utilisateur
Le _login_ à un appareil de confiance étant _protégé_ par un mot de passe théoriquement connu seulement de quelques personnes de confiance : empreinte digitale ou mot de passe sur un mobile, login / mot de passe sur un PC. Dans ce cas l'accès à son _Safe_ d'un utilisateur peut être allégé:
- il peut se désigner lui-même dans la la courte liste des _pseudos_ des utilisateurs ayant déclaré cet appareil de confiance ce qui désignera l'identifiant de son _safe_.
- l'authentification par un code PIN (d'au moins 8 signes) est alors jugée _suffisante_,
  - parce que le _login_ de l'appareil a, normalement, déjà écarté l'essentiel des personnes indésirables,
  - parce que **le droit à l'erreur sur la saisie du code PIN est limité à 1 ou 2 échecs**: au delà l'authentification par le couple `p0` et (`p1` ou `p2`) devient requis (le code PIN est invalidé dans le _safe_).

### Storage local d'un _safe_ dans un appareil _de confiance_
Le _localStorage_ de l'appareil été ayant déclaré _de confiance_ pour un _safe_ est un item de clé `Bob@Ktux...` contenant quelques données spécifiques du _Safe_: 
- son identifiant `E` est `Ktux...` et dont son `pseudo` est `Bob`.
- pour chacune des applications `app1` ayant été accédée par `Bob`, une base de données IDB nommée `$Ktux...@appi.idb` est la mémoire _cache_ (cryptée par la clé `K` du _Safe_) des documents de l'application `app1` pour `Bob`.

> En _debug_ il est simple d'effacer ces données sélectivement: en revanche la lecture des contenus des items et des bases IDB est plus problématique. Les données sont soit codées, soit cryptées.

#### Propriétés de l'item `Bob@Ktux...` du _localStorage_
Par principe l'item est crypté par une clé `sha(E + X)`. C'est une _pseudo_ sécurité car malheureusement X est lisible (sans trop de difficulté) en debug de _Safe_ terminal.

C'est un objet ayant les propriétés suivantes:
- `devName` : nom identifiant le device pour le _Safe_ de `Bob`, par exemple `PC d'Alice`.
- `K1 K2 Kp` : cryptages de la clé `K` du _Safe_ respectivement par `SH(p0, p1, SEP)`, `SH(p0, p2, SEP)` et `SH(PIN + cy, cz, SEP)`.
- `cx` : challenge x aléatoire généré à la déclaration de confiance.
- `cy` : challenge y aléatoire généré à la déclaration de confiance.

En désérialisant un tel item, un hacker n'obtient rien de directement utilisable. Il ne peut pas obtenir la clé `K` du _Safe_ sans connaître `p0` et (`p1` ou `p2`), ou `PIN` et `cz`:
- à la limite il peut _deviner_ `p0`, mais ni `p1` ni `p2` ne sont accessibles par force brute en raison de leur longueur.
- tenter de cracker par force brute `Kp` est voué à l'échec: `PIN` est court mais `cz` est long.

### Section _devices de confiance_ d'un _Safe_
Cet objet **crypté par la clé de configuration `Kms` du module _Safe_ serveur.** a les propriétés suivantes:
- `mdev` : map des appareils déclarés _de confiance_ pour ce _Safe_. 
  - clés: `sha(devName)`
  - valeur : `devName` crypté par la clé `K`.
  - cette map facilite la suppression d'un appareil de confiance par le propriétaire du _Safe_.
- `cx` : challenge x aléatoire généré à la déclaration de confiance.
- `cy_K` : challenge y aléatoire généré à la déclaration de confiance, crypté par la clé `K` du _Safe_.
- `cz` : challenge z aléatoire généré à la déclaration de confiance.
- `Va` : clé de vérification de `signcy` générée (conjointement avec `Sa`) à la déclaration de confiance.
- `signcy` : signature par `Sa` du _challenge_ `SH(PIN, cy)`.
- `Kp` : clé `K` du _Safe_ cryptée par `SH(PIN + cy, cz, SEP)`.
- `nbe` : nombre d'échecs de proposition de code PIN.

### Accès au _Safe_ par code PIN depuis un appareil déjà déclaré de confiance par Bob
L'objectif est d'obtenir l'accès au _Safe_ de `Bob` à l'ouverture d'une application.
- accès à l'item `Bob@Ktux...`: l'utilisateur a sélectionné `Bob` dans la liste proposée ce qui lui donne `Ktux...` l'identifiant de son _Safe_.
- l'utilisateur saisit un PIN et le module _Safe_ terminal interroge le _Safe_ serveur en lui passant en paramètres:
  - `appi` le code de l'application.
  - `sha(devName)` le nom de l'appareil généré à sa déclaration de confiance.
  - `Ktux...` l'identifiant de son _Safe_ (sa clé `E`).
  - `cx` challenge `x` trouvé dans l'item (_version_ de la déclaration du code PIN).
  - `SH(PIN, cy)` où `cy` est le challenge `y` trouvé dans l'item et `PIN` le code PIN saisi par l'utilisateur.
- le module _Safe_ serveur:
  - accède au _Safe_ pour l'identifiant `Ktux...`.
  - vérifie que `sha(devName)` est bien une clé de la map `mdev` du _safe_.
  - vérifie que `cx` reçu en paramètre est égal à `cx` du _Safe_.
  - vérifie par `Va` que `signcy` est bien la signature de `sha(PIN + cy)`.
  - en cas de succès, il met à 0 `nbe` s'il ne l'était pas déjà.
  - retourne `cz, K1, K2, Kp ` et la liste des _droits_ pour l'application de code `appi` (chacun est doublement crypté par `K` et la clé de cryptage de `appi`).

Ayant le code `PIN` par saisie, `cy` dans l'item lu du _localStorage_ et `cz` retourné par le serveur, le module _Safe_ terminal peut désormais décrypter `Kp` et en obtenir la clé `K` du _Safe_ de `Bob`:
- **il peut décrypter les droits reçus**, chacun étant doublement crypté par,
  - la clé `K` qu'il vient d'obtenir,
  - la clé de cryptage de `appi` dont l'application a dans son code la clé privée de décryptage.

##### Échecs
- (1) si le `devName` reçu du _Safe_ terminal n'est pas dans la liste détenue dans le _Safe_ serveur, c'est que cet appareil N'EST PAS / PLUS de confiance.
- (2) si le challenge `cx` détenu en _localStorage_ ne correspond pas au `cx` enregistré dans le _Safe_, le module _Safe_ serveur considère le _localStorage_ de l'appareil pour `Bob` est basé sur une déclaration _ancienne_ du code PIN.
- (3) si la signature `signcy` n'est pas vérifiée par `Va`, c'est que le code PIN n'est pas le bon. Ayant passé l'étape (2), `Va` correspond bien au `Sa` qui a été utilisé à sa signature, `cy` était bien celui fixé à la déclaration. **Le nombre d'erreurs `nbe` est incrémenté**.
  - si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais. Les données de la section _appareils de confiance_ sont supprimées à l'exception de la liste `mdev`. L'utilisateur devra redéclarer un code PIN (ce qui exigera une authentification _forte_ par `p0` et (`p1` ou `p2`)).

En cas de réussite, le nombre d'échecs `nbe` est remis à 0 s'il ne l'était pas déjà.

#### Remarques sur la _sécurité_ du protocole
- le **code PIN** n'est jamais stocké ni passé en clair sur le réseau au module _Safe_ serveur: 
  - il ne peut pas être détourné ou être lu depuis la base de données.
  - il ne figure que temporairement en mémoire de _Safe_ terminal durant la phase d'authentification du _Safe_ de `Bob`.
- pour tenter depuis les données du _Safe_ serveur d'obtenir le code PIN par force brute, il faut effectuer une vérification de `signcy` avec le _challenge_ `SH(PIN, cy)` mais `signcy` est crypté par la clé `Kms` du module _Safe_ serveur.

Pour que cette dernière attaque pour trouver le PIN de `Bob` par force brute ait des chances de succès, il faut que le hacker ait obtenu frauduleusement:
- (1) le contenu en clair de l'objet _safe_ en base et pour cela il lui faut conjointement,
  - avoir accès à la base en lecture ce qui requiert, soit une complicité auprès du fournisseur de la base de donnée, soit **la complicité de l'administrateur technique**.
  - avoir la clé de décryptage des contenus de celle-ci (la clé `Kms`) inscrite dans la configuration de déploiement des serveurs. Ceci suppose la **complicité de l'administrateur technique** effectuant ces déploiements.
- (2) le terme `cy` lisible en _debug_ (et un peu d'effort) dans le _localStorage_ d'un appareil de confiance **débloqué** (session utilisateur ouverte) de Bob.
  - sur un mobile avoir le mobile _déverrouillé_,
  - sur un PC avoir une session ouverte. 

> Ce double _piratage / complicité_ donne accès à la clé `K` du _Safe_ de `Bob`, donc à toutes ses clés. Toutefois les phrases `p0 p1 p2` restent inviolées et non modifiables par le hacker, puisque ne résidant que dans la mémoire de Bob.

> Pour cracker **tous** les codes PIN, il faudrait pouvoir accéder à tous les appareils de confiance **déverrouillés / sessions ouvertes** et casser par force brute le PIN de chaque _Safe_. Si Alice n'a pas déclaré comme appareil _de confiance_ l'un de ceux déclarés par Bob, son _safe_ reste inviolé. Chaque _casse_ reste ponctuel et les utilisateurs n'ayant déclaré aucun appareil de confiance sont à l'abri. 

#### Durcir (un peu) le code PIN
Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il est quasi incassable dans des délais humains: pour être mnémotechnique il va certes s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. Mais de nombreux styles de saisie mènent au code PIN depuis la phrase `allons enfants de la patrie`: avec ou sans séparateurs, des chiffres au milieu, des alternances de mots en minuscules / majuscules, un mot sur deux, etc. La seule _bonne intuition_ d'un texte est loin de donner le code PIN correspondant.

> Un _login_ des appareils un peu conséquent et un code PIN _un peu durci_ constituent en pratique une barrière **très coûteuse** à casser. Tant qu'à être un _délinquant_ une forte pression directe sur Bob devrait permettre de lui extorquer ses phrases / PIN à moindre coût.

### Déclaration d'un nouvel appareil de confiance
Depuis l'appareil Bob doit:
- identifier son _safe_ par `p0` et (`p1` ou `p2`).
- donner un nom à cet appareil comme `mobile d'Alice`.
- déclarer son pseudo si aucun n'a jamais encore été déclaré.
- donner le code PIN déjà déclaré pour d'autres appareils, soit en inventer un.

### Changer le code PIN des appareils de confiance
Depuis l'appareil Bob doit:
- soit identifier son _safe_ par `p0` et (`p1` ou `p2`), soit si l'appareil est _déclaré de confiance_ son code PIN actuel.
- donner le nouveau code PIN.

### Supprimer des appareils de confiance
Depuis l'appareil Bob doit:
- soit identifier son _safe_ par `p0` et (`p1` ou `p2`), soit si l'appareil est _déclaré de confiance_ son code PIN actuel.
- désigner dans la liste qui s'affiche les appareils de confiance qui ne le sont plus.
- s'il n'en reste plus, le code PIN est détruit.

# Acquisition de droits par un utilisateur
Il existe plusieurs processus pour acquérir un droit selon le protocole applicatif choisi.

### Acquisition directe dans l'application terminale
Dans cette situation c'est l'application terminale qui génère le _droit_.

L'exemple pris ici est _la création d'un compte dans l'application_:
- l'application terminale a généré l'`id` du compte ou l'a obtenu de son serveur et a généré un couple de clés `S / V`.
  - soit elle fait enregistrer par le _safe_ de l'utilisateur le couple `id, S`, 
  - soit elle affiche ce couple à l'utilisateur pour qu'il l'inscrive dans son fichier CSV de droits.
- ENFIN elle _valide_ la création du compte auprès du serveur de l'application pour qu'il enregistre en base le couple `id / V`.

### Obtention par l'application terminale d'un droit stocké dans le serveur
Un certain nombre de droits peuvent être _configurés_ dans le serveur de l'application: en d'autres termes ils sont inscrits, soit _en dur_ (et chargés à l'initialisation), soit _en base_ par une autorité _supérieure_ ayant le droit de créer des droits identifiés.

> Les `S` de ces droits ne sont pas lisibles directement en base afin qu'un détournement de celle-ci ne donne pas accès à ces clés mais sont dans des objets cryptés par une clé fixée par l'administrateur technique. 

Dans ce cas, l'application terminale récupère depuis le serveur le couple `id, S` du droit attribué à  l'utilisateur et le fait enregistrer dans le _safe_ de l'utilisateur ou lui affiche pour incorporation dans son fichier CSV ou autre. 

### Attachement à un autre droit
Le serveur dispose pour un droit `dx` non pas d'une clé `S` mais d'une liste d'autres droits `d1, d2 ...` Par exemple:
- `dx` est un droit de gestion d'un tarif,
- `d1 d2 ...` sont les logins à qui ce droit a été attribué.

Pour exécuter une opération requérant le droit `dx`, il suffit que le _jeton_ ait un des droits `d1 d2 ...`

### Transmission explicite entre détenteurs de droits
Si une personne P1 dispose d'un droit, il est matérialisé par l'existence _quelque part_ de son couple `id, S` associé.

Basiquement P1 peut par exemple transmettre à une personne P2 ce droit par un simple e-mail à l'adresse de P2 qui l'ajoutera à son _safe_ ou à son fichier CSV.

Ce procédé général _d'export / import_ demande à résoudre les points suivants:
- comment P1 connaît-elle P2, est-ce que P1 a accès à un media externe (e-mail, SMS, ...) par lequel P2 est joignable ?
- quelle sécurité cet échange a-t-il, peut-il être _écouté / intercepté_ au milieu, comment s'assurer que seule P2 pourra en faire usage ?

> Quand P1 et P2 sont inscrit sur le _Safe_ et que P1 connaît l'identifiant du _safe_ de P2, elle peut transmettre le droit dans un _objet_ accessible par P2 dans ses _objets reçus_.

# TODO
- passer le _module_ en application.
- déclarer les applications _favorites / utilisables_.
- langue / mode sombre
