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
- `devAppToken` : C'est 'identification d'UNE exécution l'application terminale sur UN _device_, à un instant donné une seule exécution terminale de l'application peut s'en prévaloir.
- `time` : date-heure en milliseconde de la génération du jeton d'accès. Cette donnée fait partie du _challenge_ des signatures du jeton.
- une liste de preuves de possession des droits constituée chacune d'un triplet:
  - `type` : du droit,
  - `target` : cible à laquelle le droit s'applique,
  - `signatures` : liste de signatures `{var1: sig1, ...}` du couple `hash(devAppToken), time` par chaque variante `var1` de la clé `S` du droit correspondant.

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

Un autre solution est de faire gérer le stockage de ces _droits_ par une application _Safe_ en charge de résoudre les questions ci-dessus et de rendre accessible les _droits_ aux applications qu'ils concernent.

#### Les applications terminales ne doivent pas être _piratées_
Si l'application terminale est une application _pirate_ qui a été lancée en déjouant la vigilance de l'utilisateur (par exemple en le sollicitant d'appuyer sur un lien envoyé par un e-mail frauduleux), elle dispose des clés des droits auquel l'utilisateur lui a donné l'accès. 

Elle peut les envoyer sur un serveur pirate où elles seront à disposition de pirates pour utilisation dans la _vraie_ application et lui transmettre ces clés _usurpées_.

L'application _légitime_ a dans son code _sa clé de décryptage privée_ qui lui permet de décrypter les droits: mais en exécution en mode _debug_ un hacker un peu habile peut la retrouver. Cette _sécurité_ est plus symbolique qu'effective.

Il n'existe aucun procédé logiciel qui permette de connaître l'origine d'une application, de quelle _source_ elle vient, si elle est _légitime_ ou _pirate_. Depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle de l'application pirate,
- avoir confiance dans le fait que le CDN distribue bien un code non piraté,
- avoir obtenu d'un expert indépendant l'assurance que ce code est bien légitime et ne redistribue pas les clés d'accès,
- avoir vérifié son certificat.

L'utilisation de l'application _Safe_ pour lancer les applications _légitimes_ sans risque d'usage d'une URL _pirate_ réduit les risques: l'utilisateur n'a plus qu'é s'assurer qu'une seule application (_Safe_) est légitime au lieu de surveiller _toutes_ les URLs des applications.

### Demandes de droits par une application terminale
En supposant que l'application terminale ait obtenu, en sécurité, la **liste des droits** de l'utilisateur, lorsque celle-ci recherche la clé `S` d'un _droit_, plusieurs situations se présentent:
- (1) pour le `type` fixé (par exemple `DRTARIF` : _droit à effectuer une modification tarifaire_), il n'existe qu'une clé unique, `target` est vide. Un seul terme de la liste des droits peut s'appliquer.
- (2) le `type` ET la valeur de `target` sont fixées (par exemple `LOGIN, 1234` : _droit de l'utilisateur 1234 à se connecter_) par l'application. Une seul terme _au plus_ de la liste des droits peut s'appliquer.
- (3) le `type` est fixé MAIS PAS la valeur de `target` (par exemple `LOGIN`):  l'application recherche **à la fois une cible et sa clé**. Plusieurs droits de la liste peuvent être candidats et l'utilisateur devra **désigner** le login qu'il choisit: dans ce cas un commentaire `about` attaché à chaque droit lui est utile (par exemple en donnant un nom en clair plutôt qu'un code).

### Validation des _droits_ d'un jeton par le serveur de l'application
Quand le serveur de l'application traite une opération,
- il obtient de la requête le jeton d'accès et le décrypte par sa clé privée de décryptage.
- le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _droit_ de la liste du jeton,
  - il obtient depuis la base de données la map dés clés acceptées `{var1: V1, var2:v2 ...}` de vérification associée à sa cible,
  - il vérifie que la `signature` du couple `hash(devAppToken), time` est bien validée par la variante `var1` de `V`, ce qui prouve que l'application terminale en détient effectivement la clé de signature `S` correspondante.

> `hash(devAppToken), time` est utilisé comme _challenge_ cryptographique et n'est pas présenté plus d'une fois pour une application donnée.

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `target` d'un `type` de droit la dernière map dés clés acceptées `{var1: V1, var2:v2 ...}` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `{var1: V1, var2:v2 ...}`, et en cas de changement refait une vérification avant de valider / invalider le droit correspondant.
- le serveur conserve en cache pour chaque `hash(devAppToken)` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, leur _validation_ ne requiert qu'un calcul en mémoire sans accès à la base pour obtenir les clés de vérification.

### "Un" droit, "plusieurs" clés
Le serveur _peut_ mémoriser pour un droit non pas une clé `V` mais une liste de clés `{var1:V1, var2:v2 ...}`: pour être validé, un droit d'un jeton d'accès doit fournir une signature qui a été établie par la clé `Si` correspondant à la variante acceptée. 
- Si la variante acceptée ne figure pas dans le jeton d'accès, c'est que l'application terminale N'A PLUS ACCÈS au droit qui a été changé par l'application serveur et n'a pas jugé pertinent de retransmettre cette information à l'utilisateur pour qu'il la stocke dans ses droits.
- Si la variante figure mais que la vérification échoue, c'est une tentative de fraude.

La logique applicative peut ainsi prévoir de distribuer plusieurs _variantes_ de clés à des détenteurs différents selon ses propres critères (par exemple un _pseudo court_ du détenteur): elle peut aussi au cours du temps _désactiver_ certaines variantes dans le serveur et inhiber ainsi les détenteurs qui les possédaient.

> Ce dispositif permet aussi de gérer un _changement de clé_ pour un droit lorsqu'il a été attribué trop généreusement ou pour un temps limité. La création d'une nouvelle clé peut être faite et sa distribution restreinte aux seuls détenteurs souhaitables dans le futur, le cas échéant après un certain temps où les deux peuvent être admises, _l'ancienne_ peut être supprimée.

# L'application _Safe_

Cette application a pour objet de gérer le (ou les) _coffres forts_ des utilisateurs.

Après avoir lancé l'application _Safe terminale_ depuis son appareil, un utilisateur lui indique quel est son _coffre fort_ de manière à ce que les applications lancées ultérieurement sur cet appareil puissent y trouver diverses données _sensibles_ de l'utilisateur dont ses _droits d'accès_.

Une fois ainsi initialisée, l'application _Safe terminale_ peut:
- gérer ses appareils _de confiance_.
- gérer les données sensibles _droits, préférences ..._ de chaque application que l'utilisateur peut utiliser,
- lancer une de celles-ci.

Pour chaque application lancée, l'application _Safe terminale_ peut:
- stocker `ses droits` et les mettre à jour.
- stocker _divers objets_, dont des objets transmis par un autre utilisateur disposant lui aussi d'un _safe_. Certains de ces objets peuvent être interprétés par l'application comme des _préférences / options_.

> Le lancement d'applications par le _Safe_ évite le risque de lancement d'une application _piratée_ et permet de choisir le cas échéant une des des _options_ au lancement ouvrant la session dans un contexte déjà pré-fixé.

> L'URL de lancement de _Safe_ doit être soigneusement vérifiée: le code de cette application est lisible dans le browser et il est possible de vérifier auprès de sites certificateurs que l'application est _fair_ évitant ainsi à le faire pour chaque application gérée par _Safe_ (bien que ce soit toujours possible).

> L'application _Safe_ ne gère pas à proprement parler des _utilisateurs_ mais leurs _coffres forts_: rien n'empêche un _utilisateur_ (une personne) de posséder plus d'un coffre, rien ne relient les coffres entre eux ni à un quelconque signifiant dans le monde réel.

> Sur un appareil donné, à un instant donné **il n'y a qu'une seule application _Safe_ active** et elle ne sert **qu'un seul _safe_**, celui désigné par l'utilisateur lors du lancement de l'application.

## Appareils déclarés _de confiance_ d'un utilisateur
Un utilisateur qui veut utiliser une application depuis un _device_ est placé devant deux cas de figure:
- **soit il juge l'appareil _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche: il en a le _login_ ou a minima quelqu'un accepte de lui prêter l'appareil _session ouverte_.
  - il peut y laisser quelques informations cryptées et espérer raisonnablement les retrouver plus tard.
- **soit il n'a pas confiance dans cet appareil** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver des données.

Au lancement de l'application _Safe terminale_ il doit être en mesure d'identifier son _safe_.
- il peut toujours le faire en saisissant des phrases _longues_ `p0` et `p1` (ou `p2`).
- si c'est un appareil _de confiance_ ayant accès à Internet, il peut le faire en saisissant seulement un `code PIN` (d'au moins 8 signes) ce qui est plus rapide.

Pour un utilisateur lancer une application depuis un appareil _de confiance_ a plusieurs autres avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant une petite base de données locale (cryptée) pour chaque application comme _cache_ de ses documents: ceux qui y figurent et à jour n'auront pas besoin d'être demandés au serveur de l'application.
- **disponibilité des droits** de l'utilisateur pour chaque application le dispensant de s'en souvenir ou de les copier / coller d'un support externe.
- **possibilité d'accéder à l'application en mode _avion_** sans accès au réseau en utilisant les documents et les droits en _cache_.

> Même _de confiance_ un appareil _peut_ être utilisé par d'autres que soi-même, même dans un cadre familial ou de couple, l'appareil n'est pas strictement _personnel_.

## Stockage des _safes_

### Par le serveur _Safe_
Ce serveur et sa base de données sont le lieu de stockage _par défaut_ des _safes_.

L'accès par l'application terminale au _safe_ de l'utilisateur se fait par deux procédés:
- depuis tout appareil, la donnée des phrases _longues_ `p0` et `p1` (ou `p2`) connues uniquement du détenteur du _safe_.
- depuis un appareil _de confiance_ la donnée d'un code PIN (au moins 8 signes)

Après avoir identifié en sécurité le _safe_ de l'utilisateur, l'application _Safe terminale_ demande à son serveur son contenu et le charge en mémoire. Elle peut désormais communiquer les données qui les concernent aux applications lancées sur l'appareil.

La mise à jour du _safe_ se répercute directement sur le serveur.

> En mode _avion_, le contenu du _safe_ de chaque utilisateur ayant déclaré l'appareil _de confiance_ est lu depuis une micro base de données locale cryptée pour n'être lisible que par son détenteur. Celle-ci a été mise à jour lors de sa dernière utilisation connectée à Internet.

### Par un fichier `safe.json` crypté
Au lieu d'obtenir le contenu du _safe_ de son serveur, l'application _Safe terminale_ peut demander à l'utilisateur de lui fournir un fichier `safe.json` (gzippé et crypté).

Le cryptage est assuré par une _phrase longue_ connue seulement de l'utilisateur.

Après lecture et décryptage, l'application fonctionne comme si le contenu avait été obtenu de son serveur avec les différences suivantes:
- les mises à jour sont accumulées en mémoire mais pas transmises au serveur.
- c'est à l'utilisateur de demander la _sauvegarde_ (après compression et cryptage) dans un fichier externe et de gérer sa communication éventuelle par le cloud ou des clés USB ...

## Organisation d'un document _safe_

### Création d'un _safe_ par un utilisateur
Une clé AES `K` de 32 bytes est tirée aléatoirement: elle ne pourra pas changer et est la clé de cryptage du _safe_.

Un couple de clés `C / D` asymétriques est générée:
- `C` est en clair : c'est l'identifiant du _safe_
- `D_K` est le cryptage de `D` par `K`.
- `C_KH` est le SHA du cryptage de `C` par `K`. Cette donnée permet au _Safe serveur_ de s'assurer en traitant une requête qu'elle a bien été issue d'une application _Safe terminale_ détenant la clé `K`.

L'utilisateur donne:
- une _phrase_ `p0` qui de facto ne pourra plus être changée.
- une _phrase_ `p1`:
  - elle pourra être changée,
  - `p1_H` est le SHA du `SH(p0, p1)`. Index unique du _safe_.
- une _phrase_ `p2`:
  - elle pourra être changée,
  - `p2_H` est le SHA du `SH(p0, p2)`. Index unique du _safe_.
- `pseudo_K` un pseudo _court_ comme `Jules César` stocké crypté par la clé `K` et qui sera immuable. Il sert principalement à l'affichage sur un appareil qui peut être partagé par plusieurs utilisateurs leur permettant de lever un doute sur l'utilisateur ayant ouvert l'application.

> Question: enregistrer un `SH(pseudo)` afin d'interdire les doublons ?

Les _phrases_ sont _longues_, au moins 16 signes pour `p0` et 24 pour `p1` et `p2`. Pour accéder à son _safe_ après création, son propriétaire doit fournir:
- sa phrase `p0`,
- l'une de ses deux phrases `p1` OU `p2`.

La clé `K` du safe est stockée dans les propriétés `K1` et `K2` cryptages respectifs par  `SH(p0, p1, SEP)` et `SH(p0, p2, SEP)`.

Après avoir identifié son _safe_ son propriétaire peut:
- changer `p1` ou `p2` à condition de fournir `p1` ou `p2` actuel.
- de facto il n'est pas possible de changer `p0`, bien que non stockée directement dans le _safe_ sous aucune forme.

#### Remarques:
- A aucun moment les phrases `p0 p1 p2` ne sont stockés nulle part ni transmis _en clair_. Elles ne figurent _en clair_ que très temporairement à la saisie par l'utilisateur dans l'application _Safe terminale_ et y sont effacées dès la fin de la saisie.
- L'existence de deux phrases `p1` et `p2` autorise l'oubli de l'une des deux. En revanche le propriétaire du _safe_ ne doit pas oublier `p0`: il peut donner une adresse _e-mail, son état civil, etc._ Ce texte N'EST JAMAIS exposé extérieurement et même si un hacker essayait la bonne phrase en tablant sur la banalité d'une adresse email, il n'aurait aucune chance de trouver `p1` ou `p2` avant la fin du monde (si un minimum de règles de choix est respecté).

### Structure du document _safe_
Il comporte les parties suivantes:
- **entête** : ce sont les propriétés décrites ci-avant:
  - `C D_K C_KH K1 K2 p1_H p2_H pseudo_K`
  - `lastAccess` : numéro du dernier mois d'utilisation du _safe_ afin de pouvoir procéder périodiquement à une _purge_ des _safe_ obsolètes / fantômes.
- **devices** de confiance: map avec une entrée par `devId`, identifiant aléatoire d'un device lors de sa déclaration _de confiance_. Voir ci-après **Déclaration d'un device de confiance**.
- **applications**: map avec une entrée par `appId`, identifiant d'une application pour laquelle des données spécifiques sont accumulées.

### Ouverture d'un _safe_ sur tout appareil
L'utilisateur saisit `p0` et `p1` (ou `p2`).

L'application _Safe_ accède au _safe_ par la clé d'index `p1_H` (ou `p2_H`).

L'application _Safe_ décode `K1` (ou `K2`) par `SH(p0, p1 (ou p2), SEP)`, et dispose ainsi de la clé `K` requise pour décrypter les items des sections `devices` et `applications`.

### Déclaration d'un _device de confiance_
Quand l'application _Safe terminale_ déclare, sur demande de l'utilisateur, son device _de confiance_:
- un identifiant aléatoire `devId` est généré.
- un libellé court `devName` est saisi par l'utilisateur, par exemple `PC d'Alice` qui lui permettra ultérieurement de retirer sa confiance à ce device.
- un code PIN est saisi par l'utilisateur: il pourra utiliser, sur ce device seulement, pour identifier son _safe_.
- un challenge `cx` aléatoire est généré. 

#### Sur le device
Une entrée du _localStorage_ `safes` est créé, si elle ne l'était pas déjà, contenant la liste des identifiants comme `['Bob@Ktux...', 'Alice@Qxc6...'` des _safes_ dont le device est _de confiance_
- `Bob` est le pseudo court du _safe_,
- `Ktux...` est son'identifiant (sa clé `C`).

Une micro base de données _IndexedDB_ `Ktux...@safe.idb` existe pour chaque _safe_ listé dans le localStorage `safes`. 

Cette base de données a:
- un row singleton `header` crypté par la clé de l'application _Safe terminale_.
_ une collection de rows `application` ayant deux propriétés:
  - `id` : code l'application.
  - `data` : données du row cryptées,
  _ - par la clé publique C de cryptage de l'application,
    - puis par la clé K du _safe_.

#### Row `header`
Il porte les propriétés suivantes:
- `devId` : identifiant du device venant d'être généré.
- `K1 K2 Kp` : cryptages de la clé `K` du _safe_ respectivement par `SH(p0, p1, SEP)`, `SH(p0, p2, SEP)` et `SH(PIN + cx, cz, SEP)`.
- `cx` : le challenge généré sur le device à la déclaration de confiance.

> `header` est crypté par la clé publique `C` de l'application _Safe terminale_ afin de ne pas apparaître en clair en _debug_. Cette sécurité est un peu illusoire, la clé privée D de décryptage figurant dans le source d'application _Safe terminale_ (et donc accessible avec un peu d'effort en _debug_).

#### Dans le document _safe_ géré par _Safe serveur_
Dans la map **devices** de confiance de ce document un item est ajouté sous la clé `devId` avec les propriétés suivantes:
- `devName`: le libellé du device donné par l'utilisateur à la déclaration de confiance.
- `Kp Va cy sign nbe`: ces données ont été initialisées à la déclaration de confiance par l'application _Safe terminale_ et envoyées à _Safe serveur_ pour ajout dans la map.

A la déclaration de confiance par l'application _Safe terminale_:
- génère un couple `Sa Va` de clés asymétriques signature / vérification est généré.
- génère Un challenge `cy` est tiré aléatoirement.
- initialise `nbe`, le nombre d'échecs, à 0.
- disposant de la clé `K` de cryptage du _safe_ par la donnée par l'utilisateur de `p0` et `p1` (ou `p2`) 
  - calcule `Kp`, cryptage de la clé K par le `SH(PIN + cx, cy, SEP)`.
  - génère dans `sign` la signature par `Sa` du `SH(PIN, cx)`.

Après ce calcul,
- le _safe_ est mis à jour par _Safe serveur_ avec un nouvel item **devices de confiance**,
- le `header` de la micro base de données locale `Ktux...@safe.idb` est enregistré. 

### Obtention d'un _safe_ sur un device de confiance de `Bob` par code PIN
A l'ouverture de _Safe terminale_, la liste des utilisateurs ayant déclaré ce device _de confiance_ est obtenue en scannant l'entrée du _localStorage_ `safes`. On y trouve par exemple `Bob@Ktux...` et `Alice@QxYg...`:
- Bob clique sur l'entrée `Bob`, ce qui donne à l'application `Ktux...` l'identifiant de son _Safe_.
- l'application _Safe terminale_ ouvre `Ktux...@safe.idb` lit et décrypte le row `header`.
- l'utilisateur saisit un PIN et l'application _Safe terminale_ interroge son serveur en lui passant en paramètres:
  - `Ktux...` l'identifiant de son _Safe_ (sa clé `C`).
  - `devId` trouvé dans `header`.
  - `SH(PIN, cx)` où `cx` est le challenge trouvé dans `header` et `PIN` le code PIN saisi par l'utilisateur.
- l'application _Safe_ serveur:
  - accède au _Safe_ pour l'identifiant `Ktux...`.
  - vérifie que `devId` est bien la clé d'une entrée `e` de la map des devices de confiance du _safe_.
  - vérifie par `e.Va` que `e.sign` est bien la signature de `SH(PIN, cx)`.
  - en cas de succès, il met à 0 `e.nbe` s'il ne l'était pas déjà.
  - retourne `e.cy, e.Kp` et la section `applications` du _safe_.

Ayant le code `PIN` par saisie, `cx` dans le `header` et `cy` retourné par le serveur, l'application _Safe terminale_ peut désormais décrypter `Kp` du `header` et en obtenir la clé `K` du _safe_ de `Bob`:
- **elle décrypte tous les items de la section `applications` du _safe_ qui sont cryptés par K**.

#### Échecs
- (1) si le `devId` reçu de _Safe terminale_ n'est pas dans la map détenue dans le serveur, c'est que cet appareil N'EST PAS / PLUS de confiance.
- (2) si la signature `sign` n'est pas vérifiée par `Va`, c'est que le code PIN n'est pas le bon. `Va` correspond au `Sa` qui a été utilisé à sa signature, `cx` était bien celui fixé à la déclaration. **Le nombre d'erreurs `nbe` est incrémenté**.
  - si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais, l'entrée `devId` de la map _devices de confiance_ est supprimée. L'utilisateur devra refaire une déclaration de _confiance_ avec un code PIN (ce qui exigera une authentification _forte_ par `p0` et (`p1` ou `p2`)).

En cas de réussite, le nombre d'échecs `nbe` est remis à 0 s'il ne l'était pas déjà.

#### Remarques sur la _sécurité_ du protocole
- le **code PIN** n'est jamais stocké ni passé en clair sur le réseau au _Safe serveur_: 
  - il ne peut pas être détourné ou être lu depuis la base de données.
  - il ne figure que temporairement en mémoire de l'application _Safe terminale_ durant la phase d'authentification du _safe_ de `Bob`.
- pour tenter depuis les données du _Safe serveur_ d'obtenir le code PIN par force brute, il faut effectuer une vérification de `sign` avec le _challenge_ `SH(PIN, cx)` mais `sign` est crypté par la clé privée de cryptage général de l'application _Safe serveur_.

Pour que cette dernière attaque pour trouver le PIN de `Bob` par force brute ait des chances de succès, il faut que le hacker ait obtenu frauduleusement:
- (1) le contenu en clair de l'objet _safe_ en base et pour cela il lui faut conjointement,
  - avoir accès à la base en lecture ce qui requiert, soit une complicité auprès du fournisseur de la base de donnée, soit **la complicité de l'administrateur technique**.
  - avoir la clé de décryptage des contenus de celle-ci inscrite dans la configuration de déploiement des serveurs. Ceci suppose la **complicité de l'administrateur technique** effectuant ces déploiements.
- (2) le terme `cx` lisible en _debug_ (et un peu d'effort) dans le _localStorage_ d'un appareil de confiance **débloqué** (session utilisateur ouverte) de Bob.
  - sur un mobile avoir le mobile _déverrouillé_,
  - sur un PC avoir une session ouverte. 

> Ce double _piratage / complicité_ donne accès à la clé `K` du _safe_ de `Bob`, donc au contenu du _safe_, dont les clés d'accès. Toutefois les phrases `p0 p1 p2` restent inviolées et non modifiables par le hacker, puisque ne résidant que dans la mémoire de Bob.

> Cracker le code PIN de Bob sur un device ne compromet en rien, NI les codes PIN de Bob sur d'autres devices, NI le code PIN d'Alice sur ce device.

> Pour cracker **tous** les codes PIN, il faudrait pouvoir accéder à tous les appareils de confiance **déverrouillés / sessions ouvertes** et casser par force brute le PIN de _chaque safe pour chaque device_. 

#### Durcir (un peu) le code PIN
Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il est quasi incassable dans des délais humains: pour être mnémotechnique il va certes s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. Mais de nombreux styles de saisie mènent au code PIN depuis la phrase `allons enfants de la patrie`: avec ou sans séparateurs, des chiffres au milieu, des alternances de mots en minuscules / majuscules, un mot sur deux, etc. La seule _bonne intuition_ d'un texte est loin de donner le code PIN correspondant.

> Un _login_ des appareils un peu conséquent et un code PIN _un peu durci_ constituent en pratique une barrière **très coûteuse** à casser. Tant qu'à être un _délinquant_ une forte pression directe sur Bob devrait permettre de lui extorquer ses phrases / PIN à moindre coût.

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
