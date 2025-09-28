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

### Droit d'une application
Un _droit_ d'une application est matérialisé par les données suivantes:
- `type`, un code correspondant à sa _classe / catégorie_ `cpt, mbr, trf ...`
- `target` : l'identifiant dans l'application de sa _cible_ qui peut comporter aussi bien des données lisibles (une adresse e-mail, un numéro de mobile ...) qu'être le résultat d'une génération aléatoire. Par exemple pour un droit `cpt` l'identifiant du compte.
- une _liste de clés_, souvent d'un seul terme:
  - `var` : un code (vide par défaut) permettant de disposer de plusieurs clés pour un même droit identifié par `type / target`.
  - `S` : clé privée de **signature** (environ 400 bytes).
  - `V` : clé publique de **vérification** (environ 100 bytes).

Le serveur d'une application gère les _droits d'accès_ mais n'en connaît **QUE** `type target V` et n'a pas (sauf exception décrite ci-dessous) accès à le clé `S` correspondante. Le _serveur_ vérifie la validité d'un droit transmis par _l'application terminale_ de la manière suivante:
- l'application terminale génère un texte _challenge_ qui n'a jamais été généré et ne sera jamais plus présenté à l'application serveur.
- elle _signe_ ce _challenge_ par sa clé `S` et transmet au serveur le couple du challenge et de sa signature.
- le serveur utilise sa clé `V` correspondante à la clé `S` utilisée à la signature et peut vérifier que la signature reçue est bien celle du challenge transmis.

> Cette technique permet au serveur de s'assurer de la validité d'un droit sans jamais avoir eu en mémoire la clé `S` de signature: c'est un avantage de confiance par rapport aux solutions basées sur un mot de passe qui, à un moment ou à un autre, a besoin d'être présent dans la mémoire du serveur, même si un hachage fort de mots de passe longs limite le risque.

### Jetons d'accès
Quand l'application terminale soumet une opération au serveur, elle fournit dans sa requête un **jeton d'accès** qui réunit les preuves que son utilisateur dispose des _droits_ requis pour exécuter cette opération. Un jeton comporte:
- `sessionId` : C'est l'identification d'UNE exécution l'application terminale sur UN _device_, à un instant donné une seule exécution terminale de l'application peut s'en prévaloir. Pour une application fonctionnant en web-push, `sessionId` est un hash du (long) `devAppToken` attribué par le browser à l'application.
- `time` : date-heure en milliseconde de la génération du jeton d'accès. Cette donnée fait partie du _challenge_ des signatures du jeton.
- une liste de preuves de possession des droits constituée chacune d'un triplet:
  - `type` : du droit,
  - `target` : _identifiant_ de la cible à laquelle le droit s'applique,
  - `signatures` : liste de signatures `{var1: sig1, ...}` du couple `sessionId,  time` par chaque variante `var1` de la clé `S` du droit correspondant.

**Remarques**:
- un _hacker_ un peu entraîné peut obtenir l'identifiant `devAppToken` en lançant en _debug_ l'application sur ce _device_.
- dans la liste des preuves, plusieurs peuvent concerner le même couple `type target`: il suffit que l'une d'elle soit reconnue comme valide pour que le droit le soit. C'est une manière de traiter les situations, temporaires ou non, où un même droit peut avoir plusieurs _variantes_ de clés de signature / vérification (une _ancienne_ et une _nouvelle_, une variante par _pseudo_ ...).
- un _jeton d'accès_ est crypté par la clé publique d'encryption du serveur applicatif ciblé de sorte que seul celui-ci puisse le lire. Cette clé fait partie de la _configuration_ du serveur que son administrateur technique délivre lors du déploiement et qu'il doit conserver confidentiellement.

#### Solutions: simpliste versus _Safe_
Quand l'application terminale doit soumettre une opération requérant un _droit_ donné (par exemple `cpt` pour un compte `1234`), elle pourrait en obtenir la clé `S` en la demandant à l'utilisateur. 

Ce dernier effectue une forme ou une autre de _copier / coller_ depuis par exemple un fichier de texte externe détenu sur son _device_ et lui affichant sur une ligne la clé `S` pour le compte `1234`.

Cette solution _simpliste_ est difficile à gérer dans la pratique:
- comment l'utilisateur gère la disponibilité sur plusieurs appareils ?
- comment éviter l'écrasement partiel ou perte du fichier lors d'une mise à jour ?
- comment _sécuriser_ un tel fichier contre le vol ?

Un autre solution est de stocker ces _droits_ par une base de données _Safe_ gérée par un couple de modules, _safe terminal_ embarqué dans _myApp1 terminal_, _safe server_ embarqué _myApp1 server_ en charge de résoudre les questions ci-dessus et de rendre accessible les _droits_ aux applications qu'ils concernent.

#### Les applications terminales ne doivent pas avoir été _piratées_
Si l'application terminale est une application _pirate_ qui a été lancée en déjouant la vigilance de l'utilisateur (par exemple en le sollicitant d'appuyer sur un lien envoyé par un e-mail frauduleux), elle _peut_ disposer des clés des droits auxquels l'utilisateur lui a donné accès. 

Elle _peut_ les envoyer sur un serveur pirate où elles seront à disposition de pirates pour utilisation dans la _vraie_ application et lui transmettre ces clés _usurpées_.

L'application _légitime_ a dans son code _sa clé de décryptage privée_ qui lui permet de décrypter les droits: mais en exécution en mode _debug_ un hacker un peu habile peut la retrouver. Cette _sécurité_ est plus symbolique qu'effective.

Il n'existe aucun procédé logiciel qui permette de connaître l'origine d'une application, de quelle _source_ elle vient, si elle est _légitime_ ou _pirate_.

Depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle de l'application pirate,
- s'être assuré que le CDN correspondant distribue bien le source _officiel_ et non pas un _source modifié_. Pour cela il faut,
  - comparer le hash des fichiers sources distribués avec les hash des fichiers source du repository _officiel_,
  - avoir obtenu d'un expert indépendant l'assurance que le code _officiel_ est bien légitime et ne redistribue pas de clés d'accès,

> Ces conditions sont toutefois possibles à vérifier pour une _application terminale_ Web. En revanche il n'existe aucun procédé technique permettant à un utilisateur de savoir si l'application _serveur_ hébergée est bien celle dont les sources (en Open Source) seraient disponibles dans un repository public: il faut _faire confiance_ à l'hébergeur ... et avoir une conception globale qui ne transmet pas d'informations sensibles en clair au serveur. 

### Demandes de droits par une application terminale
En supposant que l'application terminale ait obtenu, en sécurité, la **liste des droits** de l'utilisateur, lorsque celle-ci recherche la clé `S` d'un _droit_, plusieurs situations se présentent:
- (1) pour le `type` fixé (par exemple `DRTARIF` : _droit à effectuer une modification tarifaire_), il n'existe qu'une clé unique, `target` est vide. Un seul terme de la liste des droits peut s'appliquer.
- (2) le `type` ET la valeur de `target` sont fixées (par exemple `LOGIN, 1234` : _droit de l'utilisateur 1234 à se connecter_) par l'application. Un seul terme _au plus_ de la liste des droits peut s'appliquer.
- (3) le `type` est fixé MAIS PAS la valeur de `target` (par exemple `LOGIN`):  l'application recherche **à la fois une cible et sa clé**. Plusieurs droits de la liste peuvent être candidats et l'utilisateur devra **désigner** le login qu'il choisit: dans ce cas un commentaire `about` attaché à chaque droit lui est utile (par exemple en donnant un nom en clair plutôt qu'un code).

### Validation des _droits_ d'un jeton par le serveur de l'application
Quand le serveur de l'application traite une opération,
- il obtient de la requête le jeton d'accès et le décrypte par sa clé privée de décryptage.
- le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _droit_ de la liste du jeton,
  - il obtient depuis la base de données la map dés clés acceptées `{var1: V1, var2:v2 ...}` de vérification associée à sa cible,
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

# Les modules _safe terminal_ et _safe server_

Ils sont embarqués respectivement dans _myApp1 terminal_ et _myApp1 server_: les deux modules communiquent entre eux, le _terminal_ pouvant sollicité des opérations du _server_ par des requêtes HTTPS.

Ils ont pour objet de gérer le (ou les) _coffres forts_ des utilisateurs.

Après avoir lancé l'application _myApp1 terminal_ depuis son appareil, un utilisateur va lui indiquer quel est son _coffre fort_, du moins la partie de son _coffre fort_ dédiée à myApp1 ait accès aux diverses données _sensibles_ de l'utilisateur (ses _droits d'accès_, préférences, options de lancement).

Une fois ainsi initialisé, par _myApp1 terminal_, le module embarqué _safe terminal_ peut:
- gérer ses profils sur des appareils _de confiance_.
- gérer les données sensibles _droits, préférences ..._ de l'application que l'utilisateur peut utiliser.

> Les modules _safe terminal / server_ ne gèrent pas à proprement parler des _utilisateurs_ mais leurs _coffres forts_: rien n'empêche un _utilisateur_ (une personne) de posséder plus d'un coffre, rien ne relient les coffres entre eux ni à un quelconque signifiant dans le monde réel.

> Dans la base de données _safe_ chaque coffre fort à une entrée:
- avec un **header d'authentification** (indépendant des applications) et une clé K immuable pour le _safe_ cryptée.
- une **section d'authentification par application** `myApp1` ayant une entrée par _profil_ `idPrf1` déclaré par l'utilisateur cryptée par la clé C de _myApp1_.
- une **section par application**: la section de _myApp1_ est doublement cryptée,
  - par la clé C de _myApp1_: les autres applications ne peuvent pas les décrypter,
  - par la clé K du _safe identifié_ de sorte que la section n'est lisible que par une application _myApp1_ ayant acquis la clé K du _safe_ depuis la saisie de l'utilisateur soit d'une phrase longue, soit d'un code PIN dans certaines conditions.

## _Profils_ des utilisateurs sur des appareils déclarés _de confiance_
Un utilisateur qui veut utiliser une application depuis un _device_ est placé devant deux cas de figure:
- **soit il juge l'appareil _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche: il en a le _login_ ou a minima quelqu'un accepte de lui prêter l'appareil _session ouverte_.
  - il peut y laisser quelques informations cryptées et espérer raisonnablement les retrouver plus tard.
- **soit il n'a pas confiance dans cet appareil** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver des données.

**Sur un appareil qu'il a jugé de confiance**, un utilisateur Bob peut déclarer un, voir plusieurs, **profils**. 

Pour Bob initialiser sa session de l'application _myApp1_ depuis un _profil_ préalablement déclaré sur cet appareil a plusieurs autres avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant une petite base de données locale (cryptée) pour chaque profil comme _cache_ de ses documents: ceux qui y figurent et à jour n'auront pas besoin d'être demandés au serveur de l'application.
- **disponibilité des droits** de Bob pour _myApp1_ le dispensant de s'en souvenir ou de les copier / coller d'un support externe.
- **possibilité d'utiliser _myApp1_ en mode _avion_** sans accès au réseau en utilisant les documents et les droits en _cache_.

> Même _de confiance_ un appareil _peut_ être utilisé par d'autres que soi-même, même dans un cadre familial ou de couple, l'appareil n'est jamais considéré comme strictement _personnel_.

## Stockage des _safes_

### La base de données _Safe_
Elle stocke les données de chaque _safe_. Elle est accédée par le moule _safe server_ embarqué dans les applications serveur comme _myApp1 server_.

L'accès au _safe_ de Bob par _myApp1 terminal_ est effectué par le module _safe terminal_ embarquée. L'authentification du _safe_ de Bob se fait par de ces deux procédés:
- depuis tout appareil, la donnée des phrases _longues_ `p0` et `p1` (ou `p2`) connues uniquement de Bob.
- depuis un appareil _de confiance_ sur lequel Bob a déclaré un _profil_ `prf1` la donnée d'un code PIN (au moins 8 signes) beaucoup plus court.

Après avoir identifié et authentifié en sécurité le _safe_ de Bob, l'application _myApp1 terminal_ sollicite son module _safe terminal_ pour qu'il communique avec le module _safe server_ embarqué dans _myApp1 server_ et reçoive sa _section spécifique de myApp1 pour Bob, et la charge en mémoire. L'application _myApp1 terminal_ dispose ainsi des droits d'accès, préférences et options de Bob pour _myApp1_.

Les mises à jour du _safe_ de Bob se répercutent sur la base _safe_ par l'intermédiaire des modules embarqués _safe terminal / server_.

> Depuis un _profil_ _prf1_ de Bob pour _myApp1_ accédé en mode _avion_, le contenu du _safe_ de Bob pour _myApp1_ est lu depuis une micro base de données locale cryptée pour n'être lisible que par _myApp1_ et Bob. Celle-ci a été mise à jour lors de sa dernière utilisation connectée à Internet du profil _prf1_.

### Par un fichier `safe.json` crypté (intérêt à confirmer)
Au lieu d'obtenir le contenu du _safe_ de son serveur, le module _safe terminal_ peut demander à l'utilisateur de lui fournir un fichier `safe.json` (gzippé et crypté).

Le cryptage est assuré par une _phrase longue_ connue seulement de l'utilisateur.

Après lecture et décryptage, l'application fonctionne comme si le contenu avait été obtenu du _safe_ avec les différences suivantes:
- les mises à jour sont accumulées en mémoire mais pas transmises au serveur.
- c'est à l'utilisateur de demander la _sauvegarde_ (après compression et cryptage) dans un fichier externe et de gérer sa communication éventuelle par le cloud ou des clés USB ...

## Organisation d'un document _safe_

### Création d'un _safe_ par un utilisateur
Une clé AES `K` de 32 bytes est tirée aléatoirement: elle ne pourra pas changer et est la clé de cryptage du _safe_.

Un couple de clés `C / D` asymétriques est générée:
- `C` est en clair : son hash court est l'identifiant du _safe_
- `D_K` est le cryptage de `D` par `K`.
- `C_KH` est le hash du cryptage de `C` par `K`. Cette donnée permet au module _safe server_ de s'assurer en traitant une requête qu'elle a bien été issue d'un module _safe terminal_ détenant la clé `K`.

L'utilisateur donne:
- une _phrase_ `p0` qui de facto ne pourra plus être changée.
- une _phrase_ `p1`:
  - elle pourra être changée,
  - `p1_H` est le SHA du `SH(p0, p1)`. Index unique du _safe_.
- une _phrase_ `p2`:
  - elle pourra être changée,
  - `p2_H` est le SHA du `SH(p0, p2)`. Index unique du _safe_.

Les _phrases_ sont _longues_, au moins 16 signes pour `p0` et 24 pour `p1` et `p2`. Pour accéder à son _safe_ après création, son propriétaire doit fournir:
- sa phrase `p0`,
- l'une de ses deux phrases `p1` OU `p2`.

La clé `K` du safe est stockée dans les propriétés `K1` et `K2` cryptages respectifs par  `SH(p0, p1, SEP)` et `SH(p0, p2, SEP)`.

Après avoir identifié son _safe_ son propriétaire peut:
- changer `p1` et / ou `p2` à condition de fournir `p1` ou `p2` actuel.
- de facto il n'est pas possible de changer `p0`, bien que non stockée directement dans le _safe_ sous aucune forme.

#### Remarques:
- A aucun moment les phrases `p0 p1 p2` ne sont stockés quelque part ni transmis _en clair_. Elles ne figurent _en clair_ que très temporairement à la saisie par l'utilisateur dans le module _safe terminal_ et y sont effacées dès la fin de la saisie.
- L'existence de deux phrases `p1` et `p2` autorise l'oubli de l'une des deux. En revanche le propriétaire du _safe_ ne doit pas oublier `p0`: il peut donner une adresse _e-mail, son état civil, etc._ Ce texte N'EST JAMAIS exposé extérieurement et même si un hacker essayait la bonne phrase en tablant sur la banalité d'une adresse email, il n'aurait aucune chance de trouver `p1` ou `p2` avant la fin du monde (si un minimum de règles de choix est respecté).

### Structure du document _safe_
Il comporte les parties suivantes:
- **entête** : ce sont les propriétés décrites ci-avant:
  - `C D_K C_KH K1 K2 p1_H p2_H`
  - `maxLife` : durée de vie du _safe_, sachant que toute utilisation recule cette date (_purge_ automatique des _safe_ obsolètes / fantômes).
- **profils**: map avec une entrée par couple `myApp1 / idprf1`, ou `idprf1` est l'identifiant aléatoire d'un profil attribué à sa création. Ces items sont immuables après création.
- **applications**: map avec une entrée par `myApp1` contant les données spécifiques de Bob pour _myApp1_.

### Ouverture d'un _safe_ sur tout appareil
L'utilisateur saisit `p0` et `p1` (ou `p2`).

L'application _Safe_ accède au _safe_ par la clé d'index `p1_H` (ou `p2_H`).

L'application _Safe_ décode `K1` (ou `K2`) par `SH(p0, p1 (ou p2), SEP)`, et dispose ainsi de la clé `K` requise pour décrypter les items des sections `devices` et `applications`.

### Déclaration d'un profil pour l'application `myApp1` sur un device de confiance
Un utilisateur peut déclarer ou ou plusieurs _profils_ sur un _device de confiance_ pour une application `myApp1`.

Une base de données locale IDB de nom `safes` a une entrée par application ayant déclaré un profil sur ce device. 
- Cette entrée `myApp1` est une **liste d'objets _profile_** cryptée par la clé C de l'application `myApp1` qui devrait n'être lisible que par l'application myApp1. 
  - toutefois sa clé D correspondante étant disponible dans le source de myApp1,  avec un peu d'effort un hacker peut la retrouver, la sécurité est _faible_. 
- chaque objet _profile_ a les propriétés suivantes:
  - `idprf`: id aléatoire générée à la création du profil.
  - `about`: texte donné par l'utilisateur qualifiant son profil et le device sur lequel il se trouve, par exemple: `Bob sur PC Alice, accès à mon compte`.
  - `safeId`: identifiant du safe de l'utilisateur Bob.
  - `K1` : clé K du safe cryptée par SH(p0, p1, SEP) la phrase secrète 1 de Bob pour accéder à son safe d'identifiant safeId.
  - `K2` : clé K du safe cryptée par SH(p0, p2, SEP) la phrase secrète 2 de Bob pour accéder à son safe d'identifiant safeId.
  - `Kp` : clé K du safe cryptée par SH(pin + cx, cy, SEP) où,
    - `pin` est le code PIN choisi par Bob pour ce profil (il peut réemployer le même PIN pour plusieurs profils).
    - `cx`: challenge aléatoire généré à la création du profil.
    - `cy`: challenge aléatoire généré à la création du profil.
  - `cx`: challenge cx.
- **pour chaque _profil_ `idprf`** la base IDB dispose d'un objet `safeData` doublement crypté par la clé C de myApp1 et par la clé K du safe de Bob:
  - cette entrée est la copie de l'entrée correspondante du _safe de Bob pour myApp1_ effectuée lors du dernier accès à ce profil avec une connexion Internet active.

#### Process de déclaration d'un profil
Depuis l'application myApp1 une page permet de créer un (voire plusieurs) _profil_ d'accès pour l'utilisateur Bob:
- Bob donne son couple de phrase `p0, p1 ou p2` ce qui permet au module _safe_ de myApp1:
  - d'obtenir la clé `K` d'accès au safe de Bob et l'identifiant `safeId` de ce safe.
  - d'obtenir l'entrée dans ce _safe_ pour l'application myApp1, objet crypté par la clé K obtenue ci-dessus et la clé C de myApp1.
- Bob donne un libellé `about` explicitant à quoi sert le profil et sur quel appareil il se trouve.

> Ces libellés _about_ permettront ensuite à Bob de sélectionner depuis une liste en clair de ses profils, celui ou ceux qu'il voudra détruire.

L'application terminale _myApp1_ : 
- génère aléatoirement `idprf` et les challenges `cx cy` et enregistre dans la base IDF _safes_ l'objet décriavant le profil généré.
- génère un couple `Sa Va` de clés asymétriques signature / vérification.
- initialise `nbe`, le nombre d'échecs, à 0.
- disposant de la clé `K` de cryptage du _safe_ par la donnée par l'utilisateur,
  - calcule `Kp`, cryptage de la clé `K` par le `SH(PIN + cx, cy, SEP)`.
  - génère dans `sign` la signature par `Sa` du `SH(PIN, cx)`.

Le module _safe terminal_ inclus dans l'application terminale myApp construit un objet _profile server_ et le transmet au module _safe server_ (inclus dans _myApp server_) pour enregistrement dans l'entrée `safeId` du safe de Bob relatif à ce _profile_ d'identifiant `idprf`:
  - `about`: le libellé donné par l'utilisateur à propos du profil créé.
  - `Kp Va cy sign nbe`: les données calculées ci-avant.
  - cet objet est doublement crypté avant transmission au module _safe server_, par la clé K du safe de Bob et la clé C de l'application.

> Remarque: `Sa` a servi à générer la signature sign mais n'est plus utilisé ensuite et n'est pas mémorisé. `Va` l'est et servira à authentifier la signature d'un PIN saisi par Bob.

Après ce calcul,
- le _safe_ a été mis à jour par le module _safe server_ avec un nouvel profil contenant les données cryptographiques permettant à Bob de s'authentifier pour ce profil par un code PIN assez simple.
- sur le _device_ la base locale IDB _safe_ contient une entrée cryptée relatif au profil créé. 

## myApp1 : accès à son entrée dans le safe de Bob depuis un _device_ où il n'a pas de `profil`
Sur n'importe quel poste, Bob peut se trouver dans la situation,
- soit où aucun _profil_ n'est déclaré pour l'application _myApp1_,
- soit où il y a un ou plusieurs profils pour l'application _myApp1_ mais il n'en reconnaît aucun, n'en n'en pas les codes PIN (vraisemblablement parce qu'ils ont été déclarés par d'autres utilisateurs).

L'application terminale _myApp1_ ne peut pas accéder p  ar son module inclus _safe terminal_ à l'entrée du _safe_ de Bob depuis la base locale IDB _safe_ (qui peut-être d'ailleurs n'existe pas). 

**Bob saisit sa phrase _longue_** `p0 et (p1 ou p2)` et le module _safe server_ inclus dans l'application myApp1 server accédera à l'entrée pour myApp1 dans le safe de Bob:
- le module _safe server_ retourne,
  - K1 et K2 (qu'il ne peut pas décrypter),
  - l'objet pour myApp1 doublement crypté.
- le module _safe terminal_ connaissant p0 et p1 ou p2, peut décrypter K1 ou K2 et obtenir la clé K du safe de Bob. 
  - Il peut de ce fait décrypter l'entrée du safe de Bob pour _myApp1_ ayant la clé K et la clé D privée de myApp1.

## myApp1 : accès à son entrée dans le safe de Bob depuis un _device_ où il a reconnu / choisi son `profil`
Le module _safe terminal_ a trouvé la base de donnée locale IDB _safe_ et a présenté à Bob la liste des profils qu'elle contient:
- Bob choisit son profil.

### Cas 1: Internet est accessible
Bob peut authentifier son _safe_,
- soit en donnant sa phrase _longue_ `p0 et (p1 ou p2)`,
- **soit en donnant son code PIN beaucoup plus court**.

In fine à la fin de ce processus, 
- le module _safe terminal_ dispose de l'objet entrée de myApp1 dans le _safe_ de Bob obtenu du module _safe server_ inclus dans _myApp1 server_.
- cet objet est inscrit dans la base locale IDB pour l'entrée idPrf pour myApp1. Elle permettra un accès en mode _avion_ ultérieur depuis ce profil.
- myApp1 dispose en clair de cet objet décrypté par le module _safe terminal_ ce qui lui permet de lire les _droits d'accès_ et _préférences_ de Bob pour ce profil et myApp1.

#### Processus d'authentification du code PIN
Le module _safe terminal_ inclus dans myApp1 terminal dispose:
- de l'identifiant `idprf` du profil sélectionné par Bob,
- du code PIN sais par Bob,
- de l'objet `idprf` crypté par la clé C de _myApp1_ avec les propriétés suivantes:
  - `about`: texte donné par l'utilisateur qualifiant son profil et le device sur lequel il se trouve, par exemple: `Bob sur PC Alice, accès à mon compte`.
  - `safeId`: identifiant du safe de l'utilisateur Bob.
  - `K1` : clé K du safe cryptée par SH(p0, p1, SEP) la phrase secrète 1 de Bob pour accéder à son safe d'identifiant safeId.
  - `K2` : clé K du safe cryptée par SH(p0, p2, SEP) la phrase secrète 2 de Bob pour accéder à son safe d'identifiant safeId.
  - `Kp` : clé K du safe cryptée par SH(pin + cx, cy, SEP) où,
    - `pin` est le code PIN choisi par Bob pour ce profil (il peut réemployer le même PIN pour plusieurs profils).
    - `cx`: challenge aléatoire généré à la création du profil.
    - `cy`: challenge aléatoire généré à la création du profil.
  - `cx`: challenge cx. 

Il interroge le module _safe server_ inclus dans _myApp1 server_ en lui passant en paramètres:
  - `safeId` l'identifiant du _Safe_ de Bob.
  - `idprf` l'identifiant du _profil_ choisi par Bob.
  - `SH(PIN, cx)` où `cx` est le challenge trouvé ci-dessus et `PIN` le code PIN saisi par l'utilisateur.
- le module _safe server_:
  - accède au _Safe_ de Bob par l'identifiant `safeId`.
  - vérifie que `idprf` est bien la clé d'une entrée `e` de la map des profils de myApp1.
  - vérifie par `e.Va` que `e.sign` est bien la signature de `SH(PIN, cx)`.
  - en cas de succès, il met à 0 `e.nbe` s'il ne l'était pas déjà.
  - retourne `e.cy, e.Kp` et la section `myApp1` du _safe_ de Bob.
- le module _safe server_ peut ainsi,
  - calculer x = SH(pin + cx, cy, SEP) disposant désormais de `cy`.
  - vérifier que x décrypte bien Kp, ce qui à la fois lui prouve que le challenge cy est le bon, que le code PIN a bien été reconnu et lui donne la clé K du _safe_ de Bob.

### Échecs
- (1) si le `idprf` reçu du module _safe terminal_ n'est pas dans la map détenue dans le serveur pour l'application myApp1, c'est que cet appareil N'EST PAS / PLUS de confiance, que le _profil_ a été désactivé par Bob.
- (2) si la signature `sign` n'est pas vérifiée par `Va`, c'est que le code PIN n'est pas le bon. `Va` correspond au `Sa` qui a été utilisé à sa signature, `cx` était bien celui fixé à la déclaration. **Le nombre d'erreurs `nbe` est incrémenté**.
  - si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais, l'entrée `idprf` est supprimée. L'utilisateur devra refaire une déclaration de _profil_ avec un code PIN (ce qui exigera une authentification _forte_ par `p0` et (`p1` ou `p2`)).

En cas de réussite, le nombre d'échecs `nbe` est remis à 0 s'il ne l'était pas déjà.

#### Remarques sur la _sécurité_ du protocole
- le **code PIN** n'est jamais stocké ni passé en clair sur le réseau au module _safe server_: 
  - il ne peut pas être détourné ou être lu depuis la base de données.
  - il ne figure que temporairement en mémoire dans le module _safe terminal_ inclus dans l'application _myApp1 terminal_ durant la phase d'authentification du _safe_ de `Bob`.
- pour tenter depuis les données du _Safe server_ d'obtenir le code PIN par force brute, il faut effectuer une vérification de `sign` avec le _challenge_ `SH(PIN, cx)` mais `sign` est crypté par la clé privée de cryptage général du module _safe server_.

Pour que cette dernière attaque pour trouver le PIN de `Bob` par force brute ait des chances de succès, il faut que le hacker ait obtenu frauduleusement:
- (1) le contenu en clair de l'objet _safe_ en base et pour cela il lui faut conjointement,
  - avoir accès à la base en lecture ce qui requiert, soit une complicité auprès du fournisseur de la base de donnée, soit **la complicité de l'administrateur technique**.
  - avoir la clé de décryptage des contenus de celle-ci inscrite dans la configuration de déploiement des serveurs. Ceci suppose la **complicité de l'administrateur technique** effectuant ces déploiements.
- (2) le terme `cx` lisible en _debug_ (et un peu d'effort) dans la base de données IDB d'un _device_ **débloqué** (session utilisateur ouverte) de Bob.
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
