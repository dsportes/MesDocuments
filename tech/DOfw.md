---
layout: page
title: Conceptions d'applications réactives selon le paradigme de "Documents synchronisées"
---

Le mot _application_ (Facebook, TikTok ...) correspond à une architecture schématiquement en deux niveaux:
- des _serveurs centraux_ détiennent les données et effectuent les calculs sollicités par des demandes émises par ...
- des _applications terminales_ s'exécutant sur les terminaux / appareils des utilisateurs et sollicitant les serveurs centraux.

### Installation de l'application sur un appareil / terminal (_device_)
Un PC, une tablette, un mobile sont des _appareils / terminaux_ munis d'un moyen de communication avec un humain (écran, clavier, souris ...).

Selon la variante technique choisie, un utilisateur démarre une application:
- soit dans un browser _en ouvrant la page Web_ de l'application.
- soit _en l'installant_ sur un de ses appareils puis en la lançant.

#### Application de type Web : PWA _Progressive Web Application_
Depuis un browser l'utilisateur appelle une URL d'un _magasin d'applications_ qui ouvre la page d'accueil de l'application:
- l'application peut être directement utilisable depuis cette page.
  - L'utilisateur peut déclarer un _raccourci sur son bureau_ ou dans son browser vers cette URL afin d'éviter la ressaisie de celle-ci. 
  - Certains OS (comme iOS) des appareils ne permettent pas une utilisation directe d'une telle page Web et oblige à une _installation_, au demeurant simple, de l'application depuis cette page.
- l'application _peut ou doit_ selon le browser utilisé et l'OS de l'appareil, être _installée_ par le browser. Elle apparaît ensuite comme une application locale de l'appareil avec une icône de lancement, typiquement sur le bureau.

Le changement de version d'une application PWA est automatique: au lancement la vérification d'existence et le téléchargement d'une nouvelle version est automatique, l'utilisateur étant convié à appuyer sur un bouton pour redémarrer celle-ci.

#### Application de type _mobile_
L'utilisateur l'installe depuis le ou un des magasins d'application supportés par l'OS du mobile.

Le changement de version est en général automatique mais peut être opéré manuellement.

> Il n'y a ensuite quasiment pas de différence perceptible par l'utilisateur à l'utilisation de l'application, il clique sur une icône pour l'ouvrir (la lancer).

> On peut installer une application Web-PWA sur un mobile: elle est dans la suite du document considérée comme application PWA (et non comme _mobile_).

### Serveurs
Les applications en exécution sur leur appareil envoient à des **serveurs centraux** des demandes de mise à jour et consultation des données de l'application.

Le terme générique de _serveurs_ recouvre des variantes techniques non perceptibles de l'extérieur:
- **Serveurs permanents**: plusieurs processus sont en exécution en permanence afin de traiter les requêtes qui leur parviennent sur l'URL du pool de processus et ont été routées vers l'un ou l'autre.
- **Cloud Functions**: un _serveur éphémère_ du _Cloud_ est lancé pour traiter une demande de service reçue sur son URL:
  - la demande est traitée et le serveur éphémère reste actif un certain temps pour traiter d'autres demandes. Un serveur éphémère peut traiter plusieurs dizaines de demandes en parallèle.
  - en l'absence de nouvelles demandes, un serveur éphémère reste en attente, entre 3 et 60 minutes pour fixer les idées, puis s'interrompt.
  - si le flux des demandes sature la capacité d'un serveur éphémère, un deuxième, voire un troisième etc. sont lancés.

> Ces choix de déploiement technique ne sont pas détectables par les applications terminales qui envoient des demandes aux serveurs.

# Les _services_ et leurs _prestataires_
Un _prestataire de services_ est une organisation / entreprise / association disposant techniquement de serveurs et proposant des services centralisés. Chaque service à une **finalité applicative** bien délimitée, par exemple:
  - le service `circuitscourts` : gestion de prises de commandes entre des producteurs et des consommateurs.
  - le service `discussions` : gestion de groupes de partage de documents et d'échanges interactifs.
  - le service `randos` : proposition de randonnées, inscription, échanges, etc.
  - le service `boutiques` : gestion du catalogue d'une boutique, de son stock, etc.

Deux prestataires, par exemple **Rouge** et **Bleu**, peuvent proposer un même service:
- **Rouge** peut proposer `randos` et `discussions`,
- **Bleu** peut proposer `circuitscourts` et `randos`.

Les services `randos` proposés par **Rouge** et **Bleu** sont-ils _identiques_ ?
- **ils peuvent l'être** avec exactement le même logiciel, pas forcément à tout instant la même version.
- **ils peuvent différer** avec un `randos/Rouge` proposant des fonctionnalités additionnelles par rapport à `randos/Bleu`.
- enfin ils peuvent différer selon le prix de leurs prestations et leur qualité d'usage: temps de réponse, disponibilité, restrictions de volume...

> Si le nom d'un service comme `randos` traduit sa fonctionnalité générale, un _service précis_ est désigné par le couple du **nom de service / nom du prestataire** le proposant : `randos/Rouge`. Ce couple se traduit par une URL qui permet de solliciter _ce_ service spécifiquement délivré par _ce_ prestataire.

# Les _applications terminales_, leurs magasins, leurs variantes
Le logiciel d'une application terminale est disponible dans un magasin: dans le cas d'une application Web le magasin est un site Wen statique dont chaque URL correspond à une application.

Pour afficher et gérer ses randos, il faut en conséquence installer l'application `randos` depuis un magasin. Pour un nom donné d'application, il peut exister le cas échéant plus d'une variante:
- `randos-mobile` par exemple peut se limiter au sous-ensemble des fonctionnalités utiles pendant la rando et sous un format simplifié adapté à consulter un écran de petite taille dans des conditions de luminosité pas optimale.
- `randos` par exemple propose toutes les fonctionnalités de l'inscription à la consultation d'historique en supposant une surface de lecture plus importante (PC ou tablette).

C'est à chaque utilisateur de choisir la **variante** qu'il veut installer (voire les variantes) quand plusieurs sont disponibles et selon chacun de ses appareils.

Les _variantes_ de l'application terminale `randos` ont pour caractéristiques de faire appel au service `randos` d'un des prestataires le proposant.

> Une application _terminale_ donnée peut avoir des exigences vis à vis du choix du prestataire de service. Un prestataire de service _haut de gamme_ peut proposer aussi dans un magasin une variante plus complète de l'application terminale avec des écrans accédant aux prestations supplémentaires qu'il offre.

# Organisations: services _multi-tenant_
Un service comme `randos`, peut à la manière de Discord, héberger les applications d'associations de randonneurs distinctes: chaque organisation / _tenant_ dispose de _son_ espace de données propre complètement étanche à celui des autres.

Un service `boutiques` propose de gérer plusieurs boutiques, pas une seule, mais de manière à ce que les données de chacune soient totalement isolées de celle des autres.

Les données d'un service d'un prestataire sont stockées dans deux _mémoires persistantes_ (voir plus loin):
- **UNE base de données** logiquement strictement partitionnée par organisation, sans aucun lien ou référence à des données / documents d'une organisation par une autre.
- **UN _storage_ de fichiers**, comme un directory de fichiers classiques, avec une **racine** par organisation.

### Pour un service donné, UNE organisation donnée n'est hébergée que par UN prestataire
Pour un service `randos` proposés par les prestataires **Rouge** et **Bleu**, une organisation donnée _val-de-bièvre_ est _hébergée_ chez **Rouge** ou chez **Bleu** mais pas dans les deux.

> Une organisation peut _migrer_ d'un prestataire à un autre: ce transfert technique des données est génériquement possible, sauf quand un prestataire a des données additionnelles absentes chez l'autre.

### Une application terminale peut accéder à plus d'une organisation
Dans le cas de l'application `randos`, un utilisateur peut être membre de plus d'une association de randonneurs: une pour ses randonnées près de chez lui, une autre pour les randonnées de montagne et une troisième pour les treks lointains. Depuis la même application il peut basculer d'une organisation à une autre.

Un gestionnaire de boutiques peut par exemple gérer trois boutiques différentes (trois organisations) avec des rôles différents pour chacune.

Les utilisateurs de Discord accèdent souvent à plusieurs _serveurs_ qui s'ignorent entre eux, ayant des sujets d'intérêt totalement différents.

L'utilisateur qui ouvre sur son appareil son application `randos` peut disposer de pages de synthèse lui montrant ce qui est important pour chacune des associations auxquelles il participe. Pour agir effectivement sur l'une d'entre elles, il basculera sur une page d'accueil spécifique de l'association sélectionnée et ses actions de mises à jour ne porteront que sur celle-là.

# L'exécution d'une application sur un appareil
Sur un appareil donné, on ne peut pas lancer plus d'une exécution d'une application donnée, par exemple une seule application `randos`.

> Dans le cas d'une application Web-PWA, chaque browser (Firefox, Chrome ...) est vu comme un **appareil différent**: on peut avoir s'exécutant au même instant sur son PC, une application sous Firefox ET une application sous Chrome.

**Une application sur UN appareil** peut avoir trois états:
- être en exécution au **premier plan**. Sa fenêtre est affichée et a le _focus_, elle capte les actions de la souris ou du clavier. Pour un mobile c'est celle, ou l'une des deux, visibles.
- être en exécution en **arrière plan** : elle a été lancée mais est recouverte par d'autres.
  - sur un browser, c'est un autre onglet qui a le focus ou la fenêtre du browser est en icône: l'utilisateur peut cliquer sur son onglet pour l'amener au premier plan ou sur l'icône du browser dans la barre d'icônes pour l'afficher.
  - sur un mobile elle est cachée mais peut être ramenée au premier plan quand l'utilisateur la choisit dans sa liste des applications _ouvertes mais cachées_.
- être **non lancée**: son exécution n'a pas encore été demandée, ou a été active puis fermée.

### Une application peut _envoyer_ des requêtes aux serveurs
C'est l'application qui appelle un serveur identifié par son URL: le serveur **traite la requête et retourne un résultat**.
- requêtes et réponses peuvent être volumineuses.

### Une application peut _écouter_ des notifications émises par les serveurs
Une application donnée sur un appareil donné est identifiée par un _jeton_ qui une sorte de numéro de téléphone universel: tout serveur ayant connaissance de ce jeton peut envoyer des _notifications_ à l'application correspondante sur le poste correspondant.

Une notification ressemble à un SMS:
- son texte est _court_ (certes plus long que celui d'un SMS).
- on ne répond pas à une notification: le serveur émetteur ne sait rien de la suite donnée, ou non, par l'application destinataire.

> L'application _peut_ en tenir compte et effectuer des traitements et des requêtes ultérieures aux serveurs.

**Quand l'application destinatrice d'une notification est au PREMIER PLAN:**
- elle _peut_ afficher un court message dans une petite fenêtre _popup_ (voire émettre un son ...) pour alerter l'utilisateur,
- elle _peut_ et généralement va, effectuer le traitement adapté aux données portées par la notification.

**Quand l'application destinatrice est en ARRIÈRE PLAN:**
- elle _peut_ (ou l'OS de l'appareil ou le browser dans lequel elle s'exécute) afficher en _popup_ la notification ce qui alerte l'utilisateur,
- si l'utilisateur clique sur cette _popup_, l'application correspondante repasse au premier plan.

**Quand l'application destinatrice N'EST PAS en exécution:**
- l'OS de l'appareil ou le browser dans lequel elle est enregistrée, _peut_ selon que l'utilisateur l'autorise ou non, afficher en _popup_ la notification ce qui alerte l'utilisateur,
- si l'utilisateur clique sur cette _popup_, l'application est lancée.

## Des applications _écoutantes_ réagissant au flux d'informations poussées
Les applications **_sourdes_** classiques ne peuvent afficher des écrans que sur sollicitation de l'utilisateur. L'écran ne se remet à jour que suite à une action de l'utilisateur: si ce dernier ne fait rien, l'écran ne change pas et affiche des données plus ou moins anciennes qui ont pu être déjà modifiées par l'action d'autres utilisateurs, du temps qui passe, etc.

Les applications **_écoutantes_** peuvent remettre à jour leurs écrans et données détenues localement même sans action d'un utilisateur simplement en fonction des _notifications_ poussées vers elles par les serveurs.

# Les stockages des données

## Pour un _service donné pour un prestataire donné_ 
Le prestataire dispose de deux stockages dédiés:
- une base de données,
- un _storage_ de fichiers.

Les stockages sont _partitionnés_ par _organisation_, une partition pour chaque organisation hébergée par ce service.

### La base de données
Elle gère les documents selon un mode _transactionnel_ (ACID).

Elle gère aussi les _abonnements_ des applications terminales aux _documents (synchronisables)_ qui les intéressent: chaque application sur un appareil a un _token_ qui l'identifie de manière unique. 

> Une _micro base de données locale_ pour chaque application / appareil peut détenir en _cache_ les _documents_ récemment demandés et les _abonnements_ en cours de l'application. 

Quand un ou des documents évoluent par exécution d'une opération, elle retrouve toutes les applications terminales abonnées et effectue une publication de notifications vers elles.

> Chaque application terminale est en conséquence susceptible de s'abonner éventuellement auprès de plus d'un prestataire si toutes les organisations de son domaine d'intérêt ne sont pas toutes gérées par le même prestataire.

### Le _Storage_ de fichiers 
Il stocke des _fichiers_ identifiés par leur _path_: la présence de caractères `/` dans un path définit une sorte d'arborescence. Le _contenu_ de chaque fichier est une suite d'octets opaque.

En lui-même il n'est pas soumis à un protocole transactionnel (ACID): sa sécurité transactionnelle est déportée sur la base de données avec un protocole simple à deux phases. Le couple base de données / storage permet de garantir qu'un fichier existe ou non, de manière atomique.

Le Storage permet de disposer d'un volume pratiquement 10 fois plus importants à coût identique par rapport à la base de données: de nombreuses applications ont des données historiques / mortes ou d'évolutions sporadiques qui s’accommodent bien d'un support sur Storage.

# Le répertoire des _organisations par application_

Toute _application_ terminale détient, en tant que ressource statique, la liste des _prestataires_ fournissant les services centraux, avec pour chacun:
- son _code_,
- son _URL d'accès_.
- la liste des _organisations_ hébergées.

L'ajout / retrait d'un prestataire et / ou d'une organisation  demande de générer une nouvelle version de l'application concernée. Toutefois une organisation pas encore _statiquement répertoriée_ peut être temporairement référencée par un utilisateur en indiquant le code du service qui l'héberge.

Une session d'une application terminale peut concerner plusieurs organisations, à l'instar du randonneur faisant partie de plusieurs associations selon l'endroit où il randonne. Pour chaque organisation concernée elle obtient de ce répertoire le prestataire gestionnaire et son URL d'accès.

Chaque service peut ensuite gérer **dans _sa_ base de données**,
- un document unique concernant toutes les organisations,
- un document relatif à chaque organisation.

Ces documents peuvent comporter par exemple:
- un **statut récapitulatif** : ouverture, restriction en lecture seule (archive), fermeture jusqu'à nouvel ordre.
- une **courte liste des dernières _news_ ayant modifié ce statut** données par l'administrateur.

Les applications terminales peuvent s'abonner aux modifications de ces documents et ainsi afficher pour la ou les organisations référencées dans leur session,
- son statut d'accessibilité, globalement et pour chaque organisation spécifiquement,
- les _news_ récentes ayant modifié ce statut.

# Documents et fichiers

## Document
Selon le standard JSON:
- deux **structures** de données sont utilisables:
  - des _listes_ ordonnées,
  - des _maps_, associations _clé (string) / valeur_.
  - chaque terme d'une structure peut être une structure ou une donnée primitive.
- les données primitives sont des _string, number, boolean_.

Un document est un agrégat de données de structure JSON ayant une map pour racine et dont chaque nom / clé donne la valeur d'une propriété qui elle-même peut être une structure ou une donnée élémentaire.

Des _fichiers_ peuvent être attachés à un document
- chaque descriptif d'un fichier est une _map_ de quelques propriétés (nom, type, taille ...).
- les descriptifs sont inscrits dans une map `_files_` de la racine du document.
- les contenus effectifs des fichiers sont des suites de bytes stockés à part dans un _storage_.

> Un document peut être volumineux et même _très_ volumineux en incluant ses fichiers attachés.

**Il y a plusieurs _classes_ de document**, chacune correspondant à une structure dont la racine est une map de _propriétés_.

**Une liste `pk` ordonnée de propriétés _string_ immuables constitue l'identifiant fonctionnel du document** (clé primaire en SQL, path en NOSQL). Cette `pk` peut ne contenir qu'un terme, le cas échéant généré aléatoirement à la création.

La propriété `v` version du document est le _time_ de l'opération qui l'a mise à jour. Une version ne régresse jamais pour un document donné.

### Les index d'une classe de document
Plusieurs _index_ peuvent être déclarés pour une classe de document. Un index peut avoir deux catégories d'usage:
- **_filtre_** : une propriété indexée permet d'obtenir une liste _report_ des documents de cette classe filtrés selon la valeur de cette propriété. Deux sous-catégories:
  - `SIMPLE` : les documents filtrés sont de la même organisation.
  - `GLOBAL` : les documents peuvent être de n'importe quelle organisation: usage réservé à d**es opérations _d'administration_.
- **_collection_ : la collection des documents de cette classe ayant une valeur donnée est _notifiable / synchronisable_. Deux sous-catégories:
  - `COL` : la propriété définissant la collection peut être mise à jour.
  - `IMUTCOL` : cette propriété reste constante pour le document après sa création.

> Un **report** N'EST PAS SYNCHRONISE: une fois calculé (en utilisant les index simple et global) il reste tel quel. Pour être _rafraîchi_ il doit être redemandé / recalculé. Par opposition une **collection** est synchronisable, une session qui y est abonnée reçoit les _notifications de son changement_.

Un index a un _type_ de données: `STRING, INTEGER, FLOAT, UNIQUE, LIST, HASH`
- `STRING` : la valeur de la propriété est un _string_.
- `INTEGER` : nombre sur 32 bits.
- `FLOAT` : flottant en double précision.
- `UNIQUE` : la valeur de la propriété est un _string_: UN seul document peut avoir cette valeur qui est stockée hachée en index. Sert à définir des clés identifiantes depuis un code fonctionnel alternatif à `pk`.
- `LIST` : la valeur de la propriété est une liste de _strings_.
- `HASH` : permet d'indexer soit UNE propriété `sujet`, soit un N-uplet `[sujet, sousSujet]`.
  - la ou les propriétés sont des _strings_.
  - l'index est **opaque**, c'est un hash de la valeur ou de la liste des valeurs pour un N-uplet.

Les index de type `STRING, INTEGER, FLOAT` supportent des filtres d'égalité et de comparaison: `LT LE EQ GE GT`. Exemple: `volume GE 100`

Les index de type `LIST` ne supporte que le filtre `CONTAINS`. Exemple : `membres CONTAINS 'Bob'`

Les index de type `UNIQUE HASH` ne supportent que le filtre `EQ`. Exemple : `sujet EQ 'écologie'`. La valeur de sujet n'apparaît pas en clair dans la base de données mais seulement son _hash_.

Les _collections_ ne peuvent être déclarées que sur les types `LIST HASH`.

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

Clé primaire `pk` : `[id]`
- la classe étant _synchronisée_.
  - Une session peut s'abonner à un document et être notifié de son évolution.
  - Une session peut s'abonner à la classe de document et être notifié de tout ajout, suppression ou modification d'un `Article`.

#### Index
`auteurs: { type: LIST, use: COL }`
- la propriété est auteurs est une liste et peut changer de valeur.
- elle définit une **collection** : `Article/auteurs/Hugo` définit la collection des articles dont un des auteurs est 'Hugo'.

`sujdet: { type: HASH, use: COL, props: [sujet, sousSujet]`
- le N-uplet de propriétés `[sujet, sousSujet]` forme un index de nom `sujdet`.
- il est de type HASH et permet une sélection sur égalité à un couple `[s1, ss1]`.
- elle définit (aussi) une **collection**: `Article/sujdet/écologie/solaire` définit la collection des articles dont le sujet détaillé est `écologie/solaire`.
- `sujdet` n'est pas une liste, un article n'a QU'UN SEUL sujet détaillé qui peut changer au cours du temps pour un document.

`taille: { type: INTEGER, use: SIMPLE }`
- l'index par taille permet des sélections de comparaison d'ordre.
- Par exemple: tous les articles dont la taille est supérieure à 5000.

**Exemple: usage des collections `Article/auteurs`**
- une session peut _s'abonner_ à la collection `Article/auteurs/Zola`.
- elle recevra une notification à chaque fois que la liste des articles dont l'un des auteurs est `Zola` change (et quand `Zola` ait été ajouté ou retiré de la liste des auteurs d'un article).
- elle pourra demander tous les articles de cette collection ayant changé ou ayant été ajouté ou ayant été retiré de cette collection depuis une version t1.

**Exemple des synchronisations possibles**
Pour une session donnée, les synchronisations possibles des documents `Article` sont `Article/pk Article/auteurs Article/sujdet`
- `Article/pk/1234` : synchronisation de l'article par sa clé primaire '1234'.
- `Article/auteurs/Hugo` : liste synchronisée des articles dont 'Hugo' est un des rédacteurs.
- `Article/sujet/écologie/solaire` : liste synchronisée des articles ayant pour sujet détaillé `[écologie, solaire]`
- la liste `Article` de tous les articles est synchronisable mais pourrait avoir un volume très important.
- la session peut synchroniser plusieurs collections auteurs (Hugo, Zola), plusieurs articles (1234, 5678, 9876) et plusieurs collections sujets détaillés.

#### Vue d'un _auteur_
Un auteur peut voir:
- _synchronisé_ : sa propre fiche d'information.
- _synchronisé_ : la liste des articles dont il est un des auteurs. Si son identifiant est 'Hugo', l'abonnement `Article/auteurs/Hugo` fournit la synchronisation de cette liste.
- _synchronisé_ : la liste des chats auxquels il participe et le détail de chacun.
- la liste des sujets gérés, possiblement avec un filtre.

La demande de la collection `Article/auteurs/Hugo` à `t1`:
- fournit _intégralement_ **tous** les articles dont Hugo est un des auteurs.
- fournit _incrémentalement_ les articles créés, modifiés, supprimés ou n'ayant PLUS Hugo comme auteur depuis `t1`.

Un _auteur_ reçoit des _notifications_ textuelles:
- même quand l'application n'est pas lancée lorsqu'un de ses chats évolue.
- quand l'application est lancée lorsqu'un de ses articles évolue.

### Suppression des documents synchronisables
Pour que les sessions _abonnées_ soient informés qu'un document a été _supprimé_ on opère une _suppression logique_, le document est marqué _zombi_ au lieu d'être _purgé_.
- La propriété `z` contient le jour de suppression _logique_.
- Après quelques mois, les sessions abonnées sont supposées avoir été synchronisées et le document est purgé physiquement. Les sessions n'ayant pas opéré une telle synchronisation devront effectuer une demande de liste _intégrale_ et non pas _incrémentale_ depuis t (date-heure de la dernière synchronisation incrémentale).

### Stockage d'un document d'un type donné
Le document est stocké dans une table (SQL) ou une collection (NOSQL) spécifique du type de document.

**L'ensemble des propriétés** est sérialisé dans un champ dénommé `data`: ce contenu est désérialisable dans les applications terminales et les serveurs.

En base de données, les propriétés **visibles de la base de données** sont:
- `pk` : clé primaire ou path.
- les _index_ déclarés pour la classe de document (s'il y en a). Par exemple pour la classe _Article_ `auteurs sujet taille`.
- `v` : version: _time_ de l'opération ayant créé / mis à jour / zombifié le document.
- `ck x z` : trois propriétés _techniques_ gérer les _collections_ et la suppression des documents de manière par synchronisation incrémentale.
- `data`.

Le contenu structuré du document `data` est crypté, _opaque_ pour la base de données. Selon leur type, les propriétés de data peuvent être remplacées dans les _index_ par leur _hash_ (opaques également) sauf celles utilisables par les opérateurs _d'ordre_  `LT LE GE GT` qui sont conservées telles quelles.

### Fichiers attachés à un document
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

Le contenu du fichier est stocké sous un _path_ dans l'espace de stockage `folderId/fid`:
- `fid` est suffisant pour garantir l'unicité du contenu.
- `folderId` définit un _folder_ de rangement et a une structure `a/b/c ...` dont le seul intérêt est de pouvoir purger en une seule commande tous les fichiers sous une partie de ce path, par exemple les fichiers dont le path commence par `a/b`.

### Protocole de stockage / suppression d'un ou plusieurs fichiers
Une ou plusieurs opérations de **preload** chargent le contenu du fichier dans le storage sous le path `folderId/fid`, `fid` étant généré à cet instant.

Avant le stockage physique la ou les opérations de _preload_ notent dans la table `FTP` le couple (`folderId/id`, `date du jour`).

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
- **la collection complète des documents** d'une classe _D1_ synchronisée : référence `D1`.
- **UN document** d'une classe _D1_ et de clé primaire _1234_ : référence `D1/pk/1234`.
- **la collection des documents** d'une classe _D1_ ayant une propriété _coll1_ d'usage COL à pour valeur _abcd_: référence `D1/coll1/abcd`.

Une application soumet sa liste _d'abonnements_ en une fois et attribue un _code numérique court_ à chacun.

> Une session reçoit au fil de l'eau des avis de changement donnant **la liste des codes** des abonnements dont le contenu a évolué: pour maintenir à jour une copie légèrement différée des documents concernés, la session interroge ensuite le serveur pour obtenir les documents eux-mêmes pour chaque abonnement ayant changé.  

### Références d'abonnement ayant un texte de _pop-up_
Certaines références d'abonnements peuvent être utilisés comme _alertes_. 
- Quand un document change (par exemple `Chat`), les _références_ des abonnements `Chats....` concernés sont _poussés_ en notification des sessions abonnés.
- Quand un de ces abonnements a spécifié un _texte de pop-up_, celui-ci est affiché par le browser lors de sa réception, **que l'application soit lancée OU NON**.

> **Des pop-ups d'alertes peuvent ainsi s'afficher**, même quand l'application n'est pas lancée, et informer l'utilisateur qui peut cliquer sur le pop-up. Si l'application n'était pas lancée, elle l'est. Sinon si l'écran montrait un contenu concerné par cette annonce de changement, la vue à l'écran se synchronise au nouveau contenu. 

Suivant ce paradigme, une application présente à son utilisateur trois concepts:
- des **notifications d'alerte** annonçant des évolutions de documents ou de collections de documents qui l'intéresse: l'arrivée de nouveaux échanges sur un _chat_ (un document), une évolution de nomenclature d'articles (liste des documents Sujet). Elles **annoncent** par des notifications courtes une évolution de certains documents, mais n'en donne q'un minimum d'information.
- des **notifications de documents synchronisés**: les documents correspondant sont systématiquement maintenus à jour dans l'application dans un état le plus proche techniquement possible de l'état des documents sur le serveur (du moins quand l'application est _au premier plan_).
- des **_rapports_**: ce sont des vues calculées à un instant donné et qui ne changent qu'à redemande du même rapport.

**Les documents synchronisés dans une application** le restent a minima tant que l'application est **au premier plan**:
- l'application **peut** décider de ne plus maintenir cette synchronisation quand elle passe **en arrière plan**: en pratique l'utilisateur ne voit d'une application en arrière plan que les _pop-ups_ de notification, c'est donc une économie de ressources que d'éviter de les synchroniser à cet instant.
- en repassant au premier plan, l'application **peut** demander aux serveurs de lui fournir les mises à jour des documents synchronisés depuis le dernier état mémorisé en session.

### Quand l'application n'est plus en exécution
Quand une application est en exécution elle peut rester _abonnée_ à des _alertes_.

Quand son exécution s'arrête, sauf décision explicite de l'utilisateur, certains de ces abonnements restent actifs, du moins un certain temps:
- les notifications correspondantes continueront à s'afficher en _pop-ups_, l'OS ou le browser de l'application s'en chargeant.
- l'utilisateur reste informé des _news_ auxquelles il était abonné.
- un clic sur un ces _pop-ups_ ouvre l'application ce qui lui permet plus ou moins directement de connaître en détail les documents ayant changé.

> Chaque application détermine pour chaque abonnement élémentaire auquel elle est abonné, si l'abonnement s'interrompt ou non quand l'application s'arrête.

# Modules génériques utilisables dans les logiciels serveurs
Ces modules forment une couche logicielle offrant un certain nombre de services de bases raccourcissant et sécurisant l'effort de développement.

## Gestion des _opérations_
**Une opération est initiée par la réception d'un requête HTTP**. La couche de base:
- identifie l'opération demandée en fonction de l'URL.
- fixe son _time_ une date-heure unique (pour le serveur) et croissante qui sera attribuée aux versions des documents créés / modifiés / supprimés par l'opération.
- récupère les paramètres d'entrée et les met à disposition de l'opération dans une structure.
- **gère l'opération comme une succession de phases:**
  - **phase 1:** vérification que les paramètres d'entrée sont bien formés, présents s'ils étaient obligatoires, etc. _Normalement_ l'application terminale s'en est assuré mais l'opération dans le serveur fait cette vérification pour éviter de tomber sur des exceptions dues à des valeurs qui n'auraient pas dues être trouvées en entrée. Cette phase s'effectue sans accès à la base de données.
  - **phase 2:** c'est une transaction au sens de la base de données. Elle lit, met à jour créé et supprime des documents et élabore un résultat. En cas d'exception détectée comme liée à une saturation technique de la base de données, cette phase est annulée et relancée (du moins un certain nombre de fois).
  - **phase 3:** c'est une phase de nettoyage qui peut effectuer quelques mises à jour dans la base de données, typiquement dans la table `FTP` pour la gestion des fichiers.
  - **phase 4:** la phase 2 a produit une liste de documents mis à jour. Cette phase identifie les abonnements associés et génèrent les _notifications_ aux applications abonnées.
- **retourne le résultat à l'application appelante,**
  - construit en phase 2,
  - ainsi que la liste des abonnements calculés en phase 4 auxquels l'application appelante était abonné. Ceci évite à l'appelant d'attendre une _notification_ (qui peut être un peu plus tardive) venant par le browser

## Gestion des documents
Un module gère une _mémoire cache_ des documents les plus récemment demandés et mis à jour. 

Ainsi la demande par une opération en phase 2 d'un document qui se trouve en _cache_,
- soit délivre le document s'il n'est pas exigé à être absolument dans sa version la plus récente mais tolère une _certaine ancienneté_,
- soit, et c'est le cas général, s'assure que c'est bien la dernière version: si c'est le cas seul un index a été lu, sinon le document est lu et conservé en _cache_.

Le module gère également une _mémoire cache de documents_ **pour chaque opération en cours**.
- les documents lus y sont stockés dans leur forme _objet compilé_ (et non le format sérialisé de stockage en base).
- les documents _zombifiés_ et créés y sont aussi stockés.

A la fin de l'opération le module gère les _abonnements_ impactés par les documents modifiés / créés / zombifiés / détruits, et les met à jour dans la base : ils seront utilisés en phase 4 de l'opération pour notifier les applications abonnées.

## Requête de collecte des documents sur un _abonnement élémentaire_
Quand une application terminale souhaite disposer des documents mis à jour pour un _abonnement_ donné, ce module détermine:
- si la mise à jour peut être _incrémentale_ ou si elle doit être _intégrale_,
- quels documents sont à retourner en fonction de la dernière version détenue par l'application terminale.

## Modules _providers_ d'accès à la base de données
Un module _provider_ présente un interface indépendant de la base de données gérée. Pour chacun des services de cet interface, il implémente l'accès effectif à la base qu'il gère:
- mise en forme / sérialisation des documents,
- cryptages / hachages éventuels des _data_ et propriétés indexées,
- gestion des transactions commit / rollback.

Pour un serveur donné, seul le _provider_ requis est importé.

> Typiquement en _test_ l'usage du provider _SQLIte_ simplifie le développement plutôt que ceux qui seront utilisés effectivement en production (_Postgresql_, _Firebase_...).

## Module _providers_ d'accès au _storage_
Les quelques services généraux d'accès sont développés pour chaque type de storage souhaité (AWS-S3, GCP, File-system ...).

## Principe de _déclaration_ statique
La structure des documents (leurs propriétés) se fait par _déclaration statique_ dans des modules de configuration.

## Modules utilitaires
Un module de cryptographie évite de gérer les subtilités du paramétrages des algorithmes.

> Vis à vis des applications terminales, les serveurs ne connaissent **QUE** les concepts de documents / abonnements et mises à jour incrémentales. Le reste de la logique applicative passe par l'usage des opérations.

# L'application _Safe_ 
Cette application a pour objet de gérer des _coffres forts_ pour des utilisateurs.

En lançant _Safe_ un utilisateur donne les éléments d'identification de son _coffre fort_ de manière à ce que les applications lancées ultérieurement sur cet appareil puissent y trouver les diverses données _sensibles_ de l'utilisateur dont ses _droits d'accès_ aux documents des applications.

L'application _Safe_ une fois initialisée sur un appareil est en charge:
- de gérer ses appareils _de confiance_.
- pour chaque application:
  - de stocker `ses droits` et les mettre à jour.
  - de stocker _divers objets_:
    - des objets transmis par un autre utilisateur disposant lui aussi d'un _safe_. 
    - des objets interprétés comme des _options de lancement_.
    - des objets interprétés par l'application comme des _préférences_.
  - de lancer l'application, le cas échéant selon _l'option de lancement_ que l'utilisateur a sélectionnée.

> Le lancement d'applications par le _Safe_ évite le risque de lancement d'une application _piratée_ et permet de choisir le cas échéant des _options_ au lancement ouvrant la session dans un contexte déjà préfixé.

> L'URL de lancement de _Safe_ est un élément de sécurité: le code de cette application est lisible dans le browser et il est possible de vérifier auprès de sites certificateurs que l'application est _fair_. Ceci évite d'avoir à le faire pour chaque application gérée par _Safe_, quoi que ce soit toujours possible. 

# Un utilisateur, ses appareils et ses applications

## Appareils _de confiance_ d'un utilisateur
Un utilisateur qui veut utiliser une application depuis un _device_ est placé devant deux cas de figure:
- **soit il juge l'appareil _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - il peut y laisser quelques informations cryptées et espérer raisonnablement les retrouver plus tard.
- **soit il n'a pas confiance dans cet appareil** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver des données.

Lancer une application depuis un appareil _de confiance_ a plusieurs autres avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant une petite base de données locale (cryptée) pour chaque application comme _cache_ de ses documents: ceux qui y figurent et à jour n'auront pas besoin d'être demandés au serveur de l'application.
- **disponibilité des droits** de l'utilisateur pour chaque application le dispensant de s'en souvenir ou de les copier / coller d'un support externe.
- **possibilité d'accéder à l'application en mode _avion_** sans accès au réseau en utilisant les documents et les droits en _cache_.

> Même _de confiance_ un appareil _peut_ être utilisé par d'autres que soi-même: même dans un cadre familial ou de couple, l'appareil n'est pas strictement _personnel_.

## Lancement d'une application
Depuis son application _Safe_ qui gère son _coffre fort_, un utilisateur peut lancer ses applications:
- **soit depuis un appareil _anonyme_:**
  - soit normalement en laissant _Safe_ obtenir son contenu depuis le serveur _Safe_.
  - soit en fournissant un _file / clé USB_ disposant de ce contenu (crypté).
  - Les applications opèrent en mode **_incognito_**.
- **soit depuis un appareil _de confiance_:**
  - **soit en mode _avion_:** le contenu du _Safe_ est obtenu depuis le cache local crypté du _Safe_. Les applications opèrent en mode **_avion_**.
  - **soit en mode _normal_:** le contenu du _Safe_ est obtenu depuis le serveur _Safe_ (mettant à jour un _cache_ local crypté). Les applications opèrent en mode **_synchronisé_**.

Au lancement d'une application une page d'accueil est présentée:
- elle peut être spécifique de l'option de lancement choisie;
- elle peut être _pré-initialisée_ selon cette même option, voire tout _objet_ stocké dans son _safe_ par l'utilisateur.

Au cours de l'exécution de l'application des droits peuvent être ajoutés / modifiés si l'application est en mode _synchronisé ou incognito_: ils sont répercutés sur le serveur _Safe_.

**En mode _synchronisé_** en fermant l'application, l'utilisateur peut choisir:
- de laisser ses _abonnements de news_ activés: des notifications apparaîtront, même quand l'application sera fermée et même si ce n'est plus le même utilisateur qui dispose de l'appareil. Les _notifications_ ne font qu'annoncer des changements sans en donner les détails et évitent de délivrer des informations confidentielles. 
- de fermer ses _abonnements de news_: aucune notification ne parviendra plus sur l'appareil relativement à cette application. L'utilisateur peut le prêter à quelqu'un d'autre sans risque ... mais lui-même ne recevra plus de notifications (il faut choisir).

**En mode _avion_**, il n'y a pas de réseau, pas _d'abonnements actifs_: les dossiers peuvent être consultés mais pas mis à jour. 
- la base de données locale est cryptée par la clé `K` du _Safe_ de l'utilisateur. Elle contient pour chaque sous-collection associée à un abonnement, les documents qui en faisaient partie à un instant t: ceux-ci ne sont pas du tout dernier état mais a minima dans l'état t où ils ont été accédés la dernière fois sur ce appareil.
- l'utilisateur peut saisir des textes ou formulaires purement locaux et stocker des fichiers (comme des photos prises en mode avion): toutes ces informations pourront être utilisées pour mettre à jour des documents quand le réseau sera à nouveau disponible en sécurité.

**En mode _incognito_** la fermeture de l'application ne laisse pas le choix: les _abonnements de news_ sont tous supprimés, aucune notification ne parviendra plus sur cet appareil résultant de l'usage précédent de l'application.

### Mode _veille_
Les applications ouvertes ont un mode _veille_ optionnel: s'il est activé, l'application se met en veille en cas de non utilisation pendant quelques minutes (fixés selon le degré de paranoïa de l'utilisateur). Pour sortir de la veille un code est nécessaire et au second échec l'application se ferme.

# Glossaire technique

### Strong Hash: `PBKDF`
- **SH(s1, s2, SEP)** (Strong Hash): le SH s'applique à un couple de textes `s1 s2`, typiquement un login / mot de passe, mais aussi aux _passphrase_ en une ou deux parties. Il a une longueur de 32 bytes et est unique pour chaque couple de textes `s1 s2`. Il est _strong_ parce qu'incassable par force brute dès lors que le couple de textes ne fait pas partie des _dictionnaires_ des codes fréquemment utilisés. `SEP` est un caractère de séparation / remplissage qui allonge le couple `s1 + SEP + s2` à une taille minimale.

### Clés asymétriques C / D : cryptage / décryptage
Un couple de clés `Ca / Da` asymétriques généré par A:
- `Ca` est une clé publique de **cryptage**: elle est utilisée par B pour crypter par une clé X un objet qui ne pourra être décrypté que par A.
- `Da` est la clé privée de **décryptage**: elle est utilisée par A pour décrypter par la clé X un objet qui a été crypté par B en utilisant `Ca`.
- la clé X est la même qu'elle ait obtenue depuis Ca/Db ou Cb/Da.

### Clés asymétriques S / V : signature / vérification
Un couple de clés `Sa / Va` asymétriques généré par A:
- `Va` est une clé publique de **vérification**: elle est utilisée par B pour _vérifier que la signature S d'un texte T_ a bien été générée par A en utilisant `Sa`.
- `Sa` est la clé privée de **signature**: elle est utilisée par A pour _générer la signature S d'un texte challenge T_.

-----------------------------------------------------------------------

# Décompter les consommations

Une application peut être gratuite pour certains de ses utilisateurs (voire tous) et payante pour d'autres selon l'usage qu'ils en font. 

Mais même dans le cas où l'application est toujours gratuite pour tous, il peut être cependant nécessaire / intéressant de décompter des unités d'œuvre d'utilisation afin le cas échéant de les limiter.

La mise en œuvre de **contrats** permet,
- d'effectuer des décomptes d'usage par des _utilisateurs_, 
- d'en gérer des limites,
- si souhaité d'établir des _factures / paiements / régularisations_.

## Contrats et utilisateurs, leurs consommations

Un _utilisateur_ est identifié par l'application et tout utilisateur est toujours rattaché **à un instant donné** à un _contrat_. Au cours de sa vie un _utilisateur_ peut être détaché de son contrat précédent et être rattaché à un autre.

Un _contrat_ est un document d'identifiant généré à sa création par l'application visant à suivre et gérer les consommations des utilisateurs qui lui sont rattachés: les consommations sont décomptées pour le mois _courant_ et le mois précédent (immuable par principe).

> Dans le cas général un _contrat_ concerne un _petit collectif d'utilisateurs_ une famille, une équipe, une petite association ... mais éventuellement un seul utilisateur.

Le décompte des consommations sur un contrat peut entraîner des contraintes applicatives, par exemple:
- rejet d'opération pour dépassement de certains seuils,
- ralentissement en cas de dépassement,
- blocage éventuel de certaines opérations pour certains utilisateurs.

## Unités d'œuvre des contrats (UC)

Chaque application déclare ses _unités d'œuvre_ selon deux catégories: les unités de _stock_ et les unités de _calcul / travail_.

Chaque unité a ses compteurs spécifiques: elles ne se moyennent ni se s'additionnent pas entre elles. Définir si c'est nécessaire des unités génériques englobantes.

Les unités sont décomptées,
- pour chaque _utilisateur_ d'un contrat.
- globalement pour le _contrat_ (somme des valeurs pour chacun de ses utilisateurs).

### Unités de _stock / volume_
Leur existence a un coût par le seul effet du temps qui passe. Par exemple:
- S1: nombre de commandes en cours.
- S2: volume en Mo de fichiers stockés.

> Même sans utiliser l'application, le stockage a un coût dépendant du temps.

Le contrat conserve pour chaque unité:
- le valeur _courante_, c'est à dire la dernière valeur attribuée,
- la moyenne sur le mois en cours,
- la moyenne sur le mois précédent.

La moyenne est calculée au prorata du temps passé (à la milliseconde) à chaque valeur.

### Unités de _calcul / travail_
Elles _coûtent_ effectivement lorsqu'elles sont sollicitées. Par exemple:
- C3: **nombre** de lecture de documents D1 et D2.
- C4: **volume en Mo** des fichiers de type F1 téléchargés.

> Tant qu'une application n'est pas utilisée, ces coûts sont nuls.

Le contrat conserve pour chaque unité:
- la valeur _courante_.
- la somme des consommations pour le mois en cours,
- la somme des consommations pour le mois précédent,

Le nombre _courant_ est une **valeur moyenne journalière récente** lissée sur au moins 20 jours:
- après le 20 du mois c'est le total du mois en cours ramené à 24h.
- avant le 20 du mois, par exemple le 5, c'est la somme,
  - de 5/20 fois le total du mois en cours ramené à 24h.
  - de 15/20 fois le total du mois précédent ramené à 24h,

#### _Quotas_
Un _quota_ est codifié par deux chiffres `en`, sa valeur étant `(10**e) * n`.

      3 => 3
     14 => 40
     25 => 500
     37 => 7000 - Kilo
     67 => 7,000,000 - Mega
     99 => 9,000,000,000 - Giga
    125 => 5,000,000,000,000 - Tera
    155 => 5,000,000,000,000,000 - Peta

Un _quota_ permet d'exprimer sur un entier très court (un byte),
- soit un _seuil maximum_,
- soit un ordre de grandeur _forfaitaire_ d'une quantité. Par exemple la quantité 5342 correspond au _quota_ 36 (6000, le plus petit supérieur à sa valeur).

## Quotas applicables à un contrat

Pour chaque UC déclarée, le contrat définit un _quota_ à rapprocher de la valeur courante correspondante. Par exemple:
- S1-nombre de commandes en cours :  un quota de `23` correspond à 300 documents.
- C4-volume en Mo des fichiers de type F1 téléchargés : un quota de `65` correspond à 5000000, soit 5Mo par jour (lissé sur 20 jours).

**Pour les unités de _stock_, le _quota_ est un maximum qui ne peut pas être dépassé**, l'opération demanderesse tombe en exception.
- _sauf_ si l'opération est marquée _privilégiée_,
- _sauf_ si le maximum est certes dépassé mais en baisse.
- le **niveau d'alerte** est le pourcentage de dépassement (au plus 999%) au delà de 80% (0 en deçà).

**Pour les unités de _calcul_ le dépassement du quota peut provoquer un ralentissement de l'opération**, un temps d'attente fonction du taux de dépassement afin de _dissuader l'usage d'opérations coûteuses_, voire pour les grands dépassements de faire tomber l'opération en exception (si elle n'est pas _privilégiée_).

### Pour chaque _utilisateur_
Des _quotas_ par UC sont également fixés par _utilisateur_: 
- l'action de blocage / ralentissement et le niveau d'alerte sont les plus restrictifs des deux _utilisateur / contrat_.

## Décompte continu des unités sur un contrat: arrêté mensuel
Le décompte est établi en nombre pour toutes les unités effectivement consommées dans le mois courant et le mois précédent: il est disponible,
- par utilisateur,
- par sommation, globalement pour le contrat.
- pour les _unités de stock_ le nombre n'est pas entier puisqu'il s'agit d'une moyenne au temps passé effectivement (en milliseconde).

> A la fin de chaque mois un arrêté mensuel est calculé et mémorisé comme _mois précédent_ désormais invariant: l'application peut archiver les factures sur la profondeur d'historique de son choix avec un format adapté au traitement statistique.

## Historique des _quotas_ sur N mois
On conserve après l'arrêté mensuel, pour chaque compteur, sa valeur **forfaitisé** ramené au quota immédiatement supérieur.

### Tarif: prix unitaire pour chaque UC
Un _tarif_ donne un _prix unitaire_ pour chaque UC.

L'application du tarif à tous les compteurs courants _forfaitisés_ donne un _montant monétaire_:
- _provisoire_ pour le mois courant,
- _définitif_ pour le mois précédent.

> La _monnaie_ est virtuelle: c'est à l'application de prciser le _cors_ de sa monnaie en monnaies réelles (euro, dollar ...) pour convertir en monnaie virtuelle de l'application les versements reçus (en monnaie réelle).

> Quand un _contrat_ est utilisé d'une manière assez stable dans le temps, ces montants ont des chances d'être identiques d'un mois sur l'autre.

## Débits et crédits, solde
La _facture mensuelle d'un contrat_ comporte une liste de _débits / crédits_: chacun a,
- un _code_ qui indique sa nature: par exemple _paiement reçu_,
- une _référence_ ou _commentaire_ permettant d'en savoir plus dans l'application.
- un montant positif ou négatif.

La première ligne est un _débit_ correspondant à la consommation du _contrat_ dans le mois.

### Ajustement global de l'application
Une fois la consommation ainsi calculée, l'application peut introduire une ligne finale d'ajustement, au débit ou au crédit:
- remise sur volume,
- remise / pénalité selon le type de contrat,
- le cas échéant **la remise peut être égale au montant de la consommation** ce qui revient à de la gratuité. 

La _balance_ en fin de mois correspond à la balance en fin du mois précédent, plus les crédits, moins les débits.

En cours de mois une _balance provisoire_ est calculable:
- depuis la balance au mois précédent,
- la consommation en cours _forfaitisée_,
- la ligne d'ajustement spécifique de l'application.
- la balance des débits / crédits du mois.

### Politique vis à vis d'une balance négative
L'application fixe sa politique vis à vis d'un _contrat_ présentant une balance _provisoire_ négative. Par exemple:
- tolérance si la balance en début de mois était positive,
- tolérance pour les opérations _privilégiées_,
- alerte selon le nombre de jours estimés avec _balance positive_: simple calcul du nombre de jours pendant lequel la balance restera positive si la consommation se maintient au même niveau que la moyenne du mois courant et du mois précédent.
- rejet de l'opération.

## Authentification d'utilisateurs via leur contrat

Tout utilisateur rattaché à un contrat _peut_ y être associé à une ou plusieurs **passphrases**: 
- une _passphrase_ donnée ne peut être associée qu'à un seul contrat à un instant donné et à un seul _utilisateur_ dans le contrat.
- dans son contrat, une map associe chaque utilisateur à son _objet d'authentification_.

#### Exemple 1: authentification par clés de _signature / vérification_
- un _objet d'authentification_ d'un utilisateur `id` est formé d'un couple `[hkp, kv]`
  - `kv` est la _clé de vérification_ de signature.
  - `hkp` est le hash court de la _clé de signature_ et est listé comme _passphrase du contrat_.
- une opération dispose dans ses _jetons d'accès_ d'un couple `[hkp, sign]`. La méthode d'authentification,
  - recherche un _contrat_ ayant la passphrase `hhp`,
  - dans ce contrat récupère `id kv` de l'utilisateur et vérifie la signature reçue par la `kv` enregistrée.

#### Exemple 2: authentification par _phrase secrète_ de l'utilisateur
- un _objet d'authentification_ d'un utilisateur `id` est formé d'un couple `[lhps, shps]`
  - `lhps` est un hash _long et fort_ comme le PBKDF d'une phrase secrète connue du seul utilisateur.
  - `shps` est un hash _court_ de la phrase secrète et est listé comme _passphrase du contrat_.
- une opération dispose dans ses _jetons d'accès_d'un couple `[lhps, shps]`. La méthode d'authentification,
  - recherche un _contrat_ ayant la passphrase `shps`,
  - dans ce contrat récupère `id lhps` de l'utilisateur et vérifie que le `lhps` reçu correspond au `lhps` enregistré.

> Dans ces deux exemples une seule action localise un contrat, identifie et authentifie un _utilisateur_.

> Un contrat est aussi accessible par son identifiant et contient tous ses utilisateurs enregistrés par leur identifiant.

### Suivi des consommations dans les opérations

Si l'application enregistre les _consommations_, sauf exception toute opération du serveur commence par récupérer un _contrat par défaut de l'opération_ à qui imputer ses consommations d'UC. 

Au cours de l'opération, d'autres _contrats_ peuvent être cités à qui imputer certaines consommations spécifiques selon la logique de l'application. Par exemple le stockage et l'utilisation de documents partagés par un **groupe** d'utilisateurs peut être imputé au _contrat du groupe_ plutôt qu'aux _contrats_ des utilisateurs eux-mêmes.

### Début et fin d'opération
Le début d'une opération va, en général, 
- lire le _contrat_ depuis la base et authentifier l'utilisateur demandeur de l'opération,
- en cas de basculement sur le mois suivant, calculer la facturation et repositionner les compteurs.

En cours d'opération, les compteurs du mois courant sont mis à jour (si nécessaire), la _balance provisoire_ peut être réévaluée ainsi que le nombre de jours en _balance positive_.

La fin d'une opération va, en général, enregistrer en base la mise à jour du _contrat_.

Un _contrat_ a un mois de dernier calcul d'arrêté comptable: une tâche périodique à partir du N d'un mois effectue a minima une opération sur chaque contrat dont le dernier mois d'arrêté n'est pas le mois courant afin d'éviter de perdre des facturations sur les _contrats_ peu utilisés.

Un traitement périodique peut aussi collecter un historique sous forme de fichier CSV pour analyse externe.

## Détail du document `Contrat`
Propriétés:
- `statCode` (indexé) : code statistique: pour déclenchement sélectif d'actions / reports.
- `lcm` (indexé) : dernier mois calculé / facturé: pour déclencher le calcul de la facture.
- `balance` : balance temporaire en cours de mois.
- `negBal` (indexé) : niveau d'alerte de balance négative.
- `nbdPos` : nombre de jours estimés restant avec balance positive si la consommation continue de suivre la tendance de M et M-1.
- `minLrdu` (indexé) : `lrdu` le plus ancien des _users_.
- `time` : date-heure du dernier calcul.
- `users` : map avec une entrée par `userId`: `{ ldr, quotas, counts }`
  - `lrdu` : dernier jour de référence, utilisation comme entrée d'un contrat. 
  - `quotas` : array de N+1 éléments pour un historique de N mois.
    - le premier élément donne les quotas courants (_maximum_),
    - les éléments suivants donnent les consommations _forfaitaires_ constatées sur le mois.
    - chaque éléments a C compteurs (courts).
  - `counts` : Un array de C éléments chacun étant `[ vc, cm, pm ]` 
    valeur courante, mois courant, mois précédent.
- `total` : `{ quotas, counts }`. somme des compteurs des _users_.
- `invoices` : deux éléments, mois en cours, mois précédent `[dbcr, balance]`:
  - `dbcr` : liste de _débits / crédits_: chacun a `[ code, ref, m ]`:
    - `code` : indique sa nature: par exemple _paiement reçu_,
    - `ref` : une _référence_ ou _commentaire_ permettant d'en savoir plus dans l'application.
    - `m` : un montant positif ou négatif.
  - `balance` : balance au début du mois
- `authKeys` (indexé) : liste des clés d'accès externes aux _users_.
- `authUsers` : map avec une entrée par userId. 
  - _val_ : objet contenant les informations d'authentification. 

**Remarques**
- La liste `authKeys` est calculable par l'application depuis la map `authUsers`.
- les _compteurs_ sont des _number_, les _compteurs forfaitaires_ sont des bytes.
- L'application déclare une _configuration_:
  - le nombre de mois historisés.
  - une liste avec un terme [code, stock] par UC:
    - _code_: ce code court sert aux traces / éditions.
    - _stock_: true si c'est une unité de _stock_ (sinon c'est une unité de calcul).

### _Users_ virtuels
Il est possible de décompter des consommations et définir des quotas maximum pour des _usages_ identifiés. Par exemple:
- des _groupes_ sont rattachés à un contrat.
- chaque _groupe_ peut avoir des quotas pour certains compteurs, voire disposer d'un calcul de consommation propre.

Certains users virtuels peuvent être gérés comme des entrées de _comptabilité analytique_: chaque consommation _réelle_ est _dédoublée_ dans un _user_ ET dans un _user virtuel analytique_.

On peut déclarer ainsi un _user_ identifié par l'application, mais ce _user_ étant virtuel n'a pas d'entrée dans `authUsers`.
- l'application donne dans sa configuration une liste des identifiants des users ayant un comportement _analytique_.
- la ligne **total** du contrat ne totalise pas les lignes des _users analytiques_ mais les lignes correspondant à des consommations réelles.

### Contrats / _users_ disparus
L'application peut considérer comme _disparu_ des _users_ qui n'ont pas _utilisé_ leur contrat depuis un certain nombre de jours.

Il est possible de filtrer par une tâche tous les contrats ayant un _user_ présumé disparu et le cas échéant de gérer:
- leur suppression dans le contrat,
- voire la suppression du contrat lui-même si aucun _user_ n'est plus vivant.

--------------

# Contributions diverses en attente

### Lecture d'un fichier
Elle peut s'effectuer de deux manières:
- en retournant le contenu binaire du fichier dans la couche applicative,
- en retournant une URL d'accès sécurisé valable un certain temps, typiquement à transmettre à une application externe.

### Cohérence _forte_ / _faible_ entre collections de documents
Toutes les collections de documents et documents dont la synchronisation a été demandée par la même requête sont en cohérence _forte_, calculés exactement sur le même état de la base noté par la date-heure t de l'opération.

En revanche, quand des documents sont synchronisés par deux requêtes (r1 à t1) et (r2 à t2) différentes, leur cohérence est _faible_, certains étant connus dans l'état t1 de la bse et d'autres dans l'état t2 de la base: toutefois pour ces documents rien ne dit que les état à t1 et t2 sont différents ou identiques.


> On pourrait certes grouper dans une même requête toutes les _synchronisations de tous les abonnements_ mais la requête put être longueet le volume (trop) important. Il y a applicativement un compromis à choisir entre _force de la cohérence entre documents_ et lourdeur technique.

### Authentification _double_
(questions, pertinence)

Faut-il prévoir d'obliger à une authentification depuis un appareil #1 exigeant une confirmation sur un appareil #2 (favori ou non).
- que se passe-t-il quand l'appareil #1 n'est déclaré _favori_ ?
- et si l'utilisateur N'A PAS d'appareil #2 (au moins sous la main) ?
- si #2 n'est PAS favori, pour valider le login de #1 il lui faut un couple (s1, s2) qu'il vient a priori déjà de donner sur #1 ?

### Attacher des _droits d'accès_ aux _abonnements_
Il peut être requis par une application qu'un _droit d'accès_ soit fourni pour chaque abonnement élémentaire.
- ceci évite qu'une session soit _notifiée_ de mises à jour de documents auxquels elle n'a pas droit d'accès.
- la _synchronisation effective_ (récupérer le contenu des documents) est sauf exception soumise à des _droits d'accès_.

## OBSOLÈTE ??? : _activités_ définies dans une application

Dans une application terminale une _activité_ désigne un ensemble de tâches cohérentes qu'un utilisateur peut effectuer. Par exemple dans l'exemple _circuitscourts_:
- l'activité _commande d'un consommateur_ où un consommateur peut déclarer les quantités qu'il souhaite recevoir pour les livraisons en cours.
- l'activité _contrôle et réception des livraisons_ pour les animateurs d'un point-de-livraison visant à vérifier les commandes, réceptionner les camions et noter les quantités reçues.
- l'activité _préparation d'une livraison_ pour un groupement de producteurs, gérant le calendrier de la livraison, rassemblant les cartons préparés par les producteurs pour effectuer une tournée auprès des points-de-livraison associés.

Une _activité_ décrit à la fois:
- sur quelles données elle doit opérer, quels documents doivent être rendus visibles aux utilisateurs ayant opté pour cette activité.
- quels processus _suites d'actions élémentaires concourant à un objectif plus global_, un utilisateur peut engager dans le cadre de cette activité.
- quels _credentials_ un utilisateur doit présenter pour avoir le droit de voir les données et d'exécuter les processus.

### Une session d'une application pour un utilisateur peut avoir plusieurs activités ouvertes

#### L'activité d'amorce
Il est toujours possible de lancer une application sans avoir d'activité favorite enregistrée.
La session doit présenter un minimum d'information à l'utilisateur:
- pour qu'il puisse décider quelle activité il va choisir d'exercer.
- saisir les _credentials_ requis.

Pour ce faire l'utilisateur va souvent avoir besoin de voir quelques données: elles sont définies dans une activité bien identifiée _amorce_ qui peut présenter des informations non soumises à un _credential_.

Le _desktop de l'application_ est affiché pour présenter ces choix.

Ultérieurement plusieurs activités peuvent être ouvertes:
- plusieurs du même type: assurer _le contrôle et réception des livraisons_ pour deux points-de-livraisons.
- plusieurs de types différents: assurer _la commande d'un consommateur_ (pour lui-même) et contribuer à _la préparation d'une livraison_ à titre d'aide d'un groupement de producteurs ami.

### Un _type d'activité_ a des paramètres et est associé à un ou plusieurs _types de credentials_
Par exemple l'activité _commande d'un consommateur_ a pour paramètres,
- `gc` : le code d'un point-de-livraison.
- `co` : le code d'un consommateur récupérant ses produits auprès de ce point.
- `initials` d'un utilisateur,
- `pwd` : mot de passe déclaré pour le couple `gc co` pour les `initiales` fournies.

Ce type d'activité est associé à un ou plusieurs types de _credential_, ici par exemple `CREDCO` qui peut se construire à partir des paramètres de l'activité:
- `gc co` : sont les identifiants d'un _credential_ `CREDCO` dont les autres propriétés sont `initals` et `pwd`.

Le ou les _types de credentials_ déclarés associés à une activité, doivent avoir tous leurs paramètres dans les paramètres du _type d'activité_: un _credential_ peut ainsi être généré depuis ceux-ci et sera délivré aux opérations qui le requièrent.

### Choisir et exercer une activité dans une session d'une application terminale
Le _desktop_ de l'application présente à l'utilisateur les _types d'activité_ qu'il peut choisir. 
- Ce peut être une liste courte,
- Ce peut être pour une application complexe une liste longue, avec une possibilité de sélection par mot clé et / ou une présentation arborescente.
- Le paramétrage du _desktop_ déclare comment il apparaît et comment l'utilisateur peut sélectionner une activité.

Quand l'utilisateur a choisi un type d'activité il doit saisir tous les paramètres de l'activité: par exemple un code `gc` et un code `co`, des `initials` et un `pwd`.

Il a alors une _activité ouverte_, ce qui apparaît sur son _desktop_.

Il peut en ouvrir d'autres, du même type ou non, chacune figurée par exemple par un onglet et / ou une icône et / ou un libellé.

Depuis le _desktop_ l'utilisateur peut _basculer d'une activité à une autre_ par exemple en cliquant sur un onglet ou une icône, ou si l'application le permet en voir plus d'une affichée (une en haut, une en bas).

### Enregistrement d'une _session favorite_ dans son _profil_
Si l'utilisateur a un profil enregistré, à n'importe quel moment de sa session de travail en cours il peut:
- sélectionner en les cochant certaines de ses activités en cours,
- enregistrer cette sélection en lui donnant un libelle clair pour lui, comme _commandes Bob à JP_.
