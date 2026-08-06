---
layout: page
title: Documents et fichiers, souscriptions et synchronisations
---

# Documents et fichiers

## Document
Selon le standard JSON:
- deux **structures** de données sont utilisables:
  - des _listes_ ordonnées,
  - des _maps_, associations _clé (string) / valeur_.
  - chaque terme d'une structure peut être une _liste_ ou une _map_ ou une donnée primitive.
- les _données primitives_ sont des _string, number, boolean_ et par extension _binary_ (pas vraiment dans JSON qui prévoit de les transmettre en base64: toutefois le module _msgpack_ assume ce format en Javascript).

Un **document** est un agrégat de données de structure JSON ayant **une _map_ pour racine** et dont chaque nom / clé donne la valeur d'une propriété qui elle-même peut être une _liste_, une _map_ ou une donnée primitive.

Des _fichiers_ peuvent être attachés à un document:
- chaque descriptif d'un fichier est une _map_ de quelques propriétés (nom, type, taille ...).
- les descriptifs sont inscrits dans une map `_files_` de la racine du document.
- les contenus effectifs des fichiers sont des suites de bytes stockés à part dans un _storage_.

> Un document peut être volumineux et même _très_ volumineux en incluant ses fichiers attachés.

**Il y a plusieurs _classes_ de document** `svc$name`, chacune correspondant à une structure dont la racine est une map de _propriétés_.
- `svc`: code du _service_, préfixe des noms de classe pour éviter les collisions.
- `name`: nom de la classe dans le _service_.


**Une liste `pk` ordonnée de propriétés _string_ immuables constitue l'identifiant fonctionnel du document** (clé primaire en SQL, path en NOSQL). Cette `pk` peut ne contenir qu'un terme, le cas échéant généré aléatoirement à la création. Les propriétés identifiantes ne contiennent pas de `/`.

La propriété `v` version du document est le _time_ de l'opération qui l'a mise à jour. Une version ne régresse jamais pour un document donné.

### Schéma d'un document
Le schéma est déclaré ainsi:

    new DocDescriptor(svc,
      { ... }, //header
      new Map<string, collection>([ // 2 collections
        [ ... ],  // première collection
        [ ... ]   // seconde collection
      ]),
      new Map<string, idx>([  // 1 filtre
        [ ... ] // filtre 1
      ])  
    )

##### Header: premier paramètre
`{ name: 'Article', sync: true, pk: ['artid'] }, //header`
- `name` : son nom (de table / classe de documents).
- `sync: true` : la classe de documents est synchronisée.
- `pk: ['artid']` : sa clé primaire n'est constituée ici que d'une seule propriété (il pourrait y avoir une liste ordonnée). Si `pk` est absent la classe est un singleton dont par convention l'unique document à pour `pk` la valeur `1`.
- `nohash?: boolean`. La `pk` de valeur `p1/p2/p3` (ou juste `p1`) est normalement hachée pour ne pas faire apparaître en base de données des valeurs fonctionnellement signifiante. La présence de `nohash` supprime le hachage.
- `virtual?: boolean`. La classe est virtuelle, n'a pas d'instance et _n'existe_ que par la liste énumérée de ses pk.
- `enum?: string[]`. Pour une classe virtuelle c'est la liste de ses `pk`.
- `extenum?: string = ''`. Pour une classe virtuelle c'est la clé d'un _SINGLETON_ de configuration dont la valeur est le JSON de la liste de ses `pk`.
- `subClassBy?: string = 'sc'` . Ordinairement un document `svc$name` donne lieu en mémoire à une instance d'un document de classe `svc$name`. Il est possible d'instancier des objets de sous-classes `svc$name_v1` `svc$name_v2` ... où  `v2` ... sont les valeurs d'une propriété _immuable_ `sc` du document (appartenant ou non à la `pk`).
- `sync?: boolean`. Si le document est _synchronisé_ ou non.
- `embedCreds?: boolean` Si _true_ le document possède une propriété `creds` qui est une map dont chaque entrée correspond à un _credential_ ayant ce document pour document maître. Sinon, les _credential_ associé sont des documents séparés de classe `svc$Credential_name` ayant pour propriété,
  - `docCl` avec la valeur `name`,
  - `docPk` avec la valeur de la `pk` du document maître.

##### Collections: second argument
`new Map<string, collection>([['section',  { key: ['section'], mutable: true }], ...])`

- `key: string[]` . Liste des noms des propriétés définissant la collection.
- `mutable: boolean`. Deux sous-catégories:
  - `mutable: true` : la propriété (ou le N-uplet) définissant la collection peut être mise à jour.
  - `mutable: false` : la propriété (ou le N-uplet) est immuable pour le document après sa création.
- `list?: boolean`. Si _true_ la propriété est multi-valuée.

##### Indexations
`new Map<string, idx>([['nom',  { type: propType.STRING, key: ['nomAuteur'], testable: true }], [...])`

Une propriété filtrée permet d'obtenir une liste _report_ des documents de cette classe filtrés selon la valeur de cette propriété. Deux sous-catégories:
  - `global: false` : les documents filtrés sont de la même organisation.
  - `global: true` : les documents peuvent être de n'importe quelle organisation, mais toutes hébergées dans la même base: un tel filtre est réservé à des documents techniques pour des opérations _d'administration_.

- `type: propType`
- `global?: boolean`
- `key: string[]` : propriété ou N-uplet de propriétés, indexé.
- `testable?: boolean` : si _true_ l'existence d'un document ayant cette propriété peut être testée sans disposer pour autant d'un _credential_ d'accès au document.
- `nohash?: boolean`

Un filtre a un _type_ de données: `enum propType { STRING, INTEGER, FLOAT, LIST, HASH }`
- `STRING` : la valeur de la propriété est un _string_.
- `INTEGER` : nombre sur **32 bits**.
- `FLOAT` : flottant en double précision.
- `LIST` : la valeur de la propriété est une liste de _strings_.
- `HASH` : permet d'indexer soit UNE propriété `sujet`, soit un N-uplet de propriétés `[sujet, sousSujet]`.
  - la ou les propriétés sont des _strings_.
  - la valeur de filtre est **opaque**: c'est un hash de la valeur ou de la liste des valeurs pour un N-uplet.

Les types `STRING, INTEGER, FLOAT` supportent des filtres d'égalité et de comparaison: `LT LE EQ GE GT`. Exemple: `volume GE 100`

Le type `LIST` ne supporte que le filtre `CONTAINS CONTAINSANY`. Exemple : `membres CONTAINS 'Bob'` ou `membres CONTAINSANY 'Bob, Alice'`.

Le type `HASH` ne supporte que des filtres d'égalité `EQ`.

> Un **report** N'EST PAS SYNCHRONISE: une fois calculé en utilisant un _filtre_ il reste tel quel. Pour être _rafraîchi_ il doit être redemandé / recalculé. Par opposition une **collection** est synchronisable, une session qui y est abonnée reçoit les _notifications de son changement_.

### Exemple du document `Article` dans le Use-case _revues_
Propriétés: la classe est _synchronisée_.
- `id` : générée aléatoirement.
- `auteurs`: liste des auteurs.
- `sujet`: sujet de l'article.
- `sousSujet`: sujet détaillé de l'article.
- `taille`: taille de l'article.
- `texte`: texte de l'article.
- `fichiers`: fichiers attachés et leur tailles.
- `volume`: volume total des fichiers attachés.

    new DocDescriptor( 'AS2',
      { name: 'Article', sync: true, pk: ['artid'] }, //header
      new Map<string, collection>([
        ['auteurs', { key: ['autid'], mutable: true, list: true }]
        ['sujet', { key: ['sujet', 'sousSujet'], mutable: true }],
      ]), // collections
      new Map<string, idx>([
        ['volume',  { type: propType.FLOAT, global: true }]
      ]) // index 
    )

Clé primaire: `pk: [artid]`

La classe est _synchronisée_: `sync: true`
- Une session peut s'abonner à UN document et être notifiée de son évolution.
- Une session peut s'abonner à LA CLASSE de document et être notifiée de tout ajout, suppression ou modification d'un `AS2$Article`.

#### Collections
`new Map<string, collection>([`

`['auteurs', { key: ['autid'], mutable: true, list: true }]`
- le nom de la collection est `auteurs`.
  - elle ne porte que sur une seule propriété `autid`.
  - `list: true` : un article peut avoir plusieurs auteurs,
  - `mutable: true` : la liste des auteurs peut changer au cours du temps (auteurs disparaissant, nouveaux auteurs).
- elle définit par exemple une **collection** `AS2$Article/auteurs/Hugo` : _la collection des articles dont un des auteurs est 'Hugo'_.

`['sujet', { key: ['sujet', 'sousSujet'], mutable: true }]`
- le nom de la collection est `sujet`.
  - elle porte sur un couple de propriétés `[sujet, sousSujet]`.
  - `sujet` n'est PAS une liste (`list` est absent): un article n'a QU'UN SEUL sujet/sous-sujet MAIS qui peut changer au cours du temps pour un article ( `mutable: true` ).
- elle définit par exemple une **collection** `AS2$Article/sujet/écologie/solaire` : _la collection des articles dont le (sujet/sous-sujet) est `écologie/solaire`.

#### Filtres d'indexation
`new Map<string, idx>([`

`['volume',  { type: propType.FLOAT }]`
- le filtre par volume permet des sélections de comparaison. Par exemple: tous les articles dont la volume est supérieur à 5000.

#### Exemple: usage des collections `AS2$Article/auteurs`
- une session peut _s'abonner_ à la collection `AS2$Article/auteurs/Zola`.
- elle recevra une notification à chaque fois que la liste des articles dont l'un des auteurs est `Zola` change (et quand `Zola` a été ajouté ou retiré de la liste des auteurs d'un article).
- elle pourra demander tous les articles de cette collection ayant changé ou ayant été ajouté ou ayant été retiré de cette collection depuis une version t1.

#### Exemple des synchronisations possibles
Pour une session donnée, les synchronisations possibles des documents `AS2$Article` sont:

    AS2$Article 
    AS2$Article/... 
    AS2$Article/auteurs/... 
    AS2$Article/sujdetail/...

- `AS2$Article/1234` : synchronisation de l'article par sa clé primaire '1234'.
- `AS2$Article/auteurs/sh(Hugo)` : liste synchronisée des articles dont 'Hugo' est un des rédacteurs.
- `AS2$Article/sujet/sh([écologie solaire])` : liste synchronisée des articles ayant pour sujet détaillé `[écologie, solaire]`
- la liste `AS2$Article` de tous les articles est synchronisable mais pourrait avoir un volume très important.
- la session peut synchroniser plusieurs collections auteurs (Hugo, Zola), plusieurs articles (1234, 5678, 9876) et plusieurs collections sujets détaillés.

> Par la suite afin d'alléger la notation le _préfixe_ 'svc$' devant le nom de classe n'est pas mentionné.

#### Vue d'un _auteur_
Un auteur peut voir:
- _synchronisé_ : sa propre fiche d'information.
- _synchronisé_ : la liste des articles dont il est un des auteurs. Si son identifiant est 'Hugo', l'abonnement `Article/auteurs/sh(Hugo)` fournit la synchronisation de cette liste.
- _synchronisé_ : la liste des chats auxquels il participe et le détail de chacun.
- la liste des sujets gérés, possiblement avec un filtre.

La demande de la collection `Article/auteurs/sh(Hugo)` à `t1`:
- fournit _intégralement_ **tous** les articles dont `Hugo` est un des auteurs.
- fournit _incrémentalement_ les articles créés, modifiés, supprimés ou n'ayant PLUS `Hugo` comme auteur depuis `t1`.

Un _auteur_ reçoit des _notifications_ textuelles:
- même quand l'application n'est pas lancée lorsqu'un de ses _chats_ évolue.
- quand l'application est lancée lorsqu'un de ses articles évolue.

### Suppression des documents synchronisables
Pour que les sessions _abonnées_ soient informées qu'un document a été _supprimé_ on opère une _suppression logique_, le document est marqué _zombi_ au lieu d'être _purgé_.
- Dans la base il a une propriété `ttl` (time-to-live) qui indique quand il sera physiquement purgé.
- Sa version `v` indique **exactement quand** il est devenu _zombi_.
- Après quelques mois, les sessions abonnées sont supposées avoir été synchronisées et le document est purgé physiquement. 
  - Les sessions n'ayant pas opéré une telle synchronisation devront effectuer une demande de liste _intégrale_ et non pas _incrémentale_ depuis t (date-heure de la dernière synchronisation incrémentale).

### Stockage d'un document d'une classe donnée
Le document est stocké dans une table (SQL) ou une collection (NOSQL) spécifique de la classe de document.
- pour chaque classe ayant des _collections_, pour chaque _collection_ **une table trace les documents retirés de la collection**.

**L'ensemble des propriétés** est sérialisé dans un champ dénommé `data`: ce contenu est désérialisable dans les applications terminales et les serveurs.

En base de données, les propriétés **visibles de la base de données** sont:
- `pk` : clé primaire ou path.
- les _index_ déclarés pour la classe de document (s'il y en a). Par exemple pour la classe _Article_ `auteurs sujet volume`.
- `v` : version: _time_ de l'opération ayant créé / mis à jour / zombifié le document.
- `ttl` : time-to-live.
  - dans la cas standard si `ttl` existe le document est _zombi_ (quelle que soit la valeur de `ttl`), n'existe plus logiquement et est candidat à suppression physique future à un instant exact indéfini.
  - si une propriété `maxLife` est déclarée **gérée par l'application**, c'est une date-heure (_epoch_) en minutes:
    - si `maxLife` est dans le passé: le document EST _zombi_.
    - si `maxLife` est dans le futur: le document SERA automatiquement considéré comme _zombi_ à échéance de cette date-heure. Il est ainsi possible de gérer de manière applicative des _dates limite de validité_ pour certaines classes de documents.
- `data`. Quand le document est _zombi_ `data` (_null_ en base de données) est reconstitué à la lecture avec les propriétés suivantes:
  - `v`: sa version.
  - `deleted`: true.
  - `_pk` sa clé primaire.

Le contenu structuré du document `data` est crypté, _opaque_ pour la base de données. Selon leur type, les propriétés de `data` sont remplacées ou non dans les _index_ par leur _hash_ (opaques également) sauf celles utilisables par les opérateurs _d'ordre_ `LT LE GE GT` qui sont conservées telles quelles.

# Fichiers attachés à un document
Un fichier est stocké en deux parties:
- son **descriptif** figurant dans le document (dans _files_).
- son **contenu effectif** stocké dans un **Storage** (de type AWS/S3, Google Storage, etc.).

Un fichier est identifié par `fid` un code aléatoire universel:
- un fichier **ne change pas** de contenu, un autre est créé avec un nouveau contenu.

Le descriptif d'un fichier a les propriétés suivantes: 
- `name` : texte dont la seule contrainte est d'être un nom acceptable dans un système de fichiers (ne pas contenir de `/` ...).
- `time` : date-heure de la transaction qui l'a validé.
- `type` : type _mime_ du fichier comme 'image/jpg'.
- `size` : taille en bytes (son _original non crypté_).
- `sha` : digest SHA256 de l'original non crypté.

La propriété `_files_` (dans `data`) du document est une _map_ avec une entrée `fid` par fichier et pour valeur le descriptif du fichier.

> Selon la logique de l'application, la propriété `name` **est ou non unique dans son document**. Si elle est unique, le stockage d'un fichier d'un nom donné supprime d'office le fichier portant antérieurement ce nom. Si la propriété `name` n'est contrainte à être unique, plusieurs fichiers porteront le même nom dans un document (avec des propriétés `time` différentes) vus comme autant de _révisions_ pour un nom donné.

Le contenu du fichier est stocké sous un _path_ dans l'espace de stockage `svc/org/folderId/fid`:
- `svc` : code du service (qualifiant), plusieurs services _pouvant_ se partager un même _storage_.
- `org` : code l'organisation.
- `fid` est suffisant pour garantir l'unicité du contenu.
- `folderId` définit un _folder_ de rangement et a une structure `a/b/c ...` (selon ce qu'a prévu le logiciel du service) dont le seul usage est de pouvoir purger en une seule commande tous les fichiers sous une partie de ce path, par exemple les fichiers dont le path commence par `a/b`.

### Protocole de stockage / suppression d'un ou plusieurs fichiers
Une ou plusieurs opérations de **preload** chargent le contenu du fichier dans le storage sous le path `folderId/fid`, `fid` étant généré à cet instant.

Avant le stockage physique la ou les opérations de _preload_ notent dans la table `FTP` (_filet-to-purge_) le couple (`folderId/id`, `date du jour`).

Une opération de validation enregistre ensuite dans le ou les documents concernés les nouveaux fichiers `fid` et leurs descriptifs. Cette opération s'accompagne éventuellement d'une liste de `fid` à détruire dans ces mêmes documents.
- pour chaque document la propriété _files_ est mise à jour.
- s'il y a des fichiers à détruire,
  - leurs entrées sont enlevés des propriétés _files_ de leurs documents.
  - leur couple (`folderId/id`, `date du jour`) est inséré dans la table `FTP`.
- la transaction est _validée par un commit_ de la base de données, les fichiers nouveaux _existent_ les fichiers supprimés n'existent plus (logiquement).

Après cette étape transactionnelle, une étape terminale prend place:
- les fichiers à supprimer sont effectivement purgés de leur répertoire de storage.
- les références des fichiers créés et de ceux supprimés sont purgées de la table `FTP`, cette seconde phase de transaction est validée par commit.

Les mises à jour comme les suppressions sont donc en deux phases et il se peut que suite à un incident une phase s'exécute et pas la seconde.

Un traitement périodique de nettoyage liste les fichiers inscrits dans `FTP` depuis plus d'un jour:
- ils sont purgés de l'espace de storage,
- ils sont purgés de la table `FTP`.

> Moyennant le respect de ce protocole simple, la gestion des fichiers dans un document bénéficie de la même sécurité transactionnelle que les autres propriétés du document.

# Abonnements d'une application à des _documents / collections_

Un _abonnement_ peut porter sur:
- **la collection complète des documents** d'une classe _Article_ synchronisée : définition `Article`.
- **UN document** d'une classe _Article_ et de clé primaire _1234_ : référence `Article/1234`.
- **la collection des documents** d'une classe _Article_ ayant une propriété _auteurs_ d'usage COL à pour valeur _Zola_: référence `Article/auteurs/sh(Zola)`.

Une application soumet sa liste initiale _d'abonnements_ en une fois au serveur mais peut ensuite l'étendre ou la réduire.

> Une session d'une application reçoit au fil de l'eau des avis de changement donnant **la liste des définitions** des abonnements dont le contenu a évolué: pour maintenir à jour une copie légèrement différée des documents concernés, la session interroge ensuite le serveur pour obtenir les documents eux-mêmes pour chaque abonnement ayant changé.  

### Abonnements ayant un texte de _pop-up_
Certaines définitions d'abonnements peuvent être utilisées comme _alertes_. 
- Quand un document change (par exemple `Chat`), les _définitions_ des abonnements `Chats....` concernés sont _poussés_ en notification des sessions abonnés.
- Quand un de ces abonnements a spécifié un _texte de pop-up_, celui-ci est affiché par le browser lors de sa réception, **que l'application soit lancée OU NON**.

> **Des pop-ups d'alertes peuvent ainsi s'afficher**, même quand l'application n'est pas lancée, et informer l'utilisateur qui peut cliquer sur le pop-up. Si l'application n'était pas lancée, elle l'est. Si l'application était lancée et que l'écran montrait un contenu concerné par cette annonce de changement, la vue à l'écran se synchronise au nouveau contenu. 

Suivant ce paradigme, une application présente à son utilisateur trois concepts:
- des **notifications d'alerte** annonçant des évolutions de documents ou de collections de documents qui l'intéresse: l'arrivée de nouveaux échanges sur un _chat_ (un document), une évolution de nomenclature d'articles (liste des documents Sujet). Elles **annoncent** par des notifications courtes une évolution de certains documents, mais n'en donne qu'un minimum d'information.
- des **notifications de documents synchronisés**: les documents correspondant sont systématiquement maintenus à jour dans l'application dans un état le plus proche techniquement possible de l'état des documents sur le serveur (du moins quand l'application est _au premier plan_).
- des **_rapports_**: ce sont des vues calculées à un instant donné et qui ne changent qu'à redemande du même rapport.

**Les documents synchronisés dans une application** le restent a minima tant que l'application est **au premier plan**:
- l'application **peut** décider de ne plus maintenir cette synchronisation quand elle passe **en arrière plan**: en pratique l'utilisateur ne voit d'une application en arrière plan que les _pop-ups_ de notification, c'est donc une économie de ressources que d'éviter de les synchroniser à cet instant.
- en repassant au premier plan, l'application **peut** demander aux serveurs de lui fournir les mises à jour des documents synchronisés depuis le dernier état mémorisé en session.

Ce fonctionnement est naturel pour une application _mobile_ où par principe une seule fenêtre / application est _visible_ et a le focus. Ce n'est plus si naturel dans un _browser_ qui peut avoir plus d'une fenêtre visible (même si une seule a le _focus_), voire des onglets _scindés_: que la vue n'ayant pas le _focus_ continue d'offrir une vue en mise à jour continue sous l'effet des notifications fait sens.

### Quand l'application n'est plus en exécution
Quand une application est en exécution elle **peut** rester _abonnée_ à des _alertes_.

Quand son exécution s'arrête, sauf décision explicite de l'utilisateur, certains de ces abonnements restent actifs, du moins un certain temps:
- les notifications correspondantes continueront à s'afficher en _pop-ups_, l'OS ou le browser de l'application s'en chargeant.
- l'utilisateur reste informé des _news_ auxquelles il était abonné.
- un clic sur un ces _pop-ups_ ouvre l'application ce qui lui permet plus ou moins directement de connaître en détail les documents ayant changé.

> Chaque application détermine pour chaque abonnement élémentaire auquel elle est abonné, si l'abonnement s'interrompt ou non quand l'application s'arrête.

# Souscriptions
Une session est identifiée de manière unique par son jeton de web-push `subJSON`: `sessionId` est le hash court de ce jeton.

### Souscription élémentaire: _définition_
Une souscription élémentaire est:
- (type 0) soit la souscription à la collection des documents de la classe: `Article`.
- (type 1) soit une souscription à UN document de la classe par sa clé primaire_. Par exemple `Article/sh(FR/3246)`, _L'Article_ d'identifiant FR/3246.
- (type 2) soit une souscription à une sous-collection de documents synchronisable : _classe du document / nom de propriété / valeur de la propriété_. Par exemple `Article/auteurs/sh(a7689)`, _les_ Articles ayant pour un de leurs _auteurs_ 'a7689'.

### Souscription: un document de la classe `svc$Subs`
Une _souscription_ définit les abonnements d'une session vis à vis d'un service `svc` et d'une organisation `org`. 

Une session d'une application intégrant plus d'un service et supportant de gérer plusieurs organisations pour un utilisateur, va avoir autant de documents _souscription_ que de couples `svc org` actifs dans sa session.

Toutes les souscriptions d'une même session pour un couple `svc org` donné figurent dans le même document:
- `subJSON` : le token de la session.
- `sessionId` : son shaS, `pk` du document.
- `title` : (facultatif) titre des notifications textuelles.
- `url` : (facultatif) URL d'ouverture de l'application sur clic d'une notification textuelle.
- `defs` : une map `{ def1: msg1, def2: msg2 ... }` donnant pour chaque définition de souscription élémentaire son _message_ textuel à afficher dans la notification ou '' s'il n'y en a pas.

Une session _s'abonne_ à des _synchronisations / notifications_ en soumettant une opération `SetSubscription` ayant en paramètre:
- le service et l'organisation cible (comme pour toute opération),
- `sessionId / subJSON` : session souscriptrice.
- les arguments `title url defs`.
- le booléen `longLife`: si _true_ la souscription reste valable quelques jours en l'absence de session active, sinon quelques heures. 

En fin de session, l'utilisateur décide s'il souhaite ou non continuer à recevoir des notifications textuelles: 
- soit une nouvelle souscription remplace celle courante avec un `longLife` à _true_.
- soit la souscription est supprimée.

#### Mise à jour de la base de données
Un document `Subs` mémorise la souscription globale et `sessionId` est la clé primaire.

Des documents annexes `SubsItem` sont gérés pour chaque élément de `defs` d'un row `{ (org), sessionId, def }`.

L'opération `UpdateSubscription` gère les souscriptions en cours de session (ajout / suppression, changement de message de souscriptions élémentaires).
- après lecture de la version déjà enregistrée et comparaison avec la nouvelle version, les souscriptions élémentaires déjà présentes sont inchangées, les nouvelles ajoutées et les autres supprimées.
- un index sur `def` permet de récupérer depuis les `SubsItem` toutes les sessions ayant la définition et de les notifier.

## Notification des sessions en fin d'opération
Une opération peut mettre à jour des documents, par exemple des `Article` et après _commit_ va émettre des notifications à toutes les sessions abonnées.

Pour chaque document de clé primaire `pk` mis à jour, il faut notifier les sessions:
- de la mise à jour du document,
- du changement des sous-collections.

Exemple 1:
- mise à jour d'un document Article : `{ pk: FR/3246, auteurs: [a7689], data: ... }` 
  - la propriété `auteurs` avait la valeur `a7689` et la conserve.
- les souscriptions élémentaires à notifier sont :
  - `Article/sh(FR/3246)`
  - `Article/auteurs/sh(a7689)` : la liste des Articles est inchangée mais l'Article FR/3246 a changé de contenu.

Exemple 2:
- mise à jour d'un document Article : `{ pk: FR/3246, auteurs: [-a7689, +a8887], data: ... }`
  - la propriété `auteurs` avait la valeur `a7689` et prend la valeur `a8887`.
- les souscriptions élémentaires à notifier sont :
  - `Article/sh(FR/3246)`
  - `Article/auteurs/sh(a7689)` : la liste des Articles est changée, déjà a priori réduite par l'évolution de la Article FR/3246 (mais peut-être d'autres).
  - `Article/auteurs/sh(a8887)`: la liste des Articles est changée, déjà a priori augmentée par l'évolution de la Article FR/3246 (mais peut-être d'autres)

### Process
#### Étape 1
Une map des sessions à notifier est créée avec:
- pour clé: le `sessionId` d'une session à notifier,
- pour valeur: le Set des définitions à notifier.

Depuis la liste des documents changés on calcule la liste des définitions des souscriptions élémentaires impactées. Pour chacune `def` de celles-ci,
- on obtient de `SubsItem` la liste des `sessionId` qui référence `def`. Pour chaque `sessionId`,
  - on obtient le document de souscription.
  - on ajoute le `def` et son _message_ éventuel.

#### Étape 2
Pour chaque session identifiée par `sessionId` à notifier on dispose du Set des `def` à _synchroniser_.

Pour pousser une notification à la session il faut obtenir son `subJSON` (dans son document de souscription) depuis son `sessionId`.

Chaque session concernée recevra UNE notification (pour une opération), avec:
- une liste des [def] ayant changé,
- un _texte de message_ (éventuellement vide) concaténation de tous les messages des `def` notifiées.

## Notifications avec _messages_
Les souscriptions élémentaires avec pour unique objectif de _synchroniser_ des documents n'ont pas de _texte de message_: la session réceptrice ne fait pas apparaître de pop-up qui, du fait de leur nombre potentiellement important, seraient inopportunes.

Certaines souscriptions ont pour but de provoquer l'affichage d'une pop-up pour alerter l'utilisateur: ce peut être le cas quand l'application n'est PAS lancée. 
- la donnée de _synchronisation_ (la liste des `def`) est inutilisée, non transmise à l'application (qui n'est pas exécution).
- en revanche la notification peut porter un _texte de message_ généré sur le serveur.
- le _titre_ de ce message est par défaut le nom de l'application et de l'organisation concernée.
- _l'url_ de ce message permet d'ouvrir l'application sur un clic de l'utilisateur sur la notification: cette url peut être munie d'un _query string_ qui détermine (éventuellement) les conditions d'ouverture de l'application (son affichage initial et ses documents à charger).

### Message associé à UNE souscription élémentaire
La souscription peut renseigner pour chaque souscription élémentaire le texte du message qui sera émis. Par exemple pour `Article.auteurs/sh(Hugo)` le message déclaré par l'application terminale pourrait être `"Maj d'un article de Hugo"`.

Le message suit les règles d'I18N de la session et c'est l'application terminale qui dispose des _noms en clair_ et autres données et sait lesquelles il convient d'afficher dans un pop-up de notification.

Une notification peut avoir plusieurs messages: ils sont concaténés sur des lignes séparées.
