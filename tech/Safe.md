---
layout: page
title: L'application "Safe", la gestion des "droits d'accès"
---

# Gestion des _credentials / droits d'accès_ dans les applications

Une application `myApp1` met en jeu deux logiciels:
- `myApp1 server` s'exécute sur un pool de serveurs au service de l'application gérant ses données centrales persistantes, ses opérations de mise à jour et ses extractions / synchronisations de données.
- `myApp1 terminal` l'application terminale s'exécutant typiquement dans un browser Web et dont le source est lisible et délivré par un serveur statique / CDN.

Les opérations exécutées par le serveur comme les données qu'il peut retourner à l'application qui l'a sollicité, sont soumises, sauf exception, à des **droits d'accès**: d'une manière ou d'une autre l'utilisateur derrière l'application terminale doit prouver qu'il possède effectivement les _droits requis_ pour solliciter une opération afin de mettre à jour et / ou obtenir des données centrales. Par exemple:
- `cpt` : droit à lire les documents d'un compte,
- `mbr` : droit de gestion des accès des membres d'un groupe,
- `trf` : droit de modification tarifaire ...

## Vérification des _droits d'accès_ par _jetons signés_
### Droit d'accès / _credential_
Un _droit d'accès_ est matérialisé par les données suivantes:
- `type`, un code correspondant à sa _classe / catégorie_ `cpt, mbr, trf ...`
- `target` : l'identifiant dans l'application de sa _cible_ qui peut comporter aussi bien des données lisibles (une adresse e-mail, un numéro de mobile ...) qu'être le résultat d'une génération aléatoire. Par exemple pour un droit `cpt`, l'identifiant du compte.
- une _liste de clés_, souvent d'un seul terme:
  - `var` : un code (vide par défaut) permettant de disposer de plusieurs clés pour un même droit identifié par `type / target`.
  - `S` : clé privée de **signature** (environ 400 bytes).
  - `V` : clé publique de **vérification** (environ 100 bytes).

Les serveurs des applications ne détiennent des _droits d'accès_ **QUE** les propriétés `type target V` et n'ont PAS (sauf exception décrite ci-dessous) accès à le clé `S` correspondante. Un _serveur_ vérifie la validité d'un droit transmis par _l'application terminale_ de la manière suivante:
- l'application terminale génère un texte _challenge_ qui n'a jamais été généré et ne sera jamais plus présenté à l'application serveur.
- elle _signe_ ce _challenge_ par sa clé `S` et transmet au serveur le couple du challenge et de sa signature.
- le serveur utilise sa clé `V` correspondante à la clé `S` utilisée à la signature et peut vérifier que la signature reçue est bien celle du challenge transmis.

> Cette technique permet au serveur de s'assurer de la validité d'un droit sans avoir eu en mémoire la clé `S` de signature: c'est un avantage de confiance par rapport aux solutions basées sur un mot de passe qui, à un moment ou à un autre, a besoin d'être présent dans la mémoire du serveur, même si un hachage fort (type PBKDF) de mots de passe longs limite le risque.

### Jetons d'accès aux opérations
Quand une application terminale soumet une opération à un serveur, elle fournit dans sa requête un **jeton d'accès** qui réunit les preuves que son utilisateur dispose des _droits_ requis pour exécuter cette opération. Un jeton comporte:
- `sessionId` : C'est l'identification d'UNE exécution l'application terminale sur UN _device_, à un instant donné une seule exécution terminale de l'application peut s'en prévaloir.
- `time` : date-heure en milliseconde de la génération du jeton d'accès. Cette donnée fait partie du _challenge_ des signatures du jeton.
- une liste de preuves de possession des droits constituée chacune d'un triplet:
  - `type` : du droit,
  - `target` : _identifiant_ de la cible à laquelle le droit s'applique,
  - `signatures` : liste de signatures `{var1: sig1, ...}` du couple `sessionId,  time` par chaque variante `var1` de la clé `S` du droit correspondant.

**Remarques**:
- pour une application _web-push_, `sessionId` est un hash du (long) `devAppToken` attribué par le browser à l'application lors de l'enregistrement de son _service_worker_:  en conséquence `devAppToken` change si l'utilisateur du device supprime le service_worker qui se ré-enregistrera au prochain appel mais avec un token différent. Un _hacker_ un peu entraîné peut obtenir l'identifiant `devAppToken / sessionId` en lançant en _debug_ l'application sur ce _device_.
- dans la liste des preuves, plusieurs peuvent concerner le même couple `type target`: il suffit que l'une d'elle soit reconnue comme valide pour que le droit le soit. C'est une manière de traiter les situations, temporaires ou non, où un même droit peut avoir plusieurs _variantes_ de clés de signature / vérification (une _ancienne_ et une _nouvelle_, une variante par _pseudo_ ...).
- un _jeton d'accès_ est crypté par la clé publique d'encryption du serveur applicatif ciblé de sorte que seul celui-ci puisse le lire. Cette clé fait partie de la _configuration_ du serveur que son administrateur technique délivre lors du déploiement et qu'il doit conserver confidentiellement.

### Validation des _droits_ d'un jeton par le serveur de l'application
Quand le serveur d'une application traite une opération soumise par l'application _terminale_,
- il obtient de la requête le jeton d'accès et le décrypte par sa clé privée de décryptage.
- le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _droit_ de la liste du jeton,
  - il obtient depuis sa base de données la map dés clés acceptées `{var1: V1, var2:v2 ...}` de vérification associée à sa cible,
  - il vérifie que la `signature` du couple `sessionId, time` est bien validée par la variante `var1` de `V`, ce qui prouve que l'application terminale en détient effectivement la clé de signature `S` correspondante.

> `sessionId, time` est utilisé comme _challenge_ cryptographique et n'est pas présenté plus d'une fois pour une application donnée.

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `target` d'un `type` de droit la dernière map dés clés acceptées `{var1: V1, var2:v2 ...}` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `{var1: V1, var2:v2 ...}`, et en cas de changement refait une vérification avant de valider / invalider le droit correspondant.
- le serveur conserve en cache pour chaque `sessionId` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, leur _validation_ ne requiert qu'un calcul en mémoire sans accès à la base pour obtenir les clés de vérification.

### "Un" droit, "plusieurs" clés
Le serveur _peut_ mémoriser pour un droit non pas une clé `V` mais une liste de clés `{var1:V1, var2:v2 ...}`: pour être validé, un droit d'un jeton d'accès doit fournir une signature qui a été établie par la clé `Si` correspondant à la variante acceptée. 
- Si la variante acceptée ne figure pas dans le jeton d'accès, c'est que l'application terminale N'A PLUS ACCÈS au droit qui a été changé par l'application serveur et n'a pas jugé pertinent de retransmettre cette information à l'utilisateur pour qu'il la stocke dans ses droits.
- Si la variante figure mais que la vérification échoue, c'est une tentative de fraude.

La logique applicative peut ainsi prévoir de distribuer plusieurs _variantes_ de clés à des détenteurs différents selon ses propres critères (par exemple un _pseudo court_ du détenteur): elle peut aussi au cours du temps _désactiver_ certaines variantes dans le serveur et inhiber ainsi les détenteurs qui les possédaient.

> Ce dispositif permet aussi de gérer un _changement de clé_ pour un droit lorsqu'il a été attribué trop généreusement ou pour un temps limité. La création d'une nouvelle clé peut être faite et sa distribution restreinte aux seuls détenteurs souhaitables dans le futur, le cas échéant après un certain temps où les deux peuvent être admises, _l'ancienne_ peut être supprimée.

### Utilisation de la liste des droits par une application terminale
Celle-ci a obtenu la **liste des droits** de l'utilisateur: pour une opération donnée elle va devoir choisir celui approprié. Plusieurs situations se présentent:
- (1) pour le `type` fixé (par exemple `DRTARIF` : _droit à effectuer une modification tarifaire_), il n'existe qu'une clé unique, `target` est vide. Un seul terme de la liste des droits peut s'appliquer.
- (2) le `type` ET la valeur de `target` sont fixées (par exemple `LOGIN, 1234` : _droit de l'utilisateur 1234 à se connecter_) par l'application. Un seul terme _au plus_ de la liste des droits peut s'appliquer.
- (3) le `type` est fixé MAIS PAS la valeur de `target` (par exemple `LOGIN`):  l'application terminale recherche **à la fois une cible et sa clé**. Plusieurs droits de la liste pouvant être candidats c'est l'utilisateur qui devra **désigner** le login qu'il choisit: dans ce cas le commentaire `about` attaché à chaque droit lui est utile (par exemple en donnant un nom en clair plutôt qu'un code).

## Applications _légitimes / officielles_ versus _pirates_
Si une application terminale est une application _pirate_ lancée par exemple depuis un lien envoyé par un e-mail frauduleux, 
- elle peut demander à l'utilisateur ses justificatifs de droits d'accès en _singeant_ l'application légitime, 
- elle _peut_ envoyer ces clés usurpées à un serveur pirate où elles seront à disposition de pirates pour se faire délivrer des données par l'application serveur légitime.

> L'application _légitime_ a certes dans son code _sa clé de décryptage privée_ qui lui permet de décrypter les droits: mais en exécution en mode _debug_ un hacker un peu habile peut la retrouver. Cette _sécurité_ est plus symbolique qu'effective.

Il n'existe aucun procédé logiciel _universel_ qui permette de connaître l'origine d'une application, de quelle _source_ elle vient, si elle est _légitime_ ou _pirate_. Toutefois depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle de l'application pirate,
- s'être assuré que le CDN correspondant distribue bien le source _officiel_ et non pas un _source modifié_. Pour cela il faut,
  - comparer le hash des fichiers sources distribués par le CDN avec les hash des fichiers source du repository _officiel_,
  - avoir obtenu d'un expert indépendant l'assurance que le code _officiel_ est bien légitime et ne redistribue pas de clés d'accès,

> Si ces conditions sont possibles à vérifier pour une _application terminale_ Web, en revanche il n'existe aucun procédé technique permettant à un utilisateur de savoir si l'application _serveur_ hébergée est bien celle dont les sources (en Open Source) seraient disponibles dans un repository public: il faut _faire confiance_ à l'hébergeur.

# Les modules _safe terminal_ et _safe server_

Ils sont embarqués respectivement dans _myApp1 terminal_ et _myApp1 server_: les deux modules communiquent entre eux, le _terminal_ pouvant solliciter des opérations du _server_ par des requêtes HTTPS.

Ils ont pour objet de gérer le _coffre fort_ des utilisateurs.

> Les modules _safe terminal / server_ ne gèrent pas à proprement parler des _personnes_ mais leurs _coffres forts_: rien n'empêche un _personne_ de posséder plus d'un coffre, rien ne relient les coffres entre eux ni à un quelconque signifiant dans le monde réel.

Après avoir lancé l'application _myApp1 terminal_ depuis son appareil, un utilisateur va lui indiquer quel est son _coffre fort_ afin d'accéder en toute sécurité aux données confidentielles qui le concerne.

### Sessions et _profils_ de sessions
Quand un utilisateur lance une application _myapp1_ depuis un _device_ il ouvre une session, identifiée de manière unique pour cette application: sur un _device donné_, une seule session peut s'exécuter à un instant donné pour l'application _myapp1_.

A la première toute ouverture d'une session de l'application _myapp1_ l'utilisateur va devoir:
- citer le ou les droits d'accès dont il se prévaut dans cette session, typiquement en _cochant_ ceux qu'il a déjà acquis et stockés dans son coffre.
- fixer éventuellement quelques préférences d'ouverture (langue, disposition préférée, etc.).

Si l'utilisateur prévoit de rouvrir un jour plus tard une session dans les mêmes conditions (mêmes droits et mêmes préférences), il peut enregistrer dans son _coffre_ le _profil_ de sa session et lui donner un _à propos_ significatif pour lui: ainsi ultérieurement quand il voudra rouvrir une session similaire, au lieu de re-citer droits et préférences, il désignera simplement ce _profil_.

### Devices _de confiance_
Un utilisateur qui veut utiliser une application depuis un _device_ est placé devant deux cas de figure:
- **soit il n'a pas confiance dans cet appareil** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver des données.
- **soit il juge l'appareil _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - il peut y laisser _en cache_ des informations cryptées et espérer raisonnablement les retrouver plus tard.

Un utilisateur peut déclarer sa _confiance_ au _device_ qu'il utilise:
- son _coffre_ enregistre ce _device_ comme étant de confiance,
- le _device_ enregistre localement la référence à cette déclaration de confiance.

Lancer une application depuis un appareil _de confiance_ a plusieurs avantages:
- **authentification simplifiée** de son _coffre_ par l'utilisateur en donnant un code PIN court (pour accéder à ses profils de sessions des applications et à ses _droits_).
- **disposer sur ce device de _mémoires caches persistantes et cryptées de documents_** pour chaque _profil_ de session ce qui lui permet d'ouvrir une session,
  - en mode _réseau_ minimisant le nombre de documents à récupérer des serveurs,
  - en mode _avion_ (sans accès au réseau) avec accès en lecture aux documents dans l'état où ils se trouvaient lors de la dernière fin de session en mode _réseau_ sur ce _device_.

## Sections des _coffre fort_
La base de données _Safe_ stocke les données de chaque _safe_ dans un document. Elle est accédée par le module _safe server_ embarqué dans les applications serveur comme _myApp1 server_.

Le document décrivant un _coffre fort_ a plusieurs sections:
- section `auth`: données d'authentification qui permettent de s'assurer que l'utilisateur en est vraiment le propriétaire légitime.
- section `devices`: chaque entrée dans cette section identifie un _device de confiance_.
- section `creds`: liste des _credentials_ détenus dans le coffre. Chaque _credential_ y est identifié par une id aléatoire et a un _a propos_ texte signifiant pour l'utilisateur.
- section `profiles`: liste des _profils de session_ que l'utilisateur peut ouvrir (regroupés par application). Un profil est décrit par:
  - une id aléatoire,
  - un _à propos_, texte signifiant pour l'utilisateur.
  - la liste des _credentials_ qui seront attachés à une session lors de son ouverture.
  - une liste éventuelle de _préférences_ utilisées à l'ouverture d'une session.

### Section `auth`

#### Création d'un _safe_ par un utilisateur
Un identifiant safeId est généré aléatoirement.

Une clé AES `K` de 32 bytes est tirée aléatoirement: elle ne pourra pas changer et est la clé de cryptage du _safe_.

L'utilisateur donne:
- un _couple_ `p0, p1` (qui pourra être changé) _d'authentification_:
  - `p0` est un pseudo / prénom-nom / adresse mail / numéro de téléphone / etc. qui identifie de manière unique le _safe_ (le SH de `p0` est un index unique).
  - `p1` est une phrase _longue_ d'au moins 24 signes.
- un _couple_ `r0, r1` (qui pourra être changé) _de récupération_:
  - `r0` est un pseudo / prénom-nom / adresse mail / numéro de téléphone / etc. (12 signes au moins) qui identifie de manière unique le _safe_ (le SH de `r0` est un index unique) et qui peut être égal à `p0`.
  - `r1` est une phrase _longue_ d'au moins 24 signes. Il n'est pas judicieux qu'elle soit égale à p1 puisqu'elle permet justement la récupération du safe en cas d'oubli de `r0, p0`.

La clé `K` du safe est stockée,
- dans `Ka` et `Kr` cryptages respectifs par  `SH(p0, p1)` et `SH(r0, r1)`.
- `hk` : SHA du `SH(K)` permettant au module _safe server_ de vérifier sur chaque opération demandée par _safe terminal_ que celui-ci détient bien la clé K (transmise par SH(K)).

A aucun moment `p0 p1 r0 r1` ne sont stockés ni transmis _en clair_. Elles ne figurent _en clair_ que très temporairement à la saisie par l'utilisateur dans le module _safe terminal_ et y sont effacés dès la fin de la saisie.

Pour changer `p0, p1` et/ou `r0, r1` l'utilisateur doit fournir,
- soit le couple actuel `p0, p1` OU `r0, r1`.
- les nouveaux couples `p0, p1` et `r0, r1`. 

#### Synthèse des propriétés de la section `auth`
- `safeId` : identifiant.
- `maxLife` : durée de vie du _safe_, sachant que toute utilisation recule cette date (permet une _purge_ périodique des _safe_ obsolètes / fantômes).
- `hp0` : index unique, SH(p0).
- `hr0` : index unique, SH(r0).
- `hk` : SHA du `SH(K)`.
- `Ka` : clé `K` du safe cryptée par SH(p0, p1).
- `Kr` : clé `K` du safe cryptée par SH(r0, r1).
- `idx` : dernier numéro attribué à un identifiant local de credential / profil.

### Section `devices`
Chaque _device de confiance_ à une entrée identifiée par `about` dans cette section:
- `about` : code / texte court donné par l'utilisateur pour qualifier le _device_. Par exemple `Bob sur le PC d'Alice`.
- `{ Va, cy, sign, nbe }` : propriétés permettant de valider que ce _device_ est de confiance (voir plus loin).

Après avoir authentifié son accès à son _safe_, l'utilisateur peut retirer sa confiance à n'importe lequel des devices cités dans la liste en en supprimant l'entrée.

### Section `creds`
Chaque _droit d'accès / credential_ est enregistré dans un item **crypté par la clé K** du safe sous un identifiant généré aléatoirement à sa création. Ses propriétés sont:
- `about` : code / texte court donné par l'utilisateur pour qualifier le _credential_. Par exemple `Compte Bob sur circuits courts`. 
- `type, target, keys: [{var, S, V}, ...]` : données du _credential_, ses clés d'accès.

### Section `profiles`
Elle est organisée avec une **sous-section par application** regroupant une liste d'items ayant un identifiant généré aléatoirement à sa création. Chaque item est **crypté par la clé K** de _safe_ et a les propriétés suivantes: 
- `about`: un _à propos_, texte signifiant pour l'utilisateur. Par exemple `Revue des notes d'Alice et Jules`.
- `creds`: la liste des id des _credentials_ qui sont attachés à une session de ce profil lors de son ouverture.
- `prefs`: un objet facultatif donnant les _préférences_ utilisées à l'ouverture d'une session interprétable par l'application.

## Accès d'une application terminale à un _safe_
### Depuis n'importe quel _device_ (de confiance ou non)
Le module _safe terminal_ demande à l'utilisateur `p0 p1` et les transmet au module _safe server_ qui accède au document _safe_ depuis le `SH(p0)` et retourne `Ka`.

_safe terminal_ décode `Ka` par `SH(p0, p1)`: en cas d'échec c'est que `p1` était incorrect.

### Depuis un _device_ de confiance
Un device qui a été déclaré _de confiance_ par au moins un utilisateur a une micro base de données IDB nommée `Safes` ayant les tables suivantes:
- `DEVICE`: chaque row a les colonnes suivantes et correspond à une déclaration de confiance de ce device par un utilisateur:
  - `safeId`: identifiant du _safe_ de l'utilisateur.
  - `about`: par exemple `Bob sur le PC d'Alice` identifiant ce device pour cet utilisateur. Le couple `safeId about `est clé primaire.
  - `cx`: un challenge aléatoire.
  - `Ka`: clé K du safe de l'utilisateur cryptée par `SH(p0, p1)` où `p0` et `p1` sont les termes d'authentification du safe de l'utilisateur.
  - `Kp`: clé K du safe de l'utilisateur cryptée par `SH(PIN + cx, cy)` où,
    - `PIN` est le code PIN fixé par l'utilisateur à la déclaration de confiance et `cx cy` des challenges générés aléatoirement à ce moment.
- `CACHE`: chaque row identifie un cache de documents et le profil de la session auquel il correspond:
  - `app`: code l'application correspondante.
  - `idbId`: nom local aléatoire de la base locale IDB.
  - `safeId`: identifiant du _safe_ de l'utilisateur.
  - `id`: id du profil de la session utilisant ce cache.
  - `about`: _à propos_ de ce profil, par exemple `Revue des notes d'Alice et Jules`.
  - il existe une base de données IDB de nom `app.idbId` contenant les documents en cache pour une session de ce profil.

> Les rows de la base IDB Safe sont cryptés par une clé C du module _safe terminal_ afin de ne pas être directement lisible en _debug_. Toutefois cette _sécurité_ est _molle_, la clé étant d'une manière ou d'une autre inscrite dans le code, avec un peu de fatigue un hacker peut la retrouver.

#### Déclaration d'un _device_ de confiance
Depuis le _device_ à déclarer de confiance, l'utilisateur:
- saisit `about` un nom explicite pour lui, par exemple `Bob sur le PC d'Alice`.
- saisit un code `PIN` (d'au moins 6 signes).
- saisit le couple `p0 p1` d'accès à son _safe_.

Le module _safe terminal_ demande au module _safe server_ d'accéder au safe de l'utilisateur identifié par SH(p0) et de lui retourner le `safeId` et `Ka` associé:
- disposant du couple `p0 p1`, le module _safe terminal_ obtient la clé `K` du safe de l'utilisateur en décryptant `Ka` par le SH(p0, p1).

Le module _safe terminal_,
- génère les challenges aléatoires `cx cy`.
- calcule `Kp`, cryptage de cryptage de la clé `K` par le `SH(PIN + cx, cy)`.
- génère un couple `Sa Va` de clés asymétriques signature / vérification.
- calcule `sign`, signature par `Sa` du `SH(PIN, cx)`.
- calcule `hp1` comme SH(p1).
- enregistre dans la table `DEVICE` de la base IDB `Safes` un row avec les colonnes `safeId about cx Ka Kp`.
- transmet au module _safe terminal_ `safeId, hp1, about, Va, cy, sign` qui,
  - accède au _safe_ dont l'id est safeId et vérifie que les hp1 correspondent (s'assure que _safe terminal_ détient le bon p1).
  - y créé dans la section `devices` une entrée about avec les données `Va cy sign nbe = 0`.

> Remarque: `Sa` a servi à générer la signature sign mais n'est plus utilisé ensuite et n'est pas mémorisé. `Va` l'est et servira à authentifier la signature d'un PIN saisi par Bob.

Après ce calcul,
- le _safe_ a été mis à jour par le module _safe server_ avec un nouveau device de confiance avec les données cryptographiques permettant à l'utilisateur de s'authentifier par un code PIN.
- sur le _device_ la base locale IDB _safes_ contient une entrée relative à cette déclaration de confiance avec en particulier la clé K du _safe_ cryptée en Ka et Kp. 

#### Authentification par code PIN depuis un _device déclaré de confiance_
Le module _safe terminal_ lit la base IDB _Safes_ et, 
- propose à l'utilisateur de désigner la ligne de DEVICE dont la propriété about correspond à lui: par exemple `Bob sur le PC d'Alice`. Le module dispose ainsi des données `safeId about cx Kp`.
- demande à l'utilisateur de saisir le PIN associé et calcule `z` = SH(PIN, cx).
- transmet au module _safe terminal_ `safeId, about, z` qui,
  - accède au _safe_ dont l'id est `safeId`.
  - accède dans la section `devices` à l'entrée `about` ce qui lui donne les propriétés `Va cy sign nbe`. Si cette entrée n'existe pas c'est que ce _device_ N'EST PAS / PLUS de confiance,
    - soit n'a jamais été déclaré,
    - soit a été supprimé explicitement par l'utilisateur,
    - soit qu'il a été supprimé du fait d'un nombre excessif d'essai de code PIN.
  - vérifie par `Va` que `sign` est bien la signature de `z` (SH(PIN, cx)). Nn cas de succès, il met à 0 `nbe` s'il ne l'était pas déjà et sinon incrémente `nbe`.
  - retourne le challenge `cy` au module _safe terminal_ qui peut ainsi calculer la clé SH(PIN + cx, cy) qui décrypte `Kp` ce qui lui donne la clé K du _safe_.

##### Échecs
Quand la signature `sign` n'est pas vérifiée par `Va`, c'est que le code PIN n'est pas le bon. `Va` correspond au `Sa` qui a été utilisé à sa signature, `cx` était bien celui fixé à la déclaration. **Le nombre d'erreurs `nbe` est incrémenté**.

Si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais, l'entrée `about` est supprimée. L'utilisateur devra refaire une _déclaration de confiance_ de ce device avec un code PIN (ce qui exigera une authentification _forte_ par `p0` et (`p1` ou `p2`)).

### Accès d'une application en mode _avion_ (pas d'accès au réseau)
La base IDB _Safes_ permet de localiser une base IDB cache de documents: c'est l'utilisateur qui désigne celle-ci d'après le texte `about` de son profil.

MAIS cette base est cryptée par la clé K du safe de l'utilisateur. Ce dernier doit:
- désigner dans la liste de DEVICE proposés celui de sa convenance (par exemple `Bob sur le PC d'Alice`) ce qui va donner Ka.
- saisir son couple p0 p1 authentifiant son accès au _safe_ et décrypter Ka par SH(p0, p1) pour obtenir sa clé K.

> En mode _avion_ l'authentification par code PIN n'est pas possible.

### Remarques sur la _sécurité_ de l'authentification par code PIN depuis un _device déclaré de confiance_
- le **code PIN** n'est jamais stocké ni passé en clair sur le réseau au module _safe server_: 
  - il ne peut pas être détourné ou être lu depuis la base de données.
  - il ne figure que temporairement en mémoire dans le module _safe terminal_ inclus dans l'application _myApp1 terminal_ durant la phase d'authentification du _safe_ de l'utilisateur.
- pour tenter depuis les données du _Safe server_ d'obtenir le code PIN par force brute, il faut effectuer une vérification de `sign` avec le _challenge_ `SH(PIN, cx)` mais `sign` est crypté par la clé privée de cryptage général du module _safe server_.

Pour que cette dernière attaque pour trouver le PIN de `Bob` par force brute ait des chances de succès, il faut que le hacker ait obtenu frauduleusement:
- (1) le contenu en clair de l'objet _safe_ en base et pour cela il lui faut conjointement,
  - avoir accès à la base en lecture ce qui requiert, soit une complicité auprès du fournisseur de la base de donnée, soit **la complicité de l'administrateur technique**.
  - avoir la clé de décryptage des contenus de celle-ci inscrite dans la configuration de déploiement des serveurs. Ceci suppose la **complicité de l'administrateur technique** effectuant ces déploiements.
- (2) le terme `cx` lisible en _debug_ (et un peu d'effort) dans la base de données IDB _Safes_ d'un _device_ **débloqué** (session utilisateur ouverte) de Bob.
  - sur un mobile avoir le mobile _déverrouillé_,
  - sur un PC avoir une session ouverte.

Ayant obtenu le challenge cx, il faut craquer par force brute le code PIN.

> Ce double _piratage / complicité_ donne accès à la clé `K` du _safe_ de `Bob`, donc au contenu du _safe_, dont les clés d'accès. Toutefois les phrases `p0 p1 p2` restent inviolées et non modifiables par le hacker, puisque ne résidant que dans la mémoire de Bob.

> Cracker le code PIN d'un profil de Bob pour myApp1 sur un device ne compromet en rien, NI les codes PIN de Bob sur d'autres devices, NI le code PIN d'Alice sur ce device.

> Pour craquer **tous** les codes PIN, il faudrait pouvoir accéder à tous les appareils de confiance **déverrouillés / sessions ouvertes** et casser par force brute le PIN de _chaque safe pour chaque profil_. 

#### Durcir (un peu) le code PIN
Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il est quasi incassable dans des délais humains: pour être mnémotechnique il va certes s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. Mais de nombreux styles de saisie mènent au code PIN depuis la phrase `allons enfants de la patrie`: avec ou sans séparateurs, des chiffres au milieu, des alternances de mots en minuscules / majuscules, un mot sur deux, etc. La seule _bonne intuition_ d'un texte est loin de donner le code PIN correspondant.

> Un _login_ des appareils un peu conséquent et un code PIN _un peu durci_ constituent en pratique une barrière **très coûteuse** à casser. Tant qu'à être un _délinquant_ une forte pression directe sur Bob devrait permettre de lui extorquer ses phrases / PIN à moindre coût.

### Cas 2: mode _avion_, Internet n'est pas accessible
Bob ne peut pas utiliser son code PIN qui doit être validé par le module _safe server_ inclus dans l'application myApp1 server, NON joignable faute de disponibilité d'Internet.

Bob saisit sa phrase _longue_ p0 et p1 ou p2:
- le mode _safe terminal_ peut obtenir la clé K du _safe de Bob_ en décryptant K1 ou K2 depuis l'objet _profile_ lu de IDB identifié par `idprf`.
- il peut décrypter l'objet `safeData` donnant toutes les données du _safe de Bob_ relative à l'application myApp1.

L'application terminale myApp1 utilise alors la base de données IDB locale de nom idPrf contenant l'image de la dernière session _synchronisée_ ouverte par Bob sur ce device avec ce profil.

> L'accès est complètement sécurisé et est inviolable puisque protégé par une phrase longue: en contrepartie, accéder en mode _avion_ EXIGE la saisie de cette phrase longue, le code PIN ne peut pas être utilisé.

### La section _applications_ d'un _safe_
C'est une map avec une entrée par `appId`, identifiant d'une application que l'utilisateur peut lancer. C'est sa clé publique de cryptage C. La clé privée de décryptage D figure dans l'application distribuée où elle est, avec un peu d'efforts, possible à lire en _debug_.

Chaque entrée de cette map comporte trois sections:
- **droits**. C'est une map:
  - _valeur_: objet ayant les propriétés `{type, about, target, S}`. Cette valeur est cryptée,
    - par la clé publique C de cryptage de l'application,
    - puis par la clé K du _safe_.
  - _clé_: hash de `type, target`.
- **objets**: C'est une map:
  - _clé_: `hash(from + type + obj.id)`.
  - _valeur_:  `{ from, type, about, obj }`.
    - `from` est la clé publique de cryptage de l'expéditeur / déclarant. Si vide, cet objet est originaire d'une application opérant sous contrôle du _safe_ lui-même.
    - `type` : de l'objet `obj` (par exemple `PREF` ou `DROIT`).
    - `about` donne une explication _humaine_ à propos de `obj`.
    - `obj` est un objet crypté par la clé publique `C` de cryptage de l'application, puis par la clé `K` du _safe_ si `from` est vide.
- **liste noire**: c'est une map d'items:
  - _clé_: hash de la clé publique d'un _safe_ dont le _safe_ n'accepte plus de recevoir d'objets.
  - _valeur_: texte crypté par la clé `K` du _safe_ rappelant à son propriétaire qui il a mis en liste noire et pourquoi.

Le propriétaire d'un _safe_ A peut envoyer un _objet_ confidentiel au propriétaire d'un _safe_ B:
- la structure de l'objet dépend de son objectif, typiquement ce peut être,
  - un simple message textuel,
  - un _droit_ transmis de A à B que B pourra intégrer à ses droits.
- l'objet est accompagnée d'une propriété `about` fournissant un commentaire textuel court de A à l'intention de B.
- A peut s'envoyer un objet à lui-même, comme dans un _presse-papier_, confidentiel et persistant (la propriété `from` est absente dans ce cas).

## Opérations de l'application _Safe serveur_

#### Création d'un nouveau _safe_
Afin d'éviter une inflation incontrôlable de création de _safes_ fantômes, un utilisateur ne peut créer un _safe_ qu'après avoir reçu de la part d'une application un _code_ de validité limitée.

La base de données détient une liste des `SH(c)` des codes `c` en cours de validité avec leur date-heure limite de validité.

L'utilisateur est invité à saisir `code, p0, p1, p2, pseudo`.

L'application _Safe terminale_ génère les clé `K C D` et calcule les arguments suivants:
- `SH(code)` : code d'autorisation délivré par une application.
- `K1, K2, D_K, C_KH` 
- `SH(p0, p1), SH(p0, p2)`
- `pseudo_K`, `SH(pseudo) ?`

Exceptions:
- `p0` déjà utilisée pour un Safe existant.
- `pseudo` déjà utilisé.

#### Suppression d'un _Safe_
Arguments:
- soit `SH(p0, p1 (OU p2))`
- soit `C C_K` : l'identifiant du _safe_ et son cryptage par la clé K comme preuve de la détention de K acquis par exemple par le PIN.

Exceptions:
- Safe inexistant

#### Mise à jour / consultation d'un _Safe_
Le _safe_ est identifié, soit par `SH(p0, p1 (OU p2))`, soit par son `C C_K`. Les actions possibles sont:
- obtention de la section applications du _safe_.
- création, modification, suppression d'un ou plusieurs droits d'une application (import).
- suppression de tous les droits d'une application.
- modification de `p1` ou `p2` en donnant la valeur actuelle de `p0` et (`p1` ou `p2`).
- création, modification, suppression d'un objet d'une application (éventuellement issu d'un autre safe).

> L'application _Safe serveur_ N'A JAMAIS ACCÈS à aucune des clés de cryptage _en clair_: c'est un module de _stockage opaque_.

> Le module _Safe_ terminal est en charge des cryptages / décryptages et des interprétations: il est disponible en _source_ dans un browser en exécution et la validité de son source est vérifiable publiquement.

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
  - elle fait enregistrer par le _safe_ de l'utilisateur le couple `id, S`,
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
-lancement par _Safe_ d'une application, préférence, langue / mode sombre
