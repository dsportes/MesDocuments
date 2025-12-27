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
- `appli` : sauf rares exceptions pour des usages administratifs / techniques génériques, un droit est spécifique d'UNE application. Le code * indique un droit reconnu par toutes les applications.
- `org` : sauf exceptions pour certains droits d'administration technique, un droit est spécifique d'UNE organisation. Le code * indique un droit reconnu par toutes les organisations.
- `type`, un code correspondant à sa _classe / catégorie_ `cpt, mbr, trf ...`.
- **un droit correspond à une autorisations d'effectuer une ou des opérations** et non pas à identifier un utilisateur: toutefois l'opération _connexion d'un utilisateur_ revient de facto à une identification. Un droit est défini par a) la _cible_ des opérations qu'il autorise, b) la _source_, qui initie l'opération, c) les permissions, les catégories d'opérations qu'il autorise.
  - `target` : identifiant dans l'application de la _cible_ des opérations. Ce peut être aussi bien des données lisibles (une adresse e-mail, un numéro de mobile ...) qu'être le résultat d'une génération aléatoire. Par exemple pour un droit `cpt`, l'identifiant du compte.
  - `source` : identifiant dans l'application de l'entité qui enclenche l'opération. Un droit est relatif au couple _qui_ demande l'opération, sur _quoi/qui_ porte l'opération. Quand la source est aussi la cible, elle n'est pas donnée.
  - `perms` : `rwa` par exemple. Droit à effectuer les opérations `r` de lecture / consultation, `w` d'écriture / mise à jour, `a` d'administration. Les lettres sont spécifiques de chaque type de droit.
- `aes` : une clé _facultative_ de cryptage symétrique utilisée par les opérations usant de ce droit, par exemple la clé confidentielle d'un _login_ ou la clé cryptant les textes d'un chat associé à ce droit. 
- **Selon la technique d'authentification employée:**
  - un couple `SV` de clés:
    - `S` : clé privée de **signature** (environ 400 bytes).
    - `V` : clé publique de **vérification** (environ 100 bytes).
  - `hph` : le hash d'une _pass-phrase / mot de passe_.
    - si la pass-phrase est générée aléatoirement, le hsh peut être un SHA, sinon c'est un SH (PBKDF...).
    - l'opération vérifiant le droit doit s'assurer qu'il dispose en base de données du SHA (voire raccourci) de hph.
    - la pass_phrase n'est ainsi jamais stockée d'aucun côté.

La _clé_ identifiante d'UN droit est `[appli, org, type, target, source, perms]` (son SHA raccourci par exemple).

La _valeur_ d'un droit dépend de sa nature technique, par exemple `{aes, SV}` où 
`{aes, hph}`.

#### Cas d'un droit de nature _SV_
Les serveurs des applications ne détiennent des _droits d'accès_ de cette nature technique connaissent la clé `V` mais n'ont PAS (sauf exception décrite ci-dessous) accès à le clé `S` correspondante. Un _serveur_ vérifie la validité d'un droit transmis par _l'application terminale_ de la manière suivante:
- l'application terminale génère un texte _challenge_ qui n'a jamais été généré et ne sera jamais plus présenté à l'application serveur.
- elle _signe_ ce _challenge_ par sa clé `S` et transmet au serveur le couple du challenge et de sa signature.
- le serveur utilise sa clé `V` correspondante à la clé `S` utilisée à la signature et peut vérifier que la signature reçue est bien celle du challenge transmis.

> Cette technique permet au serveur de s'assurer de la validité d'un droit sans avoir eu en mémoire la clé `S` de signature: c'est un avantage de confiance par rapport aux solutions basées sur un mot de passe qui, à un moment ou à un autre, a besoin d'être présent dans la mémoire du serveur, même si un hachage fort (type PBKDF) de mots de passe longs limite le risque.

### Jetons d'accès aux opérations
Quand une application terminale soumet une opération à un serveur, elle fournit dans sa requête un **jeton d'accès** qui réunit les preuves que son utilisateur dispose des _droits_ requis pour exécuter cette opération. Un jeton comporte:
- `sessionId` : C'est l'identification d'UNE exécution l'application terminale sur UN _device_, à un instant donné une seule exécution terminale de l'application peut s'en prévaloir.
- `time` : date-heure en milliseconde de la génération du jeton d'accès. Cette donnée fait partie du _challenge_ des signatures du jeton.
- une liste de preuves de possession des droits constituée chacune de:
  - l'identifiant du droit: `[appli, org, type, target, source, perms]` (le contexte dispense de facto de transmettre `appli` et souvent `org`).
  - selon la nature technique du droit (chaque `type` a une nature technique associée):
    - `sign` : signature du couple `sessionId, time` par la clé `S` du droit.
    - `hph` : le hash ou _strong-hash_ de la pass-phrase `ph` obtenue d'une manière ou d'une autre de l'utilisateur.

**Remarques**:
- pour une application _web-push_, `sessionId` est un hash du (long) `devAppToken` attribué par le browser à l'application lors de l'enregistrement de son _service_worker_:  en conséquence `devAppToken` change si l'utilisateur du device supprime le service_worker qui se ré-enregistrera au prochain appel mais avec un token différent. Un _hacker_ un peu entraîné peut obtenir l'identifiant `devAppToken / sessionId` en lançant en _debug_ l'application sur ce _device_.
- un _jeton d'accès_ est crypté par la clé publique d'encryption du serveur applicatif ciblé de sorte que seul celui-ci puisse le lire. Cette clé fait partie de la _configuration_ du serveur que son administrateur technique délivre lors du déploiement et qu'il doit conserver confidentiellement.

### Validation des _droits_ d'un jeton SV (resp. PH) par le serveur de l'application
Quand le serveur d'une application traite une opération soumise par l'application _terminale_,
- il obtient de la requête le jeton d'accès et le décrypte par sa clé privée de décryptage.
- le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _droit_ de la liste du jeton,
  - il obtient depuis sa base de données, pour un couple `target, source, perms` une clé `V`, ou une liste de clés `V`, de vérification associée.
  - il vérifie que la `signature` du couple `sessionId, time` est bien validée par la, ou une des clés `V`, ce qui prouve que l'application terminale en détient effectivement la clé de signature `S` correspondante.

> `sessionId, time` est utilisé comme _challenge_ cryptographique et n'est pas présenté plus d'une fois pour une application donnée.

> Pour un couple donné target source, il est tout à fait normal de définir un droit pour une permission `r` de lecture et **un autre droit** pour la permission `wa` d'écriture et d'administration.

> Pour une authentification par _pass-phrase_ `ph`: au lieu de `sign` c'est `hph` (le hash ou `SH(ph)`) qui est passé sur le jeton par l'application et c'est le `SHA(hph)` qui est mémorisé par le serveur (et confronté au `hph` reçu de l'application).

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `target, source, perms` d'un `type` de droit la dernière liste de clés acceptées `[V1, V2 ...]` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `[V1, V2 ...]`, et en cas de changement refait une vérification avant de valider / invalider le droit correspondant.
- le serveur conserve en cache pour chaque `sessionId` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, leur _validation_ ne requiert qu'un calcul en mémoire sans accès à la base pour obtenir les clés de vérification.

### "Un" droit, "plusieurs" clés
Le serveur _peut_ mémoriser pour _un_ droit non pas _une_ clé `V` mais _une liste de clés_ `V1 V2 ...`: pour être validé, un droit d'un jeton d'accès doit fournir une signature qui a été établie par la clé `Si` correspondante à l'une des `[V1 V2 ...]`. 

Gérer un _changement de clé_ pour un droit attribué trop généreusement ou pour un temps limité s'effectue en créant une nouvelle clé dont la distribution est restreinte aux seuls détenteurs souhaitables dans le futur. Pendant un certain temps les deux peuvent être admises, puis _l'ancienne_ supprimée.

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

> Les modules _safe terminal / server_ ne gèrent pas des _personnes_ mais des _user_ ayant chacun un _coffre fort_: une _personne_ peut s'enregistrer sous plus d'un _user_, rien ne relient les _users_ entre eux ni à un quelconque signifiant dans le monde réel.

Après avoir lancé l'application _myApp1 terminal_ depuis son _device_, un utilisateur va lui indiquer quel est son _coffre fort_ afin d'accéder en toute sécurité aux données confidentielles qui le concerne.

### Sessions et _profils_ de sessions
Quand un utilisateur lance une application _myapp1_ depuis un _device_ il ouvre une session, identifiée de manière unique pour cette application: sur un _device donné_, une seule session peut s'exécuter à un instant donné pour l'application _myapp1_.

A la toute première ouverture d'une session de l'application _myapp1_ l'utilisateur dispose potentiellement d'une liste de droits déjà acquis antérieurement:
- chaque droit _peut_ être associé à une remontée important de données du serveur associé à son org.
- afin d'éviter une surcharge inutile pour le travail qu'il souhaite engager, l'utilisateur va restreindre cette liste en _cochant_ les seuls droits qui l'intéresse à cet instant. Ce faisant il définit ainsi un _profil_ de sa session.

Si l'utilisateur prévoit de ré-ouvrir un jour une session dans les mêmes conditions (mêmes droits), il peut enregistrer dans son _coffre_ le _profil_ de sa session et lui donner un _à propos_ significatif pour lui: ainsi ultérieurement quand il voudra ré-ouvrir une session similaire, au lieu de re-citer les droits qui l'intéressent, il désignera ce _profil_.

### _Préférences_ d'un utilisateur
Au cours d'une session d'une application _myapp1_, l'utilisateur peut fixer un certain nombre de _préférences_,
- la langue de travail,
- le mode clair ou foncé,
- page d'accueil souhaitée,
- des flags de présentation divers (portrait / paysage),
- des nombres de lignes d'affichages, etc.

Un _objet_ de préférence stocke ces paramètres.

Un utilisateur peut enregistrer quelques jeu de préférences en leur donnant un code comme `mobile tablette PC simple expert ...`, chaque jeu étant adapté à un couple désignant autant le profil technique optimisé pour un  type de _device_, qu'un mode d'utilisation.

En ré-ouvrant une session de l'application _myapp1_ l'utilisateur peut de cette façon utiliser,
- soit le jeu de préférences utilisé la fois précédente sur ce _device_,
- soit le jeu de préférences utilisé la fois précédente depuis un _device_ anonyme,
- soit choisir dans une courte liste celui qu'il préfère.

### Devices _de confiance_
Un utilisateur qui veut utiliser une application depuis un _device_ est placé devant deux cas de figure:
- **soit il n'a pas confiance dans ce _device_** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver des données.
- **soit il juge le device _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - les sessions qu'il y exécute peuvent laisser _en cache_ des informations cryptées et espérer raisonnablement les retrouver plus tard.

Un utilisateur peut déclarer sa _confiance_ au _device_ qu'il utilise:
- son _coffre_ enregistre ce _device_ comme étant de confiance,
- le _device_ enregistre localement la référence à cette déclaration de confiance.

Lancer une application depuis un appareil _de confiance_ a plusieurs avantages:
- **authentification simplifiée** de l'utilisateur en donnant un code PIN court (pour accéder à ses profils de sessions des applications et à ses _droits_).
- **disposer sur ce device de _mémoires caches persistantes et cryptées de documents_** pour chaque _profil_ de session ce qui lui permet d'ouvrir une session,
  - en mode _réseau_ en minimisant le nombre de documents à récupérer des serveurs,
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
- section `prefs`: liste de _jeux de préférences_ nommés par un code court, dans l'ordre anté-chronologique de dernière référence.

### Section `auth`

#### Création d'un _safe_ d'un utilisateur
Une clé AES `K` de 32 bytes est tirée aléatoirement: elle ne pourra pas changer et est la clé de cryptage du _safe_.

Un couple de clés C (cryptage - publique) / D (décryptage - privée).
- l'identifiant userId pour représenter l'utilisateur est le SHA raccourci de la clé C.
- la clé `C` est stockée en clair (elle est _publique_).
- la clé D est stockée crypté par la clé K dans `DK`.

L'utilisateur donne:
- un _couple_ `p0, p1` (qui pourra être changé) _d'authentification_:
  - `p0` est un pseudo / prénom-nom / adresse mail / numéro de téléphone / etc. qui identifie de manière unique le _safe_ (`hp0` le SH de `p0` est un index unique).
  - `p1` est une phrase _longue_ d'au moins 24 signes.
  - `hhp1` est le SHA de `SH(p1)`.
- un _couple_ `r0, r1` (qui pourra être changé) _de récupération_:
  - `r0` est un pseudo / prénom-nom / adresse mail / numéro de téléphone / etc. (12 signes au moins) qui identifie de manière unique le _safe_ (`hr0` le SH de `r0` est un index unique) et qui peut être égal à `p0`.
  - `r1` est une phrase _longue_ d'au moins 24 signes. Il n'est pas judicieux qu'elle soit égale à p1 puisqu'elle permet justement la récupération du safe en cas d'oubli de `r0, p0`.
  - `hhr1` est le SHA de `SH(r1)`.
- `pseudo`: un nom court compréhensible par les propriétaires des _devices_ de confiance, par exemple `Bob`.

La clé `K` du safe est stockée,
- dans `Ka` et `Kr` cryptages respectifs par  `SH(p0, p1)` et `SH(r0, r1)`.
- `hhk` : SHA du `SH(K)` permettant au module _safe server_ de vérifier sur chaque opération demandée par _safe terminal_ que celui-ci détient bien la clé K (transmise par `SH(K)`).

A aucun moment les propriétés `p0 p1 r0 r1` ne sont ni stockées ni transmises _en clair_: elles ne sont _lisibles_ que très temporairement lors la saisie par l'utilisateur dans le module _safe terminal_ et cryptées dès la fin de la saisie.

Pour changer `p0, p1` et/ou `r0, r1` l'utilisateur doit fournir,
- soit le couple actuel `p0, p1` OU `r0, r1`.
- les nouveaux couples `p0, p1` et `r0, r1`. 

#### Synthèse des propriétés de la section `auth`
- `userId` : identifiant.
- `lam` : dernier mois d'accès YYYYMM au _safe_: toute utilisation recule cette date qui permet une _purge_ périodique des _safe_ obsolètes / fantômes.
- `C` : clé de cryptage en clair (`userId` est son SHA raccourci).
- `DK` : clé de décryptage cryptée par la clé `K`.
- `hp0` : index unique, `SH(p0)`.
- `hr0` : index unique, `SH(r0)`.
- `hhp1` : SHA de `SH(p1)`.
- `hhr1` : SHA de `SH(r1)`.
- `hhk` : SHA de `SH(K)`.
- `Ka` : clé `K` du safe cryptée par `SH(p0, p1)`.
- `Kr` : clé `K` du safe cryptée par `SH(r0, r1)`.
- `pseudo` : pseudo crypté par la clé K du _safe_.

### Section `devices`
Chaque _device de confiance_ à une entrée  dans cette section identifiée par `devid` (un identifiant généré aléatoirement):
- `about` : code / texte court **crypté par la clé K du _safe_** donné par l'utilisateur pour qualifier le _device_ (par exemple `PC d'Alice`).
- `{ Va, cy, sign, nbe }` : propriétés permettant de valider que ce _device_ est de confiance (voir plus loin).

Après avoir authentifié son accès à son _safe_, l'utilisateur peut retirer sa confiance à n'importe lequel des devices cités dans la liste en en supprimant l'entrée.

### Section `creds`
Chaque _droit d'accès / credential_ est enregistré dans un item **crypté par la clé K**. Ses propriétés sont:
- `about` : code / texte court donné par l'utilisateur pour qualifier le _credential_. Par exemple `Compte Bob sur circuits courts`. 
- `appli, org, type, target, keys: {}` : données du _credential_, ses clés d'accès. La structure de keys dépend de la nature technique du _credential_ utilisé.
- son **identifiant** est le hash _court_ de `[appli, org, type, target]`.

### Section `profiles`
Elle est organisée avec une **sous-section par application** regroupant une liste d'items ayant un identifiant généré aléatoirement à sa création. Chaque item est **crypté par la clé K** de _safe_ et a les propriétés suivantes: 
- `about`: texte significatif pour l'utilisateur **crypté par la clé K** décrivant le _profil_ d'une session (par exemple `Revue des notes d'Alice et Jules`).
- `creds`: la liste des id des _credentials_ qui sont attachés à une session de ce profil lors de son ouverture.

### Section `prefs`
Elle est organisée avec une **sous-section par application** donnant une liste de couples `code, pref` ( **cryptés par la clé K**) ordonnée par dernière utilisation:
- `code` : texte court parlant pour l'utilisateur correspondant à un de ses usages habituels de l'application comme `mobile, large, simple, expert ...`.
- `pref`: un objet donnant les valeurs des _préférences_ à utiliser à l'ouverture d'une session.

## Accès d'une application terminale à un _safe_
### Depuis n'importe quel _device_ (de confiance ou non)
Le module _safe terminal_ demande à l'utilisateur `p0 p1` (ou `ro, r1`) et transmet `SH(p0) SH(p1)` au module _safe server_ qui,
- accède au document _safe_ depuis le `SH(p0)` (index unique).
- vérification que `hhp1` est bien le hash court de `SH(p1)` reçu en argument.
- retourne `Ka Kr`: le module _safe terminal_ décode `Ka` par `SH(p0, p1)`. En cas d'échec c'est que `p0 / p1` était incorrect.

### Depuis un _device_ de confiance
Un device qui a été déclaré _de confiance_ par au moins un utilisateur a une micro base de données IDB nommée `Safes` ayant les tables suivantes:
- `HEADER`: cette table _singleton_ a deux colonnes:
  - `devId`: un identifiant généré aléatoirement à la création de la base _Safes_ identifiant le _device_.
  - `devName`: le _nom_ du _device_, par exemple `PC d'Alice`, plus parlant que le code technique système pour le propriétaire du _device_ et les quelques personnes pouvant l'utiliser en confiance.
- `TRUSTING`: chaque row est associé à UN _safe_ ayant déclaré le _device_ de confiance. Il a les colonnes suivantes:
  - `userId`: identifiant de l'utilisateur (clé primaire).
  - `pseudo`: par exemple `Bob`.
  - `cx`: un challenge aléatoire.
  - `Ka`: clé K du safe de l'utilisateur cryptée par `SH(p0, p1)` où `p0` et `p1` sont les termes d'authentification du safe de l'utilisateur.
  - `Kr`: clé K du safe de l'utilisateur cryptée par `SH(r0, r)`.
  - `Kp`: clé K du safe de l'utilisateur cryptée par `SH(PIN + cx, cy)` où,
    - `PIN` est le code PIN fixé par l'utilisateur à la déclaration de confiance,
    - `cx cy` sont des _challenges_ générés aléatoirement à ce moment.
- `SESSION`: chaque row décrit une _session_ qui a été ouverte _en confiance_ sur ce _device_:
  - `app`: code l'application correspondante.
  - `userId`: identifiant de l'utilisateur.
  - `profId`: id du profil de la session.
  - `profAbout`: texte significatif pour l'utilisateur **crypté par la clé K du _safe_** décrivant le _profil_ de la session (par exemple `Revue des notes d'Alice et Jules`).
  - `size`: volume _utile_ des données de la base IDB lors de la dernière session ouverte sur ce _device_.
  - `time`: dernière date-heure d'ouverture de cette session sur ce terminal.
  - Il existe une base de données IDB de nom `app.x` (`x = hash court de (userId / profId)`)contenant les documents en cache de cette session.
- `PREFS` : chaque row décrit pour une _session_ (`app, userId`) qui a été ouverte _en confiance_ sur ce _device_:
  - `app`: code l'application correspondante.
  - `userId`: identifiant de l'utilisateur.
  - `[code, pref]`: le code et les valeurs de préférences utilisées lors de la dernière session ouverte (de manière à les retrouver par défaut la prochaine fois). `pref` est crypté par la clé K de l'utilisateur.

> Les rows de la base IDB Safe sont cryptés par une clé AES du module _safe terminal_ afin de ne pas être directement lisible en _debug_: cette _sécurité_ est _molle_, la clé étant d'une manière ou d'une autre inscrite dans le code, avec un peu de fatigue un hacker motivé peut la retrouver.

#### Déclaration d'un _device_ de confiance
Depuis le _device_ à déclarer de confiance, l'utilisateur:
- saisit son `pseudo` et `devName` le nom qu'il donne à ce _device_: les valeurs par défaut sont proposées, par exemple `Bob` et `PC d'Alice`.
- saisit un code `PIN` (d'au moins 6 signes).
- saisit le couple `p0 p1` d'accès à son _safe_.

Le module _safe terminal_ demande au module _safe server_ d'accéder au safe de l'utilisateur identifié par `SH(p0)` et de lui retourner le `userId` et `Ka` associé:
- disposant du couple `p0 p1`, le module _safe terminal_ obtient la clé `K` du safe de l'utilisateur en décryptant `Ka` par le `SH(p0, p1)`.

Le module _safe terminal_,
- génère aléatoirement `devId` si cette donnée ne figure pas encore dans le `HEADER`.
- génère les challenges aléatoires `cx cy`.
- calcule `Kp`, cryptage de cryptage de la clé `K` par le `SH(PIN + cx, cy)`.
- génère un couple `Sa Va` de clés asymétriques signature / vérification.
- calcule `sign`, signature par `Sa` du `SH(PIN, cx)`.
- calcule `sh1p / sh1r` comme `SH(p1) / SH(r1)`.
- enregistre dans la table `TRUSTING` de la base IDB `Safes` un row avec les colonnes `userId pseudo cx Ka Kr Kp`.
- transmet au module _safe terminal_ `userId, devId, sh1p, sh1r, devName(crypté par K), Va, cy, sign` qui,
  - accède au _safe_ dont l'id est `userId` et vérifie que `hhp1 / hhr1` est bien le SHA de `sh1p / sh1r` (s'assure que _safe terminal_ détient le bon `p1 / r1`).
  - y créé dans la section `devices` une entrée `devId` avec les données `devName Va cy sign nbe = 0`.

> Remarque: `Sa` a servi à générer la signature `sign` mais n'est plus utilisé ensuite et n'est pas mémorisé alors que `Va` l'est et servira à authentifier la signature d'un PIN saisi par l'utilisateur.

Après ce calcul,
- le _safe_ a été mis à jour par le module _safe server_ avec un nouveau device de confiance avec les données cryptographiques permettant à l'utilisateur de s'authentifier par un code PIN.
- sur le _device_ la base locale IDB _Safes_ contient une entrée relative à ce _safe_ avec en particulier la clé K du _safe_ cryptée en `Ka` et `Kp`. 

#### Authentification par code PIN depuis un _device déclaré de confiance_
Le module _safe terminal_ lit la base IDB _Safes_ et, 
- propose à l'utilisateur de désigner la ligne de `TRUSTING` dont la propriété `pseudo` (par exemple `Bob`) lui correspond. Le module dispose ainsi des données `userId cx Kp`.
- demande à l'utilisateur de saisir le PIN associé et calcule `z = SH(PIN, cx)`.
- transmet au module _safe terminal_ `userId, devId, z` qui,
  - accède au _safe_ dont l'id est `userId`.
  - accède dans la section `devices` à l'entrée `devId` ce qui lui donne les propriétés `Va cy sign nbe`. Si cette entrée n'existe pas c'est que le _device_ N'EST PAS / PLUS de confiance pour ce _safe_,
    - soit n'a jamais été déclaré comme tel,
    - soit la confiance en lui a été retirée explicitement par l'utilisateur,
    - soit qu'il a été supprimé du fait d'un nombre excessif d'essai erroné de code PIN.
  - vérifie par `Va` que `sign` est bien la signature de `z`. En cas de succès, il met à 0 `nbe` s'il ne l'était pas déjà et sinon incrémente `nbe`.
  - retourne le challenge `cy` au module _safe terminal_ qui peut ainsi calculer la clé `SH(PIN + cx, cy)` qui décrypte `Kp` ce qui lui donne la clé K du _safe_.

##### Échecs
SI la signature `sign` n'est pas vérifiée par `Va`, c'est que le code PIN n'est pas le bon. `Va` correspond au `Sa` qui a été utilisé à sa signature, `cx` était bien celui fixé à la déclaration. **Le nombre d'erreurs `nbe` est incrémenté**.

Si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais, l'entrée `devId` est supprimée. L'utilisateur devra refaire une _déclaration de confiance_ de ce device avec un code PIN (ce qui exigera une authentification _forte_ de sa part par `p0` et `p1`).

### Accès d'une application en mode _avion_ (pas d'accès au réseau)
La table `SESSION` de la base IDB _Safes_ permet de lister les sessions qui ont été ouvertes sur ce _device_ pour cette application avec pour chacune,
- le texte `profName` de son profil, par exemple `Revue des notes d'Alice et Jules`,
- pseudo du _safe_ correspondant, par exemple `Bob`.

L'utilisateur désigne la session qu'il souhaite ré-ouvrir ce qui lui donne:
- le `userId` de cette session,
- le `profId` du profil de cette session,
- `Ka` la clé K de ce _safe_ mais cryptée par `p0 p1` d'authentification du _safe_.
- `pref` les valeurs de _préférences- de la session cryptées par la clé K.
- le nom de la base IDB cache des documents.

L'utilisateur saisit son couple `p0 p1` pour obtenir sa clé K:
- le succès du décryptage authentifie sa propriété du _safe_.
- ses préférences d'ouverture de la session sont décryptées et sa base IDB est lisible.
- la session peut être ouverte, en lecture seulement.

> En mode _avion_ l'authentification par code PIN n'est pas possible.

### Sécurité de l'authentification par code PIN depuis un _device de confiance_
Sur un device NON déclaré de confiance l'utilisateur doit fournir un couple p0 p1 ou p0 est un pseudo / nom / etc et p1 une phrase longue, soit une bonne trentaine de signes ce qui est considéré comme inviolable par force brute avec un minimum de précaution dans le choix de p1.

Sur un device déclaré de confiance par l'utilisateur il _suffit_ d'un code PIN de 8 signes (ou plus), donc _a priori_ beaucoup plus facile à craquer. Mais, 
- en l'absence de piratage technologique, l'utilisateur a droit à deux essais infructueux, le code PIN s'auto-détruisant passé ce seuil. Les essais multiples sont voués à l'échec.
- le code PIN est spécifique de l'utilisateur ET de chacun des devices qu'il a déclaré de confiance (sauf s'il donne toujours le même): il faut s'être connecté (par login _système_) sur un device déclaré de confiance préalablement pour pouvoir l'utiliser.

Ceci veut dire que la _vraie_ sécurité repose sur,
- la connaissance d'un compte de login du device, ce qui déjà sensé être particulièrement protégé,
- le fait que le device ait été préalablement déclaré de confiance et protégé par un code PIN,
- le fait qu'au delà du second essai infructueux, la confiance dans ce device est retirée.

Le **code PIN** n'est jamais stocké ni passé en clair sur le réseau au module _safe server_: 
- il ne peut pas être détourné ou être lu depuis la base de données.
- il ne figure que temporairement en mémoire dans le module _safe terminal_ inclus dans l'application _myApp1 terminal_ durant la phase d'authentification du _safe_ de l'utilisateur.

Pour tenter depuis les données du _Safe server_ d'obtenir le code PIN par force brute, il faut effectuer une vérification de `sign` par `Va` avec le _challenge_ `SH(PIN, cx)` mais `sign` est crypté par la clé privée de cryptage général du module _safe server_.

Pour que cette dernière attaque pour trouver le PIN de `Bob` par force brute ait des chances de succès, il faut que le hacker ait obtenu frauduleusement:
- (1) le contenu en clair de l'objet _safe_ en base et pour cela il lui faut conjointement,
  - avoir accès à la base en lecture ce qui requiert, soit une complicité auprès du fournisseur de la base de donnée, soit **la complicité de l'administrateur technique**.
  - avoir la clé de décryptage des contenus de celle-ci inscrite dans la configuration de déploiement des serveurs. Ceci suppose la **complicité de l'administrateur technique** effectuant ces déploiements.
- (2) ait obtenu le challenge `cx` stocké dans la base IDB _Safes_ du device ce qui suppose,
  - d'avoir une session ouverte sur le device (mot de passe du login sur un PC, sur un mobile avoir le mobile _déverrouillé_).
  - d'ouvrir une application pour pouvoir lire en _debug_ la base de données IDB _Safes_,
  - de retrouver toujours en _debug_ la clé de décryptage de cette base (quicertes plus ou moins _cachée_, figure dans le code).

Ayant obtenu le challenge `cx`, il faut ensuite écrire une application dédiée pour craquer par force brute le code PIN en tentant la vérification de signature `sign` par la clé `Va` du challenge SH(PIN, cx).

> Ce double _piratage / complicité_ donne accès à la clé `K` du _safe_ de `Bob`, donc au contenu du _safe_. Toutefois `p0 p1 r0 r1` restent inviolées et non modifiables par le hacker, puisque ne résidant que dans la mémoire de l'utilisateur.

> Cracker le code PIN d'un _device de confiance_ de l'utilisateur Bob ne compromet pas les autres utilisateurs.

> Pour craquer **tous** les codes PIN, il faudrait pouvoir accéder à tous les appareils de confiance **déverrouillés / sessions ouvertes** et casser par force brute le PIN de _chaque safe pour chaque device_. 

#### Durcir (un peu) le code PIN
Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il est quasi incassable dans des délais humains: pour être mnémotechnique il va certes s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. Mais de nombreux styles de saisie mènent au code PIN depuis la phrase `allons enfants de la patrie`: avec ou sans séparateurs, des chiffres au milieu, des alternances de mots en minuscules / majuscules, un mot sur deux, etc. La seule _bonne intuition_ d'un texte est loin de donner le code PIN correspondant.

> Un _login_ des appareils un peu conséquent et un code PIN _un peu durci_ constituent en pratique une barrière **très coûteuse** à casser. Tant qu'à être un _délinquant_ une forte pression directe sur Bob permet en général de lui extorquer ses phrases / PIN à moindre coût 😈.

# Opérations du module _Safe server_

### Création d'un nouveau _safe_

#### Changement des clés `p0 p1 r0 r1`

### Suppression d'un _safe_

### Purge périodique des _safes_ inutilisés / obsolètes

### Clé publique de cryptage d'un _safe_
- depuis son `userId`.

Soit deux utilisateurs A et B, ayant chacun leurs clés publique Ca et Cb et leurs clés privées Da et Db:
- B peut écrire un texte secret T à destination de A en utilisant la clé AES construite depuis Ca et Db. Pour obtenir Ca le userId de A est suffisant.
- A peut décrypter le texte T à condition de savoir que B en est l'auteur. Il obtient la même clé AES que celle utilisée pour crypter T depuis Cb et Da. Pour obtenir Cb le userId de B est suffisant.
- le texte T _peut_ (sans risque) être transmis accompagné de Cb (qui donne le userId de B) et permet au destinataire A de décrypter immédiatement T.

### _login_ à un _safe_
- par `SH(p0) SH(p1)` -> `userId, K`a -> `K`
- par `userId, devId, SH(PIN, cx)` -> `cy` -> `K` décrypté par `SH(PIN + cx, cy)`

### Extractions d'un _safe_
- liste des devices de confiance
- liste des droits
- liste des applications ayant au moins un profil de session
- liste des profils de session pour une application

### Déclaration de confiance d'un _device_ 
Le changement de PIN correspond à une re-déclaration.

### Suppression de confiance d'un _device_

### Settings des droits et des profils de session
- création d'un profil
- ajouts / retraits de droits, attribution / retrait à des profils
- modification du texte _about_

### Settings de préférences

# Questions ouvertes

### Transférer / acquérir des _droits_
Comment transférer / acquérir un _droit_ comme _Comptable de asocial/demo_ ouvrant la possibilité à un utilisateur d'agir avec un rôle de _Comptable_ pour l'organisation _demo_ dans l'application _asocial_ ?
- attribution directe à un _safe_ par son détenteur actuel? Depuis un identifiant externe (p0 ?) ...
- dépôt du droit dans un _clipboard_ identifié par une phrase secrète de durée de vie limitée échangée hors application.
- cryptage: couple de clés C / D par safe.

### Phrase de contact temporaire
Un user A peut se déclarer une _phrase de contact temporaire_ (p0, p1) indexée (unique) par SH(p0) s'autodétruisant au bout de quelques jours: ainsi B peut _poster_ un message (par exemple avec un _droit_ attaché) à A, sans avoir à connaître son userId mais seulement une _phrase_ compréhensible.

Si A ne veut plus être dérangé par B, il supprime sa phrase temporaire.

**Plusieurs** _phrases temporaires_ ?

### Copier / coller des _droits_ entre _safes_
En partie une solution à la question précédente, ce dispositif permet aussi de changer de _safe_, de distribuer des droits sur deux autres _safes_, etc.
- identifier le ou les _safes_ cibles.
- les droits copiés sont-ils automatiquement valides dans les safe cibles ou doivent-ils être confirmés ? Changent-ils d'id ?
- dans ce cas il faut une clé C et une clé D par _safe_ : la clé D est-elle la clé K ?

### Utilisation d'un _userId_ comme identifiant d'un _compte_ dans une application
Dans _myApp1 server_ authentifie `userId SH(K)` en faisant un appel interne au module _safe server_ qui peut garder en cache les couples authentifiés les plus récents.

### Comment éviter une inflation incontrôlable de création de _safes_ fantômes
Un utilisateur ne pourrait créer un _safe_ qu'après avoir obtenu un ticket d'invitation déposé par un autre _safe_.
- le ticket a une durée de vie limitée.
- le code du ticket parvient par un moyen externe (mail ...).
- le nombre de tickets généré par un safe est limité (N par mois / an ...).

## Processus d'acquisition de droits par un utilisateur

### Acquisition directe dans l'application terminale
Dans cette situation c'est l'application terminale qui génère le _droit_.

Par exemple _création d'un "compte" dans l'application_:
- l'application terminale a généré l'`id` du compte ou l'a obtenu de son serveur et a généré un couple de clés `S / V`.
- elle fait enregistrer par le _safe_ de l'utilisateur un _droit_ avec le couple `id, S`,
- elle _valide_ la création du compte auprès du serveur de l'application pour qu'il enregistre en base le couple `id / V`.

### Obtention par l'application terminale d'un droit _configuré_ côté serveur
Un certain nombre de droits peuvent être _configurés_ dans le serveur de l'application, 
- soit _en dur_ dans le code, 
- soit _en base_ par un administrateur: ces droits sont chargés (et _cachés_) à première demande.

> Les `S` de ces droits ne sont pas lisibles directement en base afin qu'un détournement de celle-ci ne donne pas accès à ces clés mais sont dans des objets cryptés par une clé fixée par l'administrateur technique. 

Dans ce cas, l'application terminale récupère depuis le serveur le couple `id, S` du droit à attribuer à l'utilisateur et le fait enregistrer comme droit dans le _safe_ de l'utilisateur. 

### Attachement à un autre droit
Le serveur dispose pour un droit `dx` non pas d'une clé `S` mais d'une liste d'autres droits `d1, d2 ...` Par exemple:
- `dx` est un droit de gestion d'un tarif,
- `d1 d2 ...` sont les logins à qui ce droit a été attribué.

Pour exécuter une opération requérant le droit `dx`, il suffit que le _jeton_ ait un des droits `d1 d2 ...`
