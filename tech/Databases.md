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
- la **suppression** d'un document _SYNC_ s'effectue en le transformant en _zombi_: c'est une suppression logique dont la trace en base subsiste un certain temps afin que les bases synchronisées externes aient la possibilité d'être informées de leur _suppression_.

Les classes de documents NON synchronisables ne supportent pas d'abonnements.
- la **suppression** d'un document _NOSYNC_ est traitée une purge physique effective (_delete_ classique).

### _Existence_ d'un document
Un document **synchronisable** peut être _zombi_ (propriété `isZombi`):
- fonctionnellement il doit être considéré comme INEXISTANT.
- il peut être transmis à une session comme résultat d'une opération de _synchronisation_ afin que la session le supprime de son côté.
- en état _zombi_ le document n'a que quelques propriétés: 
  - `_deleted` : true
  - `v` : sa version, la date-heure de l'opération l'ayant zombifié.
  - les propriétés de sa clé primaire.
- un document _zombi_ a une durée de vie limitée en tant que zombi: au bout d'un certain temps il est effectivement détruit, les sessions distantes ne pourront plus être informés par synchronisation incrémentale de sa disparition. 

Un document **NON synchronisable** supprimé n'est a pas d'existence.

Un document **peut** avoir une propriété `maxLife`:
- c'est sa date-heure maximale de vie _logique_: au delà de cette date-heure le document est **fonctionnellement périmé**.
- sa lecture dans une opération **AVANT** cette limite retourne un document `d` normal ayant simplement une valeur dans le futur de `maxLife`: `d.isAlive(now)` est `true`
- sa lecture dans une opération **APRÈS** cette limite retourne,
  - soit un document `d` dont `d.isAlive(now)` est `false`.
  - soit AUCUN document (`d` est `null`) au delà d'un certain temps, sa durée de vie en tant que _zombi_ étant dépassée.

`maxLife` est une "epoch" en MINUTES: le nombre de minutes écoulées depuis le 1/1/1970. Depuis un time en ms, il suffit d'une division par 60000.

## Version des opérations / documents en base de données
Le _time_ d'une opération est sa date-heure UTC "epoch" prise au début d'une opération, et reprise au début de sa phase 2 (dans la _transaction_) : tout document mis à jour dans une opération a pour version la propriété `now` de l'opération.

Pour un document donné la version ne peut pas régresser: quand un document est lu (dans une _transaction_) si sa version est supérieure ou égale au `now` de l'opération, la transaction est sortie en exception `REGVER` (_régression de version_) et la transaction est relancée avec une nouvelle valeur de `now` de l'opération.

## Classes spéciales `Hdr Org Task Subs SubsItem Ftp`
`Hdr` est une classe synchronisable singleton représentant l'état global du _service_:
- elle ne peut être mise à jour que par une opération de niveau _administration_.
- sa _clé primaire_ par convention vaut '1'.
- elle n'a ni _sous-collections_, ni _propriétés indexées_.
- son organisation est par convention `ROOT`.
- son contenu / comportement est spécifique de chaque application.

`Org` est la classe dont chaque instance représente une _organisation_, son statut, etc.
- sa _clé primaire_ est par convention le code de l'organisation.
- son contenu / comportement est spécifique de chaque application.

`Task` est une classe NON synchronisable représentant une opération _différée_:
- son organisation est par convention `ROOT`.
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

`Ftp` _File to purge_ est une classe technique, PAS un document_ gérant le commit à 2 étapes des créations / suppressions de fichiers attachés à un document.

## Format _data_
_data_ est un objet ayant A MINIMA les propriétés suivantes:
- `v` : date-heure de l'opération l'ayant créé / mis à jour / zombifié.
- `release` : numéro de _release_ de la structure d'un document permettant à la lecture de convertir les documents ayant une ancienne structure dans celle la plus récente.
- `p1 p2 p3 i1 i2 ...` : propriétés string constituant de la _clé primaire_ du document.

> Un document _zombi_ (supprimé logiquement) a un _data_ ne contenant **que** les propriétés ci-dessus.

_data_ contient de plus,
- `maxLife`: facultative, "epoch" en MINUTES de fin de vie logique du document.
- les propriétés citées dans les index / sous-collection de sa classe.
- les autres propriétés _applicatives_ du document: 
  - elles ont n'importe quel nom commençant par une minuscule.
  - elles peuvent des `string, number, boolean, array, objet`.

**Quelques propriétés _techniques_** sont lisibles au cours d'une opération: 
- `_clazz`: la classe du document.
- `_org`: son organisation.
- `_status`:
  - `NONE`: le document a été lu mais pas (encore) mis à jour par l'opération.
  - `UPD`: le document a été lu et a été mis à jour par l'opération: cet état est à positionner à chaque mise à jour du document.
  - `NEW`: le document n'existait pas et a été créé ex-nihilo par l'opération.
  - `DEL`: le document a été lu ou créé par l'opération puis supprimé par elle.
- `_before`: objet technique contenant les valeurs des propriétés de sous-collection à la lecture du document depuis la DB.
- `_deleted`: document _zombi_.

Un objet _data_ peut être _sérialisé / désérialisé_ par `encode / decode` de `@msgpack/msgpack`: la propriété `data` d'un _row_ (objet connu de la base de données) est la sérialisation cryptée par la clé du site.

## Format _Document_
Une classe de document hérite de la class générique `Document`.

Une instance de document est créée depuis les propriétés d'un _data_. Quelques méthodes:
- `static mutate()` : réalise la mutation d'un document de release antérieure dans la dernière release.
- `compile()` : facultatif - permet de calculer certaines propriétés _helpers_ après initialisation.
- `decompile()` : facultatif - retire les propriétés _helpers_ ajouté ci-dessus.
- `serialForApp()` : facultatif - sérialise le document pour transmission / synchronisation à l'application terminale, en sélectionnant les propriétés à transmettre en fonction des autorisations de l'application.

## Format _row_
C'est un objet représentant le document stocké en DB.

En base de données, les propriétés **visibles de la base de données** sont:
- `pk` : clé primaire ou path.
  - pour les documents de classe `Org`, `pk` est le code de l'organisation.
  - pour les documents de classe `Hdr` qui sont des singletons, `pk` vaut `1`.
  - pour les autres classes, `pk` est le shaS de `p1/p2/...` ou les `pi` sont les valeurs des propriétés formant la clé primaire.
- les _index_ déclarés pour la classe de document (s'il y en a). Par exemple pour la classe _Article_ `auteurs sujet taille`.
- `v` : version: _time_ de l'opération ayant créé / mis à jour / zombifié le document.
- `deleted`: true pour un document zombi.
- `maxLife`: (le cas échéant) date-heure de fin de vie programmée par l'application ("epoch" en MINUTES)
- `ttl`: TTL pour purge automatique par la DB, format dépendant de la DB. Absent pour un document vivant, présent pour un zombi ou un document ayant une propriété `maxLife`.
- `data` : binaire, jamais indexé. Sérialisation cryptée du _data_ du document.

Les propriétés _index_ ci-dessus sont indexées:
- **une propriété par _sous-collection_** (par exemple `auteurs` pour un `Article`): ce peut être une valeur (_hash_) ou une liste de valeurs (_hash_).
- **une propriété par _index simple ou global_** (par exemple `taille` pour un `Article`): ce peut être un _string, integer, float, hash_.

# Sous-collections: mises à jour _incrémentales_ en sessions
La mémoire d'une session comporte:
- **une mémoire par document** (par classe et organisation) identifiée par sa `pk`. La copie en session d'UN document donné porte sa version `v`: toute mise à jour portant une version inférieure ou égal à `v` est ignorée (plus vieille ou déjà faite).
  - par document, la propriété `defs` donne la liste des identifiants des _abonnements_ qui ont provoqué sa présence: quand `defs` est vide, le document est _inutile_ car non référencé par aucune collection / sous-collection synchronisable. Il ne sera plus mis à jour et peut être supprimé de cette mémoire.
- **une mémoire par _sous-collection_**.

### Sous-collections en mémoire d'une session
Une collection est identifiée dans la mémoire d'une session pour une organisation donnée par sa **définition** `def` _classe / propriété @ valeur_ par exemple `Article/auteurs@XXX`:
- `XXX` est le shaS de l'identifiant de l'auteur (avec possiblement des /) 'a1/Zola'.

Une sous-collection a deux propriétés: 
- `v` : `t1` la date-heure de l'opération qui en a retourné la mise à jour la plus récente. 0 si elle n'a pas encore été chargée.
- `pks`: la liste des `pk` des documents de la sous-collection (ce qui permet d'en obtenir le contenu dans la mémoire des documents).

Le principe de mise à jour incrémentale à `t1` d'une collection consiste à solliciter une opération retournant,
- tous les documents de la sous-collection `Article/auteurs/shaC(Zola)` de version postérieure à `t1` non zombi.
- la liste `lpkv` des couples `[pk, v]` des documents de clé pk ayant quitté la sous-collection à l'instant v (le cas échéant par _zombification_) depuis `t1`. 

Cette opération a un _time_ `t2` qui replace `t1` dans l'objet sous-collection.

### Imprécision de la _version_ d'une collection en mémoire d'une session
Pour un _document_ sa version n'a pas d'ambiguïté : même si les mises à jour ont été effectuées par des serveurs différents, le contrôle de _non régression_ pour chaque document garantit bien une progression chronologique réelle.

Ce n'est pas le cas quand il s'agit d'une _collection_:
- deux instances de serveurs peuvent avoir exécuté _simultanément_ et en parallèle deux opération impactant la collection `Article/auteurs/shaC(Zola)`:
  - Sur S1 : l'article 5 a une liste d'auteurs qui passe de `[Victor, Zola]` à `[Victor, Hugo]` dans une opération marquée `t2`.
  - Sur S2 : l'article 6 a une liste d'auteurs qui passe de `[Freud]` à `[Freud, Zola]` dans une opération aussi marquée `t2`.

En effet si les horloges des serveurs sont _à peu près_ synchronisées elles ont toutes la possibilité d'indiquer le même temps `t2` à des moments qu'il est impossible de comparer. En conséquence, demander **_toutes les modifications sur la collection `Article/auteurs/shaC(Zola)` intervenues après `t1`_** expose à obtenir des résultats différents:
- selon que le `t1` en question a été retourné par S1 ou S2,
- selon le serveur S3 à qui on pose la question et dont le _now_ de l'opération a encore une autre valeur.

Il faut donc accepter cette incertitude sur les _versions des collections_ et la gérer en sécurité:
- une demande des mises à jour portant sur `Article/auteurs/shaC(Zola)` transmet une version `tx` au serveur,
- le retour porte un _now_ d'opération `ty`: la _version_ enregistrée pour la collection remplaçant `tx` ne va pas être `ty` mais `ty - DELTA`.

En conséquence, une même mise à jour _peut_ parvenir plus d'une fois.

### Maintien en session de la cohérence entre _documents_ et _sous-collections_
L'opération retournant une _sous-collection_ comme `Article/auteurs/shaC(Zola)` à `t1` retourne:
- une ligne _document_ au plus pour chaque `pk`: **la version la plus récente**. Il faut ne prendre en compte un tel document que si sa version est postérieure à celle déjà connue en session.
- pour chaque pk, zéro, une voire lignes indiquant les pk (et leur version) des documents ayant quitté la sous-collection. Il ne faut considérer que les _retraits_ de la sous-collection cohérents avec la version du document correspondant.

> La liste des _mises à jour postérieures à t1_ d'une collection est en conséquence _imprécise_: chaque session doit en contrôler la pertinence selon la version la plus récente de chaque document de cette liste dans la mémoire _documents_ de la session.

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
  - `string int float` : c'est la propriété telle quelle qui permet les filtrages d'égalité et d'ordre.
  - `list` : la propriété est un array de string et le seul filtrage possible est `in`.

**Une propriété _unique_** impose qu'il n'y ait jamais deux documents ayant cette même valeur.

> Les propriétés _applicatives_, sauf `v z` et celles indexées et de type non `hash`, ne sont pas lisibles directement dans la base, même par son hébergeur: les clés primaires et secondaires sont des _hash_ et _data_ est cryptée. Il faut la clé de cryptage du site gérée confidentiellement par _l'administrateur technique_ pour en prendre connaissance.

### Index de liste de valeurs
Les propriétés indexées peuvent être de type `list`, avoir un array de valeurs. La sélection peut se faire avec l'opérateur `IN` et rechercher les documents dont la valeur `myval1` est dans la liste des valeurs de leur propriété `i2`.

### La propriété `del` d'un row
Quand une classe de document est _synchronisable_ un document est enregistré par son row avec une version plus récente que la précédente.

Quand les sous-collections ne sont basées **QUE sur des propriétés immuables**, ce seul row permet de synchroniser, la collection de la classe, un row du document par sa `pk`, toutes les sous-collections.
- si un `Article` est déclaré avec UN auteur et UN sujet à sa création et ne peut plus en changer, les sous-collections par auteur par exemple sont synchronisables.

**Mais si une sous-collection est basée sur une propriété qui PEUT changer**, par exemple par UN auteur **MAIS** peut changer d'auteur:
- si un article passe de l'auteur 'Hugo' à l'auteur 'Victor', le row _standard_ va refléter ce changement:
  - la synchronisation de la sous-collection `Article.auteur:Victor` va bien recevoir un row supplémentaire.
  - MAIS la synchronisation de la sous-collection `Article.auteur:Hugo` ne sera PAS avertie du _retrait_ de Hugo au profit de Victor.

Pour gérer ce cas de changement de valeur d'une propriété identifiante d'une sous-collection, un second row est inséré avec:
- la même `pk` et `v`,
- pour auteur la valeur Hugo (_l'ancien_ auteur),
- l'indicateur `del` à 1 : ceci signifie que c'est un retrait / suppression de la sous-collection auteur (et de toute autre si plusieurs propriétés de sous-collection changent).
- `z` prend la valeur du jour de version.
- `data` est _réduit_, comme pour un zombi,
  - à `v`,
  - aux propriétés de la clé primaire,
  - à `del`.

La lecture de synchronisation pour l'abonnement `Article.auteur:Victor` ne récupère qu'un row (sans `del`).

La lecture de synchronisation pour l'abonnement `Article.auteur:Hugo` ne récupère qu'un row AVEC `del` ce qui indique que cet article ne FAIT PLUS PARTIE de la sous-collection.

Les lectures de synchronisation de toute la classe ou d'un document par sa pk, ignorent les rows ayant un `del`.

### Méthode Document.toRow(data): objet
Cette méthode utilise le schéma des documents pour savoir générer les propriétés `pk ski ii` depuis les valeurs des propriétés applicatives constitutives trouvées dans data:
- elle retourne l'objet row correspondant au data.
- data du row est la sérialisation cryptée par la clé du site de l'objet data passé en argument.

### Méthode Document.toData(row): objet
Cette méthode décrypte row.data par la clé du site et désérialise le résultat.

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

# Provider _Firestore_
## Paths
Le path du singleton `Hdr` est `Hdr/hdr`.
Le path du singleton `Ping` est `Hdr/ping`.
Le path d'un document `Org` est `Org/demo`.

Le path des autres classes, par exemple `Article`, sont `Org/demo/Article/kYc..`, des sous-documents de l'organisation.
- `demo` est l'organisation,
- `kYc...` est le sha16 du numéro de l'article.

**Toutes** les propriétés des _rows_ sauf `data` sont indexées basiquement, toutefois les propriétés indexées marquées `G` doivent être déclarées :

    "queryScope": "COLLECTION_GROUP"
    (au lieu de "COLLECTION" pour les autres)

C'est aussi le cas pour la propriété `z` pour pouvoir _purger_ les vieux zombis.

Chaque classe de _document_ (sauf `Hdr Task`) et de _fil_ doit avoir une déclaration spécifique qui permette de la filtrer sur son égalité de `k0` ET sa version avec `>`:

    {
    "indexes": [
      {
        "collectionGroup": "avatar",
        "queryScope": "COLLECTION",
        "fields": [
          {
            "fieldPath": "k0",
            "order": "ASCENDING"
          },
          {
            "fieldPath": "v",
            "order": "ASCENDING"
          }
        ]
      },

## Création / mise à jour
Les _créations_ **exigent** que le document n'existe pas: emploi de la méthode `dr.create()`.

Les _mises à jour_ partent d'un document qui a été lu, donc verrouillé dans la transaction, et est opéré par `dr.update()`.

## Gestion des index _unique_
Pour chaque valeur possible `uk` (un sha16), un document `locks` est créé, avec un path dépendant si la propriété est globale ou non:
- `locks/uk` : portée globale.
- `org/demo/locks/uk` : portée locale d'une organisation.

Lock n'a qu'une propriété owner (string) qui est la la clé complète du document référençant cette valeur: `[demo, cl1, pk]`.

Ces propriétés ne sont SONT PAS INDEXÉES, l'accès se fait par une méthode spécifique lisant _locks_.

# API d'accès primaire générique

Les accès retournant une _liste_ peuvent avoir comme dernier paramètre une fonction fn anonyme qui reçoit en argument chaque _data_ et la traite. Cette facilité permet d'éviter d'accumuler des listes longues quand la _data_ peut être transformée / traitée une par une.

## Accès aux documents / fils / tasks

    async getDoc (org: string, cl: string, pk: string, v?: number) { return null }
    async insertDoc (row: Object) {}
    async updateDoc (row: Object) {}
    async deleteDoc (org: string, cl: string, pk: string) {}
    async listDocs (org: string, cl: string, v?: number, fn? : Function) { return [] }
    async listDocsSk (org: string, cl: string, ik: number, val: string, v?: number, fn? : Function) { return [] }
    async listDocsIdx (org: string, cl: string, ix: number, comp: string, val: any, v?: number, fn? : Function) { return [] }

## Accès `Hdr`

    async getHdr (v? : number) { return null }
    async insertHdr (row: Object) {}
    async updateHdr (row: Object) {}

## Accès `Org`

    async getOrg (org: string, v?: number) { return null }
    async insertOrg (row: Object) {}
    async updateOrg (row: Object) {}
    async listOrgs (v?: number, fn? : Function) { return [] }  
    async listOrgsIdx (ix: number, comp: string, val: any, v?: number, fn? : Function) { return []}

## Purges hors transaction

    async purgeAllDocs (org: string, cl: string) {}
    async purgeOrg (org: string, z?: number) {}
    async purgeOrgs (org: string, z: number) {}
    async purgeDlvDocs (org: string, cl: string, ix: number, comp: string, val: any, lsp?: string[]) {}

# Autres _tables_ / clazzes de documents_

## Fichiers à purger
Des fichiers stockés en _storage_ peuvent être marqués _à purger_ jusqu'à un jour donné: au delà de ce jour, ils peuvent être purgés du storage.

Le `path` d'un fichier d'une organisation en storage est de la forme `folderId/fid` :
- `folderId` est facultatif et son string peut contenir des /.
- `fid` est un string totalement identifiant en lui-même.

**NOSQL**
- le path d'un document est `Orgs/org/FTP/path`
- sa propriété unique (et indexée) est `p`, date du jour de purge.

**SQL**
- la table a pour nom FTP.
- ses propriétés sont `org, path, p`. La clé primaire est `org, path`.

    async setFTP (org: string, path: string, dp: number) {}
    async purgeFTP (org: string, path: string) {}
    async listFTP (dp : number, fn: Function) {}
    async purgeAllFTP (dp : number) {}

## Gestion des tâches
Une tâche différée est représentée par une instance d'une classe héritant de `Task` (héritant de Document) ayant les propriétés suivantes:
- `clazz` : `Task`.
- `org` : code l'organisation.
- `starTime`: date-heure de création. Cette propriété est l'index 0 (`i0` du row), de type string / global. 
- `process` : classe de traitement de la tâche (sous-classe de `Task`).
- `pk` : array des identifiants de la cible du traitement.
- `nb`: une tâche peut être _itérative_ pour épuiser une liste. nb est le nombre d'itérations restant à effectuées.
- `exc`: code de l'exception rencontrée lors du dernier traitement.
- `endTime`: date-heure de fin.

La propriété `k0` du row correspondant est le base64 du sha16 de l'array `processPk`.

`'startTime endTime`' ont une forme string `AAAAMMJJhhmmssmmm` ce qui les rend comprables par relation d'ordre et plus lisible.

**NOSQL**
- le path d'une tâche est `Orgs/org/Task/pk`
- les propriétés sont `org k0, i0, data`.
- il n'y a ni `v` ni `z`. 

**SQL**
- table portant le nom `Task`.
- colonnes: `org, k0, i0, data`

**Méthode spécifique**

    async nextTask (time: string) { return null }

