---
layout: page
title: Design des bases de données
---

# Document : row / data / classe
## Hash de strings
sha32(s) : encodage en base 64 URL du SHA 256 du string s.

shaS(s) : le SHA 256 (32 bytes) est tronqué des bytes 3 à 19, puis encodé en base 64 URL.

## Documents _SYNC_ et _NOSYNC_
Les classes de documents **synchronisables** supportent des abonnements directs à leurs changements.
- la **suppression** d'un document _SYNC_ s'effectue en le transformant en _zombi_: c'est une suppression logique dont la trace en base subsiste un certain temps afin que les bases synchronisées externes aient la possibilité d'être informées de leur _suppression_ (la propriété `deleted` du document est true).

Les classes de documents **NON synchronisables** ne supportent pas d'abonnements.
- la **suppression** d'un document _NOSYNC_ est traitée une purge physique effective (_delete_ classique).

### _Existence_ d'un document
Un document **synchronisable** peut être _zombi_ (propriété `deleted`):
- fonctionnellement il doit être considéré comme INEXISTANT.
- il peut être transmis à une session comme résultat d'une opération de _synchronisation_ afin que la session le supprime de son côté.
- en état _zombi_ le document n'a que quelques propriétés: 
  - `deleted` : true
  - `v` : sa version, la date-heure de l'opération l'ayant zombifié.
  - `_pk` : le hash court de la concaténation des propriétés de sa clé primaire.
- un document _zombi_ a une durée de vie limitée en tant que zombi: au bout d'un certain temps il est effectivement détruit, les sessions distantes ne pourront plus être informées par synchronisation incrémentale de sa disparition. 

Un document **NON synchronisable** supprimé n'est a pas d'existence.

Un document **peut** avoir une propriété `maxLife`:
- c'est sa date-heure maximale de vie _logique_: au delà de cette date-heure le document est **fonctionnellement périmé**.
- sa lecture dans une opération **AVANT** cette limite retourne un document `d` normal ayant simplement une valeur dans le futur de `maxLife`.
- sa lecture dans une opération **APRÈS** cette limite retourne,
  - soit un document `d` dont `deleted` est `true`.
  - soit AUCUN document (`d` est `null`) au delà d'un certain temps, sa durée de vie en tant que _zombi_ étant dépassée.

`maxLife` est une "epoch" en MINUTES: le nombre de minutes écoulées depuis le 1/1/1970. Depuis un time en ms, il suffit d'une division par 60000.

## Version des opérations / documents en base de données
Le _now_ d'une opération est sa date-heure UTC "epoch" prise au début d'une opération, et reprise au début de sa phase 2 (dans la _transaction_) : tout document mis à jour dans une opération a pour version `v` la propriété `now` de l'opération.

Pour un document donné la version ne peut pas régresser: quand un document est lu (dans une _transaction_) si sa version est supérieure ou égale au `now` de l'opération, la transaction est sortie en exception `REGVER` (_régression de version_) et la transaction est relancée avec une nouvelle valeur de `now` de l'opération.

## Classes spéciales `Status Org Task Subs SubsItem Ftp`
`Status` est une classe _technique_ qui gère le status du serveur fixé par l'administrateur technique avec un texte d'information éventuel explicatif. Elle supporte deux méthodes d'accès:
- `getSrvStatus()` retourne un objet avec les propriétés suivantes:
  - `now`: date-heure de l'opération de lecture du status du serveur.
  - `st`: status: 
    - 0 - jamais initialisé.
    - 1 : UP
    - 2 : DOWN
  - `txt`: texte d'information éventuel de l'administrateur.
  - `at`: date-heure de la dernière opération d'administration ayant mis à jour le status.
- `setSrvStatus(st, txt)` : opération de l'administrateur fixant le status du serveur.

`Org` est la classe de documents dont chaque instance représente une _organisation_, son statut, etc.
- sa _clé primaire_ `pk` (par rapport à son organisation) est par convention `1`.
- son contenu / comportement est spécifique de chaque application.

`Task` est une classe NON synchronisable représentant une opération _différée_:
- quand la tâche est une tâche d'administration globale du serveur, son organisation est par convention `ROOT`. Sinon son organisation est celle de la cible de la tâche.
- sa clé primaire est constituée du couple du code de l'opération et d'un identifiant fonctionnel.
- son contenu / comportement est générique.
- les opérations peuvent inscrire des tâches à exécuter.
- des opérations de niveau _administration_ peuvent lister les tâches, voire les supprimer.
- une opération externe CRON lance l'exécution des tâches quotidiennes (par exemple).

`Subs` (souscription) et `SubsItem` (élément de souscription) sont deux classes NON synchronisable représentant une souscription:
- une opération est associée à une seule organisation et ne peut déclencher de notifications que vis à vis des souscriptions relatives à son organisation. 
- sa _clé primaire_ est l'identifiant d'une application terminale sur un appareil (`sessionId`).
- son contenu / comportement est générique.
- des opérations génériques permettent d'enregistrer une souscription pour une application terminale par organisation.
- les souscriptions ont une `maxLife` permettant leurs autodestructions automatiques.
  - une souscription pour une session _active_ a une `maxLife` courte: elle doit normalement être supprimée / remplacée à la fin d'activité de la session.
  - une souscription pour une session _fermée_ à une maxLife longue de quelques jours: elle vise à faire apparaître des _notifications textuelles_ alors que l'application terminale n'est plus en exécution.

`Ftp` _File to purge_ est une classe technique, PAS un document_ gérant le commit à 2 étapes des créations / suppressions de fichiers attachés à un document.

## Format _data_
_data_ est un objet ayant A MINIMA les propriétés suivantes:
- `v` : date-heure de l'opération l'ayant créé / mis à jour / zombifié.
- `release` : numéro de _release_ de la structure d'un document permettant à la lecture de convertir les documents ayant une ancienne structure dans celle la plus récente.
- `p1 p2 p3 i1 i2 ...` : propriétés string constituant de la _clé primaire_ du document.

> En session, les propriétés suivantes sont reconstituées:
- `_clazz`: la classe du document.
- `_pk`: le hash court de la concaténartion des propriétés de sa clé primaire.

Un document _zombi_ (supprimé logiquement) a un _data_ (reconstitué) ne contenant **que** `_clazz _pk deleted`.

_data_ contient de plus,
- `maxLife`: facultative, "epoch" en MINUTES de fin de vie logique du document.
- les propriétés citées dans les index / sous-collection de sa classe.
- les autres propriétés _applicatives_ du document: 
  - elles ont n'importe quel nom commençant par une minuscule.
  - elles peuvent des `string, number, boolean, array, objet`.

**Quelques propriétés _techniques_** sont lisibles dans les _data_ stockés en cache d'une opération: 
- `_clazz`: la classe du document.
- `_status`:
  - `NONE`: le document a été lu mais pas (encore) mis à jour par l'opération.
  - `UPD`: le document a été lu et a été mis à jour par l'opération: cet état est à positionner à chaque mise à jour du document.
  - `NEW`: le document n'existait pas et a été créé ex-nihilo par l'opération.
  - `DEL`: le document a été lu ou créé par l'opération puis supprimé par elle.
- `_before`: objet technique contenant les valeurs des propriétés de sous-collection à la lecture du document depuis la DB.
- `deleted`: document _zombi_.

Un objet _data_ peut être _sérialisé / désérialisé_ par `encode / decode` de `@msgpack/msgpack`: la propriété `data` d'un _row_ (objet connu de la base de données) est la sérialisation cryptée par la clé du site.

## Format _Document_
Une classe de document hérite de la class générique `Document`.

Une instance de document est créée depuis les propriétés d'un _data_. Quelques méthodes:
- `static mutate()` : réalise la mutation d'un document de release antérieure dans la dernière release.
- `compile()` : facultatif - permet de calculer certaines propriétés _helpers_ après initialisation.
- `serialForApp()` : facultatif - sérialise le document pour transmission / synchronisation à l'application terminale, en sélectionnant les propriétés à transmettre en fonction des autorisations de l'application. Par défaut créé un objet en ne reprenant que les propriétés dont le nom ne commence par `_`.

## Format _row_
C'est un objet représentant le document stocké en DB.

En base de données, les propriétés **visibles de la base de données** sont:
- `pk` : clé primaire ou path.
  - pour les documents de classe `Org`, `pk` est le code de l'organisation.
  - pour les autres classes, `pk` est le shaS de `p1/p2/...` ou les `pi` sont les valeurs des propriétés formant la clé primaire.
- les _index_ déclarés pour la classe de document (s'il y en a). Par exemple pour la classe _Article_ `auteurs sujet taille`.
- `v` : version: _time_ de l'opération ayant créé / mis à jour / zombifié le document.
- `ttl`: TTL pour purge automatique par la DB, **format dépendant de la DB**. Absent pour un document vivant, présent pour un zombi (`data` est null) ou un document existant ayant une propriété `maxLife` non dépassée au moment de l'enregistrement en DB.
- `data` : binaire, jamais indexé. Sérialisation cryptée du _data_ du document. Pour un document _zombi_ `data` est null.

Les propriétés _index_ ci-dessus sont indexées:
- **une propriété par _sous-collection_** (par exemple `auteurs` pour un `Article`): ce peut être une valeur (_hash_) ou une liste de valeurs (_hash_).
- **une propriété par _index simple ou global_** (par exemple `taille` pour un `Article`): ce peut être un _string, integer, float, hash_.

# Sous-collections: mises à jour _incrémentales_ en sessions
La mémoire d'une session comporte **par organisation / classe**:
- **une mémoire par document** identifiée par sa `pk`. La copie en session d'UN document donné porte sa version `v`: toute mise à jour portant une version inférieure ou égal à `v` est ignorée (plus vieille ou déjà faite).
  - par document, la propriété `defs` donne la liste des identifiants des _abonnements_ qui ont provoqué sa présence: quand `defs` est vide, le document est _inutile_ car non référencé par aucune collection / sous-collection synchronisable. Il est supprimé de cette mémoire.
- **une mémoire par _sous-collection_**.

### Sous-collections en mémoire d'une session
Une sous-collection est identifiée dans la mémoire d'une session pour une organisation donnée par sa **définition** `def` _classe / nom de propriété / valeur de la propriété_ par exemple `Article/auteurs/XXX`:
- `XXX` est le shaS de l'identifiant de l'auteur (avec possiblement des /) 'a1/Zola'.

Une sous-collection a deux versions:
- version _serveur_ : date-heure de la notification la plus récente ayant notifié un changement. 0 si aucune notification n'a encre été reçue.
- version _détenue_ : date-heure de l'opération `Sync` qui en a retourné la mise à jour la plus récente. 0 si elle n'a pas encore été chargée.

Le principe de mise à jour incrémentale à `t1` d'une collection consiste à solliciter une opération retournant,
- tous les documents de la sous-collection `Article/auteurs/shaS(Zola)` de version postérieure à `t1` non zombi.
- tous les documents ayant eu 'Zola' dans la liste d'aureurs mais ne l'ayant plus.

Pour chaque document de cette liste la session sait retrouver:
- ceux étant dans la sous-collectio,
- ceux supprimés,
- ceux n'étant plus dans cette sous-collection: ceux n'étant plus dans aucune collection de la classe sont _inutiles_ et supprimés.

> A tout instant une session détient pour une classe donnée tous les documents référencés par une _souscription élémentaire_ (`def`):
>- soit à la classe,
>- soit à des documents cités par leur `pk`,
>- soit à des sous-collections citées par _nom de propriété / valeur de la propriété_.

### Imprécision de la _version_ d'une collection en mémoire d'une session
Pour un _document_ sa version n'a pas d'ambiguïté : même si les mises à jour ont été effectuées par des serveurs différents, le contrôle de _non régression_ pour chaque document garantit bien une progression chronologique réelle.

Ce n'est pas le cas quand il s'agit d'une _collection_:
- deux instances de serveurs peuvent avoir exécuté _simultanément_ et en parallèle deux opération impactant la collection `Article/auteurs/shaS(Zola)`:
  - Sur S1 : l'article 5 a une liste d'auteurs qui passe de `[Victor, Zola]` à `[Victor, Hugo]` dans une opération marquée `t2`.
  - Sur S2 : l'article 6 a une liste d'auteurs qui passe de `[Freud]` à `[Freud, Zola]` dans une opération aussi marquée `t2`.

En effet si les horloges des serveurs sont _à peu près_ synchronisées elles ont toutes la possibilité d'indiquer le même temps `t2` à des moments qu'il est impossible de comparer. En conséquence, demander **_toutes les modifications sur la collection `Article/auteurs/shaS(Zola)` intervenues après `t1`_** expose à obtenir des résultats différents:
- selon que le `t1` en question a été retourné par S1 ou S2,
- selon le serveur S3 à qui on pose la question et dont le _now_ de l'opération a encore une autre valeur.

Il faut donc accepter cette incertitude sur les _versions des collections_ et la gérer en sécurité:
- une demande des mises à jour portant sur `Article/auteurs/shaS(Zola)` transmet une version `tx` au serveur,
- le retour porte un _now_ d'opération `ty`: la _version_ enregistrée pour la collection remplaçant `tx` ne va pas être `ty` mais `ty - DELTA`.

> **En conséquence, une même mise à jour _peut_ parvenir plus d'une fois.**

### Sous-collections _synchronisables_
Il peut ne pas y en avoir.

Chaque sous-collection synchronisable est déclarée par:

    ['sujet', { key: ['sujet', 'sousSujet'], mutable: true }],
    ['auteurs', { key: ['autid'], mutable: true, list: true }]

- le nom 'auteurs' sera le nom de la propriété dans le format _row_ du document
- `key` : indique si la ou les propriété du document identifant la sous-collection.
- `mutable` : si ces propriétés sont immutables, il n'y pas de documents quittant la sous-collection (sauf zombification).
- `list` : si true, la valeur de la propriété est une **liste** d'auteurs (vide le cas échéant mais pouvant contenir plusieurs auteurs), sinon c'est UNE valeur (un sujet/soussujet).
  - soit LE nom d'une propriété du document (par exemple `auteurs`). 
  - soit UNE LISTE de noms de propriétés _string_ du document (par exemple `sujet, soussujet`). Dans le cas d'une liste, la propriété auteurs est une liste dont chaque terme de la liste est le shaS de la valeur de la propriété.

### Propriétés indexées: i0 i1 i2 ...

    ['volume',  { type: propType.FLOAT, global: true }]

- Il peut ne pas y en avoir.
- Chacune correspond à UNE propriété applicative.
- `global`: si true l'index à une portée _globale_ à toutes les organisations, sinon ce sont des propriétés utilisables uniquement dans les opérations trans organisations (de facto d'administration).
- `type` : 
  - `HASH` : shaS de la propriété string applicative. Le seul filtrage possible est sur égalité.
  - `STRING INT FLOAT` : c'est la propriété telle quelle qui permet les filtrages d'égalité et d'ordre.
  - `LIST` : la propriété est un array de string et le seul filtrage possible est `in`.

> Les propriétés _applicatives_, sauf `v maxLife` et celles indexées et de type non `HASH`, ne sont pas lisibles directement dans la base, même par son hébergeur: les clés primaires et secondaires sont des _hash_ et _data_ est cryptée. Il faut la clé de cryptage du site gérée confidentiellement par _l'administrateur technique_ pour en prendre connaissance.

### Index de liste de valeurs
Les propriétés indexées peuvent être de type `LIST`, avoir un array de valeurs. La sélection peut se faire avec l'opérateur `CONTAINS / CONTAINSANY` et rechercher les documents dont la valeur `myval1` est dans la liste des valeurs de leur propriété `myprop`.

### Synchronisation des sous-collections
Quand une classe de document est _synchronisable_ un document est enregistré par son row avec une version plus récente que la précédente.

Quand une sous-collection est basée **sur des propriétés immuables**, ce seul row permet de synchroniser, 
- la collection de la classe, 
- un row du document connu par sa `pk`, 
- toutes les sous-collections.
- si un `Article` est déclaré avec N auteurs et UN sujet à sa création et **ne peut plus en changer**, la sous-collection par `Article/auteurs` par exemple est synchronisable d'après les seuls documents `Article`.

**Mais si une sous-collection est basée sur une propriété qui PEUT changer**, par exemple par N auteurs **MAIS** pouvant changer d'auteurs (en avoir de nouveaux, en perdre):
- si un article passe des auteurs `Zola, Hugo` à l'auteur `Zola, Victor`, le row _standard_ va refléter ce changement:
  - la synchronisation de la sous-collection `Article/auteurs/shaS(Victor)` va bien recevoir un row supplémentaire.
  - MAIS la synchronisation de la sous-collection `Article/auteurs/shaS(Hugo)` ne sera PAS avertie du _retrait_ de `Hugo` au profit de `Victor`.

Pour gérer ce cas de changement de valeur d'une propriété identifiante d'une sous-collection, un second row `rowQ` (documents _quittés_) est inséré avec:
- pour classe / table `Article@auteurs`
- pour `pk` la valeur `shaS(Hugo) `(_l'ancien_ auteur),
- pour version `v` celle de l'opération ayant mis à jour (ou zombifié) le document _Article_.

La lecture de synchronisation pour l'abonnement `Article/auteurs/shaS(Hugo)` récupère:
- les documents `Article` ayant (depuis la précédente synchronization) dans leurs auteurs 'Hugo'.
- les `pk` des documents qui **ont perdu `Hugo`** (depuis la précédente synchronization) dans leurs liste d'auteurs. 

Les lectures de synchronisation, a) de **toute** la classe, b) ou **d'un** document par sa `pk`, ne reçoivent pas les rows _quittés_.

### Méthodes applicative sur un Document

`static mutate()` : cette méthode transforme un row d'une ancienne _release_ dans le format de la _release_ courante et permet de remettre _au nouveau format_ les rows en ayant un ancien.

`compile ()` : après transformation en un _document_ cette méthode (facultative) permet d'ajouter des propriétés _helpers_ au document destinées à faciliter leur consultation et mise à jour. Par convention leurs noms commencent par `_` afin qu'elles ne soient pas écrites en base ni transmises par synchronisation.

### Export / import
L'export des documents d'une base consiste à lire tous les documents _d'une organisation donnée_ pour chaque classe de document et pour chaque document:
- en obtenir son _data_ décrypté / désérialisé,
- changer si demandé son organisation,
- effectuer si demandé un traitement spécifique de la classe du document sur ce data,
- convertir ce data en format _row_ pour insertion sur la base d'import.

L'export / import peut ne concerner qu'une classe de document avec les objectifs suivants:
- simple audit de l'administrateur, sans cryptage dans la base importée, afin de pouvoir les lire _en clair_.
- remplacement après traitement d'une classe de document pour une organisation donnée:
  - export de A et import sans cryptage dans T,
  - suppression de A des documents de cette classe.
  - export de T et import dans A.

> L'export pour _toutes les organisations_ est une itération sur les organisations de l'export d'une seule.

# Provider _Firestore_ TODO
## Paths
Le path du singleton `Status` est `Status/1`.

Le path d'un document `Org` est `Org/demo`.

Le path des documents `Task` est `Org/demo/Task/kxc...` ou `Org/ROOT/Task/cbn...`. 
  - Par convention les tâches non liées à une organisation ont un code d'organisation `ROOT`.
  - leur `pk` `kxc...` désigne un document correspondant au couple d'un code de traitement de la tâche et d'un identifiant de sa cible.

Les souscriptions sont stockées sous deux classes:
- `Org/demo/Subs/sessionId`: pour l'entête de souscription.
- `Org/demo/SubsItem/vbn...` : pour les items de souscription.

Le path des autres classes, par exemple `Article`, sont:
  - `Org/demo/Article/kYc..`, des sous-documents de l'organisation.
    - `demo` est l'organisation,
    - `kYc...` est le sha16 du numéro de l'article.
  - `Org/demo/Article@auteurs/kert...` pour la sous-collection auteurs les pk des documents ayant quitté la sous-collection.

#### _Rules_ d'indexation
Le fichier est généré depuis le schema des types de documents.

### Gestion des _locks_
Un _lock_ permet en particulier de gérer des identifiants _externes_ uniques.

Pour chaque valeur possible `cvuk...` (le sha de son identifiant), un document `Lock` est créé avec pour path `Org/demo/Locks/cvuk...`.

`Lock` n'a qu'une propriété `owner` (string) qui est la clé complète du document référençant cette valeur: `cl1/pk`.

Pour utiliser un `Lock` une transaction doit effectuer **HORS tranasction** une requête `CreateLock` qui tente de _créer_ le lock :
- s'il n'existait pas la création est un succès et le lock existe désormais.
- s'il existait l'exception est récupérée et ... ignorée, le lock existe.
- dans les deux cas la valeur du lock est retournée pour information. 

Si le lock est _libre_, dans le cours normal de la transaction sa mise à jour pour réservation / libération est protégée par le _commit_ de la transaction.

### Fichiers à purger
Des fichiers stockés en _storage_ peuvent être marqués _à purger_ jusqu'à un jour donné: au delà de ce jour, ils peuvent être purgés du storage.

Le `path` d'un fichier d'une organisation en storage est de la forme `folderId/fid` :
- `folderId` est facultatif et son string peut contenir des /.
- `fid` est un string totalement identifiant en lui-même.

**NOSQL**
- le path d'un document est `Orgs/demo/FTP/path`
- sa propriété unique (et indexée) est `p`, date du jour de purge.

**SQL**
- la table a pour nom FTP.
- ses propriétés sont `org, path, p`. La clé primaire est `org, path`.

    async setFTP (org: string, path: string, dp: number) {}
    async purgeFTP (org: string, path: string) {}
    async listFTP (dp : number, fn: Function) {}
    async purgeAllFTP (dp : number) {}

### Gestion des tâches
Une tâche différée est représentée par une instance d'une classe héritant de `Task` (héritant de Document) ayant les propriétés suivantes:
- `starTime`: date-heure de création. Cette propriété est indexée de type STRING et est **global**. 
- `process` : classe de traitement de la tâche (sous-classe de `Task`).
- `target` : un string identifiant la cible du traitement.
- `nb`: une tâche peut être _itérative_ pour épuiser une liste. nb est le nombre d'itérations restant à effectuées.
- `exc`: code de l'exception rencontrée lors du dernier traitement.
- `endTime`: date-heure de fin.

Le `pk` est le shaS du couple `process, target`.

`'startTime endTime`' ont une forme string `AAAAMMJJhhmmssmmm` ce qui les rend comparables par relation d'ordre.

**NOSQL**
- le path d'une tâche est `Org/demo/Task/pk`
- `v` n'est pas fonctionnellement significatif. 

**Méthode spécifique**

    async nextTask (time: string) { return null }

## Règles d'indexation
Le texte est généré par le _provider_ dans un fichier `firestore-indexes.json` à installer dans Firestore.

## Création / mise à jour
Les _créations_ **exigent** que le document n'existe pas: emploi de la méthode `dr.create()`.

Les _mises à jour_ comme les _suppressions_ partent d'un document qui a été lu, donc verrouillé dans la transaction, et sont opérées par `dr.update()`. Les _créations_ sont opérées par `dr.create()`.

# Provider _sqlite_ 
TODO

## Tables: schema.sql
Le script de création des tables et des index est généré depuis le descriptif des document.

**SQL**
- table portant le nom `Task`.
- colonnes: `org, processTarget, startTime, data`

