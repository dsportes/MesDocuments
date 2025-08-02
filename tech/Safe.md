---
layout: page
title: Module "Safe", gestion des "credentials"
---

# Gestion des _credentials_ dans les applications

Une application `app1` met en jeu deux parties:
- `app1Srv` : un pool de serveurs au service de l'application gérant ses données centrales persistantes, ses opérations de mise à jour et ses extractions / synchronisations de données.
- `app1App` : une application terminale, typiquement une application Web dont le source est lisible et délivré par un serveur statique / CDN.

Les opérations exécutées par le serveur comme les données qu'il peut retourner à l'application qui l'a sollicité, sont soumise, sauf exception, à des **droits d'accès / _credentials_**: d'une manière ou d'une autre l'utilisateur derrière l'application terminale doit exhiber qu'il possède effectivement les _droits requis_ par une opération pour la solliciter afin de mettre à jour et / ou obtenir des données centrales. Par exemple:
- `cpt` : droit à lire les documents d'un compte,
- `mbr` : droit de gestion des accès des membres d'un groupe,
- `trf` : droit de modification tarifaire ...

### _Credential_ / droit d'accès
Un _credential_ est matérialisé par les données suivantes:
- `type`, un code correspondant à sa _classe / catégorie_ `cpt, mbr, trf ...`
- `target` : l'identifiant (géré par l'application) de sa _cible_ qui peut comporter aussi bien des données lisibles (une adresse e-mail, un numéro de mobile ...) qu'être le résultat d'une génération aléatoire.
- une liste de clés, en général d'un seul terme:
  - `variant` : un code (vide par défaut) permettant de caractériser plusieurs clés pour un même droit identifié par `type / target`.
  - `vc` : clé publique de **vérification** du credential.
  - `sc` : clé privée de **signature** du credential.

Le serveur d'une application gère les _credentials_ mais n'en connaît **QUE** `type target vc` et n'a pas (sauf exception décrite ci-dessous) accès à le clé `sc` correspondante .

### Jetons d'accès
Quand l'application terminale soumet une opération au serveur, elle fournit dans sa requête un **jeton d'accès** qui réunit les preuves que son utilisateur dispose des _credentials_ requis pour exécuter cette opération. Un jeton comporte:
- `devAppId` : l'identification de l'application terminale s'exécutant sur un _device_. Cet id ne désigne qu'un seul couple _device / application_, à un instant donné une seule exécution peut s'en prévaloir.
- `time` : date-heure en milliseconde de la génération du jeton d'accès.
- une liste de preuves de possession des droits d'accès constituée chacune d'un triplet:
  - `type` : du credential,
  - `target` : cible à laquelle le credential s'applique,
  - `signature` : signature du couple `devAppId, time` par la clé `sc` du credential correspondant.

**Remarques**:
- un _hacker_ un peu entraîné peut obtenir l'identifiant `devAppId` en lançant en _debug_ l'application sur ce _device_.
- dans la liste des preuves, plusieurs pourraient concerner le même couple `type target`: il suffit que l'une d'elle soit reconnue comme valide pour que le credential le soit. Ceci permet de traiter les situations, temporaires ou non, où un même credential peut avoir plusieurs _variantes_ de signatures (une _ancienne_ et une _nouvelle_ ...).
- un _jeton d'accès_ est crypté par la clé publique du serveur applicatif ciblé de sorte seul celui-ci puisse le lire.

#### Solution simplifiée
Quand l'application terminale doit soumettre une opération requérant un _credential_ donné (par exemple `cpt` pour un compte `1234`), elle peut en obtenir la clé `ks` par un module la demandant à l'utilisateur. Ce dernier effectue une forme ou une autre de _copier / coller_ depuis par exemple un fichier de texte externe détenu sur son _device_ et lui affichant sur une ligne la clé `ks` pour le compte `1234`.

#### Solution par accès à un _safe_
Les _credentials_ de l'utilisateur sont stockés dans une base de données dédiées en charge de conserver des _coffres forts / safes_. Chaque _safe_:
- est accessible par un utilisateur en ayant l'accès exclusif.
- dans ce _safe_, chaque application a sa collection _d'items identifiés_. Certains de ces items contiennent des _credentials_ cryptés pour n'être lisible que par l'application opérant sous contrôle de l'utilisateur propriétaire du _safe_.

Quand l'application terminale doit soumettre une opération requérant un _credential_ donné (par exemple `cpt` pour un compte `1234`), elle peut en obtenir la clé `ks` en demandant par son module safe l'accès à l'item de l'application pour le _safe_ courant désigné par l'utilisateur.

#### L'application terminale ne doit pas être _pirate_
Si l'application terminale est une application _pirate_ qui a été lancée en déjouant la vigilance de l'utilisateur (par exemple en le sollicitant d'appuyer sur un lien envoyé par un e-mail frauduleux), elle dispose des clés que l'utilisateur lui a fourni. 

Elle peut les envoyer sur un serveur pirate où elles seront à disposition de pirates pour utilisation dans la _vraie_ application et lui transmettre ces clés _usurpées_.

L'application _légitime_ a dans son code sa clé privée qui lui permet de décrypter les clés ks: mais en exécution en mode _debug_ un hacker un peu habile peut la retrouver. Cette _sécurité_ n'est qu'apparente.

> Il n'existe aucun procédé technique qui permette de connaître l'origine d'une application, de quelle _source_ elle vient. Depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle d'un pirate,
- avoir confiance dans le fait que le site distributeur distribue bien un code non piraté,
- avoir vérifié son certificat.

### Validation des _credentials_ d'un jeton par le serveur de l'application
Quand le serveur de l'application traite une opération,
- il obtient de la requête le jeton d'accès et le décrypte par sa clé privée.
- le jeton présenté n'est acceptable que si son `time` _n'est pas trop vieux_ (quelques dizaines de secondes). Pour chaque _credential_ de la liste du jeton,
  - il obtient depuis la base de données la clé `vc` de vérification de la clé associée à sa cible,
  - il vérifie que la `signature` du couple `devAppId, time` est bien validée par `vc`.

> La _vérification cryptographique_ répond vrai si le texte `t` signé par `sc` est bien égal à la signature transmise.
  
> `devAppId, time` est utilisé comme _challenge_ cryptographique et n'est pas présenté plus d'une fois.

**Remarques de performances:**
- le serveur peut conserver en _cache_ pour chaque `target` d'un type de credential le dernier `sc` lu de la base. En cas d'échec de la vérification il relit la base de données pour s'assurer d'avoir bien la dernière version de `sc`, et cas de changement refait une vérification avant de valider / invalider le credential correspondant.
- le serveur conserve en cache pour chaque `devAppId` le dernier `time` présenté en vérification: l'application terminale doit présenter des _time_ toujours croissants afin d'éviter un éventuel vol de _vieilles_ signatures qui seraient présentées à nouveau par un hacker.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes credentials, leur _validation_ ne requiert qu'un calcul en mémoire sans accès à la base de vérification de signature.

> Ce procédé signature / vérification assure que les serveurs n'ont PAS besoin de détenir la clé `sc` d'un credential pour en vérifier la validité. `ks` peut rester cantonnée dans la mémoire d'une application terminale s'exécutant pour un utilisateur l'ayant fourni.

## "Un" credential, "plusieurs" clés
Le serveur _peut_ mémoriser pour un credential non pas une clé `vc` mais une liste de clés `[vc1, vc2 ...]`: pour être validé, le credential d'un jeton d'accès doit fournir une signature qui a été établie par **UNE DES** clés `sc` correspondantes.

La logique applicative peut ainsi prévoir de distribuer plusieurs _variantes_ de clés à des détenteurs différents selon ses propres critères: elle peut aussi au cours du temps en _désactiver_ certaines variantes dans le serveur et inhiber ainsi les détenteurs qui les possédaient.

Dans un jeton généré par l'application, s'il existait _plusieurs_ clés possibles, au lieu d'une signature pour un credential donné, c'est une liste de signatures (une par clé potentielle) qui est générée. Il suffit que l'une d'entre elles _matche_ avec la ou une des clés `vci` de la liste détenue par le serveur pour que le credential soit validé.

> Ce dispositif permet aussi de gérer un _changement de clé_ pour un credential lorsqu'il a été attribué trop généreusement. La création d'une nouvelle clé peut être faite et sa distribution restreinte aux seuls détenteurs souhaitables dans le futur: après un certain temps où les deux sont admises, _l'ancienne_ peut être supprimée.

# Gestion par un utilisateur des couples id / ks des droits qu'il a acquis
Quelqu'en soit le processus, à un moment donné un utilisateur a acquis un **credential** d'un type donné ce qui s'est manifesté par le fait qu'il connaisse le couple `target / sc` correspondant.

L'identifiant `target` peut être abscons, difficile à mémoriser.

Le ou les `sc` sont _impossibles_ à mémoriser, chacun étant constitué d'environ 400 lettres et chiffres d'apparence aléatoire.

## Fichier CSV des _credentials_ d'un utilisateur
L'utilisateur doit a minima recourir à un fichier dans lequel il a enregistré ses droits. 

Cette liste étant en soi incompréhensible, pour chaque terme `target / sc` l'utilisateur faut associer un _à propos_ parlant pour lui afin qu'il sache quelle ligne sélectionner quand l'application lui demandera un credential. 

Le fichier peut être un simple CSV ayant quatre colonnes: `application, type, about, target, sc`.

> Remarque: `sc` peut être en fait une liste de `sc` séparés par un espace.

### Demande de credentials d'une application
Lorsque l'application souhaite avoir la clé d'un _credential_, plusieurs situations se présentent:
- (1) pour le type fixé (par exemple `DRTARIF` : droit à effectuer une modification tarifaire), il n'existe qu'une clé unique, `target` est vide. Une ligne du fichier au plus est sélectionnable.
- (2) le type est fixé ET la valeur de `target` (par exemple `LOGIN / 1234` : droit de l'utilisateur 1234 à se connecter) est fixé par l'application. Une ligne du fichier au plus est sélectionnable.
- (3) le type est fixé MAIS PAS la valeur de `target` (par exemple `LOGIN`) est fixé par l'application qui recherche **à la fois une cible et sa clé**. Plusieurs lignes du fichier peuvent être sélectionnables et l'utilisateur devra **désigner** le login qu'il choisit: dans ce cas le commentaire `about` lui est utile (par exemple en donnant un nom en clair plutôt qu'un code).

L'application peut charger en mémoire le fichier CSV des credentials de l'utilisateur en lui demandant son _path_ sur le file-system local du _device_:
- dans les cas (1 2) l'application peut obtenir directement le `sc`,
- dans le cas (3) l'application peut présenter à l'utilisateur la liste des couples `about target` contenus dans le fichier. La sélection de l'utilisateur retourne le couple `target, sc` correspondant à son choix.

> Remarque: le fichier CSV est une des solutions simplistes possibles. Une autre solution encore plus simpliste est que l'application présente une double zone de texte à saisir dans laquelle l'utilisateur _colle_ le couple `target, sc` de son choix après l'avoir _copié_ quelque part (un mail, un gestionnaire de clés ...).

# Les modules _safe_
La gestion d'un fichier CSV de leurs credentials par les utilisateurs met en lumière le problème de leur gestion:
- le fichier CSV doit pouvoir être accessible depuis plusieurs _devices_, stocké sur le _cloud_ ...
- il doit être encrypté afin de ne pas exposer ses clés à des pirates et l'utilisateur doit se rappeler de sa clé de cryptage,
- la mise à jour du fichier suppose d'être vigilant: l'inscription d'une nouvelle clé (voire le _changement_ d'une clé) est un risque de dégradation / perte du fichier et en conséquence de tous ses droits précieusement récupérés au fil du temps.

Pour résoudre ce problème, un couple de modules **safe**, un pour l'application terminale, un pour le serveur, propose à un utilisateur qui le souhaite une solution pour gérer avec une confidentialité rigoureuse un **coffre fort** où il pourra stocker ses credentials, voire d'autres types d'items.

> _safe_ ne gère pas à proprement parler des _utilisateurs_ mais leurs coffres forts: rien n'empêche un _utilisateur_ (une personne) de posséder plus d'un coffre, mais absolument rien ne relient les coffres entre eux ni à un quelconque signifiant dans le monde réel.

**TODO**

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

