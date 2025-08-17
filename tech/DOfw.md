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

# Organisations: les services sont _multi-tenant_
Un service comme `randos`, peut à la manière de Discord, proposer d'héberger les applications d'associations de randonneurs distinctes: chaque organisation / _tenant_ dispose de _son_ espace de données propre complètement étanche à celui des autres.

Un service `boutiques` propose de gérer plusieurs boutiques, pas une seule, mais de manière à ce que les données de chacune soient totalement isolées de celle des autres.

Les données d'un service d'un prestataire sont stockées dans deux _mémoires persistantes_ (voir plus loin):
- **UNE base de données** logiquement strictement partitionnée par organisation, sans aucun lien ou référence à des données / documents d'une organisation par une autre.
- **UN _storage_ de fichiers**, comme un directory de fichiers classiques, avec une **racine** par organisation.

### Pour un service donné, UNE organisation donnée n'est hébergée que par UN prestataire
Pour un service `randos` proposés par les prestataires **Rouge** et **Bleu**, une organisation donnée _val-de-bièvre_ est _hébergée_ chez **Rouge** ou chez **Bleu** mais pas dans les deux.

UNE base centrale unique pour `randos` indique pour chaque organisation le prestataire qui l'héberge (l'URL d'appel du service).

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
- l'OS de l'appareil ou le browser dans lequel elle est enregistrée, , selon que l'utilisateur l'autorise ou non, afficher en _popup_ la notification ce qui alerte l'utilisateur,
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
Elle gère les documents et les fils de documents selon un mode _transactionnel_ (ACID).

Elle gère aussi les _abonnements_ des applications terminales aux _fils de documents_ qui les intéressent:
- chaque application sur un appareil a un _token_ qui l'identifie de manière unique. Ce répertoire contient les _abonnements_ en cours des applications.
- Le répertoire détecte les applications n'ayant pas été lancées depuis un certain temps.
- pour chaque application le répertoire conserve la liste des abonnements en cours aux _fils de documents_.

Quand un document évolue, le répertoire retrouve toutes les applications abonnées à un _fil_ auquel le document est attachés et effectue une publication de notifications vers elles.

> Chaque application terminale est en conséquence susceptible de s'abonner éventuellement auprès de plus d'un prestataire si toutes les organisations de son domaine d'intérêt ne sont pas toutes gérées par le même prestataire.

### Le _Storage_ de fichiers 
Il stocke des _fichiers_ identifiés par leur _path_: la présence de caractères `/` dans un path définit une sorte d'arborescence. Le _contenu_ de chaque fichier est une suite d'octets opaque.

En lui-même il n'est pas soumis à un protocole transactionnel (ACID): sa sécurité transactionnelle est déportée sur la base de données avec un protocole simple à deux phases. Le couple base de données / storage permet de garantir qu'un fichier existe ou non, de manière atomique.

Le Storage permet de disposer d'un volume pratiquement 10 fois plus importants à coût identique par rapport à la base de données: de nombreuses applications ont des données historiques / mortes ou d'évolutions sporadiques qui s’accommodent bien d'un support sur Storage.

# Le répertoire des _organisations par application_

Toute _application_ terminale détient, en tant que ressource statique, la liste des _prestataires_ fournissant les services centraux, avec pour chacun:
- leur _code_,
- leur _URL d'accès_.
- la liste des _organisations_ hébergées.

L'ajout / retrait d'un prestataire et / ou d'une organisations  demande de générer une nouvelle version de l'application concernée. Toutefois une organisation pas encore _statiquement répertoriée_ peut être référencée par un utilisateur en indiquant le code du service qui l'héberge.

Une session d'une application terminale peut concerner plusieurs organisations, à l'instar du randonneur faisant partie de plusieurs associations selon l'endroit où il randonne. Pour chaque organisation concernée elle obtient de ce répertoire le prestataire gestionnaire et son URL d'accès.

Chaque service peut ensuite gérer **dans _sa_ base de données**,
- un document unique concernant toutes les organisations,
- un document relatif à chaque organisation.

Ces documents peuvent comporter:
- un **statut récapitulatif** : ouverture, restriction en lecture seule (archive), fermeture jusqu'à nouvel ordre.
- une **courte liste des dernières _news_ ayant modifié ce statut** données par l'administrateur.

Les applications terminales peuvent s'abonner aux modifications de ces documents et ainsi afficher pour la ou les organisations référencées dans leur session,
- son statut d'accessibilité, globalement et pour chaque organisation spécifiquement,
- les _news_ récentes ayant modifié ce statut.

# Documents, fichiers et _fils_ traçant leurs évolutions

## Document
Selon le standard JSON:
- deux **structures** de données sont utilisables:
  - des _listes_ ordonnées,
  - des _maps_, associations _clé (string) / valeur_.
  - chaque terme d'une structure peut être une structure ou une donnée primitive.
- les données primitives sont des _string, number, boolean_.

Un document est un agrégat de données structurées en JSON dont la racine est une map dont chaque nom / clé donne la valeur d'une propriété qui elle-même peut être une structure ou une donnée élémentaire.

Des _fichiers_ peuvent être attachés à un document
- chaque descriptif d'un fichier est une _map_ de quelques propriétés (nom, type, taille ...).
- les descriptifs sont inscrits dans une map `_files_` de la racine du document.
- le contenu effectif des fichiers sont des suites de bytes stockés à part dans un _storage_.

> Un document peut en conséquence être volumineux.

**Il y a plusieurs _types_ de document**, chacun correspondant à une structure dont la racine est une map de _propriétés_.

**Parmi ces propriétés une liste ordonnée de propriétés _string_ immuables constitue l'identifiant fonctionnel du document** (clé primaire en SQL, path en NOSQL). 

Exemple du document `CART` du _use-case circuit court_:
- un _carton_ est un ensemble de produits emballés ensemble par un producteur `pr` d'un groupement `gp` gérant un camion à destination de points de livraison `gc` pour une livraison donnée `livr`.
- `gp pr livr gc`, forment un quadruplet identifiant exactement un carton, donnant d'ailleurs de plus une information sur qui l'a constitué et à qui il est destiné.

On peut définir des **regroupements** de propriétés identifiantes dont une valeur détermine une collection de documents:
- le regroupement #1 `gp.livr`: en fixant cette valeur un point de livraison peut obtenir la liste des cartons à décharger du camion expédié par le groupement pour cette livraison, tous producteurs confondus.
- le regroupement #2 `gp.pr`: en fixant cette valeur un producteur peut obtenir la liste de tous les cartons qu'il doit composer pour toutes les livraisons en cours et tous les points de livraison.

**Parmi les propriétés certaines (de type _string_ ou _number_) sont _indexables_**.
- soit pour être utilisées comme identifiants secondaires, mais pas immuables, du document,
- soit pour filtrer la collection de ces documents selon des seuils de valeurs.

La propriété `version` du document est un numéro d'ordre de mise à jour: la numérotation est _chronologique_ mais pas _continue_.

La propriété `zombi` contient le jour de suppression _logique_ quand la document n'a été encore purgé physiquement.

### Stockage d'un document d'un type donné
Le document est stocké dans une table (SQL) ou une collection (NOSQL) spécifique du type de document.

**L'ensemble des propriétés** est sérialisé dans un champ dénommé `_data_`: ce contenu est désérialisable dans les applications terminales et les serveurs.

En base de données, les propriétés **visibles de la base de données** sont:
- `id` : clé primaire ou path.
- `version`.
- `zombi`.
- `_data_`.
- `z1 z2 ...` : les _regroupements_ de propriétés identifiantes (s'il y en a).
- `p1 p2 ...` : les _propriétés_ indexables (s'il y en a).

Le contenu structuré complexe du document `_data_` est crypté et en conséquence _opaque_ pour la base de données (et crypté pour la plupart des types de documents).
- les propriétés identifiantes _peuvent_ être remplacées par leur _hash_ si on ne veut pas que leurs valeurs soient lisibles dans la base. Les propriétés `pi` quand elles sont utilisées par test d'égalité peuvent être _hachées_ mais pas quand elles interviennent dans des filtres _d'ordre_ (les algorithmes de _hash_ ne préservent pas les relations d'ordres de leurs sources).

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

La propriété `_files_` (dans `_data_`) du document est une _map_ avec une entrée `fid` par fichier et pour valeur le descriptif du fichier.

> Selon la logique de l'application, la propriété `name` **est ou non unique dans son document**. Si elle est unique, le stockage d'un fichier d'un nom donné supprime d'office le fichier portant antérieurement ce nom. Si la propriété `name` n'est contrainte à être unique, plusieurs fichiers porteront le même nom dans un document (avec des propriétés `time` différentes) vus comme autant de _révisions_ pour un nom donné.

Le contenu du fichier est stocké sous un _path_ dans l'espace de stockage `folderId/fid`:
- `fid` est suffisant pour garantir l'unicité du contenu.
- `folderId` définit un _folder_ de rangement et a une structure `a/b/c ...` dont le seul intérêt est de pouvoir purger en une seule commande tous les fichiers sous une partie de ce path, par exemple les fichiers dont le path commence par `a/b`.
- les termes qui définissent le `folderId` sont parmi ceux apparaissant dans l'id du document:
  - `gp pr livr gc` dans l'exemple ci-avant, l'id du document,
  - `gp livr` le groupement d'id #2 défini pour le rattachement au fil `CMDGP`.

### Protocole de stockage / suppression d'un ou plusieurs fichiers
Une ou plusieurs opérations de **preload** chargent le contenu du fichier dans le storage sous le path `folderId/fid`, `fid` étant généré à cet instant.

Avant le stockage physique la ou les opérations de _preload_ notent dans la table `todelete` le couple (`folderId/id`, `date du jour`).

Une opération de validation enregistre ensuite dans le ou les documents concernés les nouveaux fichiers `fid` et leurs descriptifs. Cette opération s'accompagne éventuellement d'une liste de `fid` à détruire dans ces mêmes documents.
- pour chaque document la propriété _files_ est mise à jour.
- s'il y a des fichiers à détruire,
  - leurs entrées sont enlevés des propriétés _files_ de leurs documents.
  - leur couple (`folderId/id`, `date du jour`) est inséré dans la table `todelete`.
- la transaction est _validée par un commit_ de la base de données, les fichiers nouveaux _existent_ les fichiers supprimés n'existent plus (logiquement).

Après cette étape transactionnelle, une étape terminale prend place:
- les fichiers à supprimer sont effectivement purgés de leur répertoire de storage.
- les références des fichiers créés et de ceux supprimés sont purgées de la table `todelete`, cette seconde phase de transaction est validée par commit.

Les mises à jour comme les suppressions sont donc en deux phases et il se peut que suite à un incident une phase s'exécute et pas la seconde.

Un traitement périodique de nettoyage liste les fichiers inscrits dans `todelete` depuis plus d'un jour:
- ils sont purgés de l'espace de storage,
- ils sont purgés de la table `todelete`.

> Moyennant le respect de ce protocole simple, la gestion des fichiers dans un document bénéficie de la même sécurité transactionnelle que les autres propriétés du document.

## _Fils_ de documents
Un _fil de document_ est défini pour que des documents puissent s'y rattacher, sachant qu'un document peut,
- n'être rattaché à aucun fil,
- être rattaché à plusieurs fils.

Créer et maintenir un _fil_ est le moyen retenu pour **tracer** les évolutions des documents qui lui sont attachés: une application terminale (voir un traitement d'un serveur) peut ainsi être informé / notifié qu'au moins un des documents d'un fil a changé ou a été ajouté ou supprimé.

### Type de  _fil_
Le _type_ d'un fil définit son objectif: tracer les évolutions d'un certain nombre de documents et pour chacun selon quel filtrage sur les valeurs de ses propriétés identifiantes. 
- l'identifiant d'un fil est une suite de propriétés telle que les documents qui y seront rattachés les auront toutes dans leur propre suite de propriétés identifiantes.

Dans le _use-case circuit court_ par exemple `CMDGP` sert à rattacher tous les documents utiles à une livraison `livr` gérée par un groupement `gp`.
- l'identifiant d'un fil de type `CMDGP` est `gp.livr`.
- les types de documents rattachés à ce fil sont `CHD, BCG, CART`.
  - `CHD` (le chat ouvert pour une livraison donnée) a pour identifiant `gp livr`: il n'y aura au plus qu'un document `CHD` rattaché au fil.
  - `BCG` (un bon de commande d'un point de livraison) a pour identifiant `gp livr gc`: il y aura donc une collection de documents `BCG` dans le fil, tous ceux ayant pour regroupement indexé de propriétés `gp.livr` (soit au plus un par point-de-livraison `gc`).
  - `CART` à pour identifiant `gp pr livr gc`: il y aura donc une collection de documents `CART` dans le fil, tous ceux ayant pour regroupement indexé de propriétés `gp.livr`. Un carton est créé par un producteur qui y met tous les produits d'une livraison destiné à un même point-de-livraison.

> Connaissant le type et l'identifiant d'un fil, par exemple `CMDGP/gp.livr` on peut _tirer_ toute une collection de documents, le cas échéant nombreuse, `CHD`,  `BCG`, `CART` rattachés au même fil, en l'occurrence ceux concernant la livraison d'un camion organisé par un groupement de producteur pour une livraison donnée à plusieurs point-de-livraison (une _tournée_).

Un _type de fil_ définit de facto un critère de sélection s'appliquant à un ensemble de documents ayant pour identifiant ou regroupement d'identifiants une valeur donnée.

**Un fil donné, une instance de son type pour un identifiant donné, est _stocké_ en base de données** dans une table / collection portant le nom du type de fil, par exemple `CMDGP`. Ces tables / collections ont toutes le même schéma.

**Propriétés indexées:**
- `id` : concaténation des ids de sa clé primaire / path: `gp.livr`. Les valeurs individuelles des éléments de la clé ne sont pas citées.
- `version` : numéro séquentiel d'ordre de mise à jour du document le plus récent attaché au fil (ou détruit).

**Propriété opaque _data_ d'un fil:**
- `versions` est une map avec une entrée pour chaque type de document donnant le dernier numéro de version, soit _du_ document si c'est un singleton, soit du document de la collection _le plus récemment mis à jour_.
- `cleandate` : c'est la date du dernier nettoyage des suppressions (voir plus avant).

### Utilisation des _fils_
Chaque fil est une **trace** de l'évolution la plus récente des documents qui lui sont attachés: 
- le fait que la version d'un fil s'incrémente à chaque mise à jour d'un de ses documents fait du fil un événement _notifiable_.
- dans cette _notification_, une application terminale peut retrouver pour chaque type de document (UN document si c'est un singleton dans le fil, sinon une collection des documents) si ce document ou cette sous-collection a évolué depuis la version qu'elle détenait.

Une application terminale qui a gardé en mémoire la dernière image d'un fil qui lui a été transmise, peut à réception d'un nouvel état de ce fil, demander à un serveur la liste des documents de version postérieure à celle qu'elle détenait et en effectuer la mise à jour dans sa mémoire. Cette mise à jour est :
- _optimale_: elle n'est demandée QUE si un des documents d'un type qui intéresse l'application a changé. De plus le filtrage s'effectuant sur la propriété indexée `version`, seul l'index est sollicité (ce qui pour certaines bases NOSQL est gratuit) la _lecture effective_ n'étant pas faite pour les documents non modifiés.
- _incrémentale_: seuls les documents ayant changé depuis la version connue de l'application terminale sont lus et transmis.

### Traitements dans un serveur: attachement / mise à jour d'un document dans un fil
- récupération des `version` `vi` de tous les fils dans lequel le document est à insérer / mettre à jour.
- la version du document `v` est le maximum des `vi` + 1.
- dans chacun de ces fils:
  - la `version` est mise à `v`.
  - dans `_data_` la `version` pour le type du document est mise à `v`.

### Traitement des _suppressions_
Pour que la mise à jour soit incrémentale dans une sous-collection d'un type de documents, un document ne peut pas être simplement _purgé_: en demandant la liste des documents ayant changé il n'apparaîtrait pas, serait considéré comme inchangé et sa suppression non détectée.

Le document _supprimé_ va être traité comme une mise jour particulière:
- sa `version` est mise jour (comme pour une mise jour normale).
- ses propriétés indexables sont mises à null.
- sa propriété `zombi` donne le jour de suppression.
- son _data_ est mis à null.

Le document est en état _zombi_, supprimé logiquement.

> _Remarque_: rien ne l'empêche de renaître plus tard.

La possibilité d'obtention d'une mise à jour incrémentale depuis un état détenu à la date `d` est bornée par la possibilité de disposer des suppressions.
- si elles sont gardées, même _zombi_ avec une taille minimale, sans limite de temps, la base peut être encombrée de zombis.
- en fixant un délai d'un an par exemple, les mises à jour depuis un état de plus d'an sont traitées comme une demande _intégrale_ avec la fourniture de tous les documents existants et non plus _incrémentale_. C'est à l'application terminale de déterminer, si besoin est, les suppressions de documents depuis l'état à la date `d` qu'elle connaît et le nouvel état complet reçu.

Le serveur va à l'occasion d'une suppression d'un document regarder les `cleandate` du ou des fils auxquels est rattaché le document: si ces dates ont plus de 18 mois, il va purger effectivement les documents zombis depuis plus d'an de ces fils (en testant leur propriété `zombi`). Il mettra à jour la ou les `cleandate` du ou de ces fils.

De cette façon les documents supprimés sont purgés au fil du temps mais avec des opérations distantes de six mois au moins.

# Abonnements d'une application à des _fils_

**Un _fil_ fait aussi office de _news_, d'alerte:** quand un document change (un bon de commande), le ou les fils auxquels il est attaché (la livraison de samedi) sont informés et le fil a noté qu'un bon de commande a changé. 

**Si des applications s'étaient abonnées à ce fil**, leurs utilisateurs vont voir apparaître sur leurs écrans une _notification_ indiquant _qu'un bon de commande a changé pour la livraison de samedi_. Si l'application d'un utilisateur était lancée et que la page courante montrait cette livraison, l'application est allé chercher les bons de commande attachés au fil de la livraison de samedi ayant une version plus récente que celle que l'application a en mémoire: la page est mise à jour à l'écran sans intervention de l'utilisateur.

Suivant ce paradigme, une application présente à son utilisateur trois concepts:
- des **_fils d'information_** annonçant des évolutions de documents ou de collections de documents qui l'intéresse: l'arrivée de nouveaux échanges sur un _chat_ (un document), une évolution tarifaire (un tarif vu comme une collection de documents). Ces fils **annoncent** par des notifications courtes une évolution de certains documents, mais n'en donne q'un minimum d'information.
- des **fils de documents synchronisables**: les documents attachés à un fil synchronisé sont systématiquement maintenus à jour dans l'application dans un état le plus proche techniquement possible de l'état des documents sur le serveur.
- des **_rapports_**: ce sont vues calculées à un instant donné et qui ne changent qu'à redemande du même rapport.

**Les documents synchronisés dans une application** le restent a minima tant que l'application est **au premier plan**:
- l'application **peut** décider de ne plus maintenir cette synchronisation quand elle passe **en arrière plan**: c'est une économie de ressources et comme en pratique l'utilisateur ne voit d'une application en arrière plan que les _popups_ de notification, maintenir à jour un volume important de documents synchronisés n'a pas forcément d'intérêt.
- en repassant au premier plan, l'application demande aux services de lui fournir les mises à jour survenues sur les fils synchronisés depuis le dernier état synchronisé qui était détenu dans l'application.

### Quand l'application n'est plus en exécution
Quand une application est en exécution elle peut rester _abonnée_ à des _fils d'information_.

Quand son exécution s'arrête, sauf décision explicite de l'utilisateur, certains de ces abonnements restent actifs, du moins un certain temps:
- les notifications correspondantes continueront à s'afficher en _popups_, l'OS ou le browser de l'application s'en chargeant.
- l'utilisateur reste informé des _news_ auxquelles il était abonné.
- un clic sur un ces _popups_ ouvre l'application ce qui lui permet plus ou moins directement de connaître en détail les documents ayant changé.

> Chaque application détermine pour chaque fil auquel elle est abonné, si l'abonnement s'interrompt ou non quand l'application s'arrête.

### Des _fils_ plus ou moins riches
Pour assurer la synchronisation d'une collection de documents, le _fil_ correspondant est riche: il peut y avoir beaucoup de documents modifiés. Les **_fils de synchronisation_** ne donnent lieu à des _popups_ que sur des critères très restrictifs gérés par l'application afin de ne pas submerger l'utilisateur.

Les **_fils de news_** sont a contrario beaucoup plus sobres: ils correspondent à quelques documents / collections bien ciblés et pas à tous ceux qui seraient nécessaires à une synchronisation complète de ces documents.

**La caractéristique d'un _fil de news_ est qu'il peut produire des textes en popups de notification alors que l'application terminale ne s'exécute pas** et en conséquence le texte complet est à produire par le serveur. Pour que ces textes soient lisibles, l'abonnement à un _fil de news_ va fournir une petite structure de données permettant au serveur de générer le texte de la notification. Par exemple:
- la langue courante de la session au moment de l'abonnement,
- l'identifiant d'un texte _template_ traduit dans plusieurs langues,
- des intitulés / labels / noms associés afin que le texte les référencent plutôt qu'un code identifiant abscons pour l'utilisateur (le _nom_ d'un producteur, plutôt que son code).

# Modules génériques utilisables dans les logiciels serveurs
Ces modules forment une couche logicielle offrant un certain nombre de services de bases raccourcissant et sécurisant l'effort de développement.

## Gestion des _opérations_
**Une opération est initiée par la réception d'un requête HTTP**. La couche de base:
- identifie l'opération demandée en fonction de l'URL.
- récupère les paramètres d'entrée et les met à disposition de l'opération dans une structure.
- **gère l'opération comme une succession de phases:**
  - **phase 1:** vérification que les paramètres d'entrée sont bien formés, présents s'ils étaient obligatoires, etc. _Normalement_ l'application terminale s'en est assuré mais l'opération dans le serveur fait cette vérification pour éviter de tomber sur des exceptions dues à des valeurs qui n'auraient pas dues être trouvées en entrée. Cette phase s'effectue sans accès à la base de données.
  - **phase 2:** c'est une transaction au sens de la base de données. Elle lit, met à jour créé et supprime des documents et élabore un résultat. En cas d'exception détectée comme liée à une saturation technique de la base de données, cette phase est annulée et relancée (du moins un certain nombre de fois).
  - **phase 3:** c'est une phase de nettoyage qui peut effectuer quelques mises à jour dans la base de données, typiquement dans la table `toDelete` pour la gestion des fichiers.
  - **phase 4:** la phase 2 a produit une liste de fils de documents mis à jour. Cette phase identifie les abonnements à ces fils et génèrent les _notifications_ aux applications abonnées.
- **retourne le résultat à l'application appelante,**
  - construit en phase 2,
  - ainsi que les fils déterminés en phase 4 auxquels l'application appelante était abonné.

## Gestion des documents
Un module gère une _mémoire cache_ des documents les plus récemment demandés et mis à jour, du moins pour les types de documents spécifiés. 

Ainsi la demande par une opération en phase 2 d'un document qui se trouve en _cache_,
- soit délivre le document s'il n'est pas exigé à être absolument dans sa version la plus récente mais tolère une _certaine ancienneté_,
- soit s'assure que c'est bien la dernière version: si c'est le cas seul un index a été lu, sinon le document est lu et conservé en _cache_.

Le module gère également une _mémoire cache de documents_ **pour chaque opération en cours**.
- les documents lus y sont stockés dans leur forme _objet compilé_ (et non le format sérialisé de stockage en base).
- les documents supprimés et créés y sont aussi stockés.

A la fin de l'opération le module gère les _fils_ impactés par les documents modifiés / créés / détruits, et les met à jour dans la base. Ces _fils_ seront utilisés en phase 4 de l'opération pour notifier les applications abonnées.

## Requête de collecte des documents d'un fil
Quand une application terminale souhaite disposer des documents mis à jour pour un fil donné, ce module détermine en fonction du fil transmis:
- si la mise à jour peut être _incrémentale_ ou si elle doit être _intégrale_,
- quels documents sont à retourner en fonction des versions détenues par l'application terminale.

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
La structure des documents et leur rattachements aux fils se fait par _déclaration statique_ dans des modules de configuration. Il en est de même pour les options techniques de cryptage / hachage par type de document.

Toutefois dans ces déclarations il peut apparaître des noms de fonctions applicatives spécifiques qui sont à développer par codage et non plus par déclaration.

## Modules utilitaires
Un module de cryptographie évite de gérer les subtilités du paramétrages des algorithmes.

> Vis à vis des applications terminales, les serveurs ne connaissent **QUE** les concepts de documents et fils de documents: abonnements et mises à jour incrémentales. Le reste de la logique applicative passe par l'usage des opérations.

# L'application _Safe_ 
Cette application a pour objet de gérer des _coffres forts_ pour des utilisateurs.

En lançant _Safe_ un utilisateur donne les éléments d'identification de son _coffre fort_ de manière à ce que les applications lancées ultérieurement sur cet appareil puissent y trouver les diverses données _sensibles_ de l'utilisateur dont ses _droits d'accès_ aux documents des applications.

L'application _Safe_ une fois initialisé sur un appareil est en charge:
- de gérer ses appareils _de confiance_.
- pour chaque application:
  - de stocker `ses droits` et les mettre à jour.
  - de stocker _divers objets_:
    - des objets transmis par un autre utilisateur disposant lui aussi d'un _safe_. 
    - des objets interprétés comme des _options de lancement_.
    - des objets interprétés par l'application comme des _préférences_.
  - de lancer l'application, le cas échéant selon _l'option de lancement_ que l'utilisateur a sélectionnée.

> Le lancement d'applications par le _Safe_ évite le risque de lancement d'une application _piratée_ et permet de choisir le cas échéant des _options_ au lancement ouvrant la session dans un contexte déjà préfixé.

> L'URL de lancement de _Safe_ doit être soigneusement vérifiée: le code de cette application est lisible dans le browser et il est possible de vérifier auprès de sites certificateurs que l'application est _fair_. Ceci évite d'avoir à le faire pour chaque application gérée par _Safe_, quoi que ce soit toujours possible. 

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

> Même _de confiance_ un appareil _peut_ être utilisé par d'autres que soi-même, même dans un cadre familial ou de couple, l'appareil n'est pas strictement _personnel_.

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
- de laisser ses _fils de news_ activés: des notifications apparaîtront, même quand l'application sera fermée et même si ce n'est plus le même utilisateur qui dispose de l'appareil. Les _notifications_ ne font qu'annoncer des changements sans en donner les détails et évitent de délivrer des informations confidentielles. 
- de fermer ses _fils de news_: aucune notification ne parviendra plus sur l'appareil relativement à cette application. L'utilisateur peut le prêter à quelqu'un d'autre sans risque ... mais lui-même ne recevra plus de notifications (il faut choisir).

**En mode _avion_**, il n'y a pas de réseau, pas de _fils de news_: les dossiers peuvent être consultés mais pas mis à jour. 
- la base de données locale est cryptée par la clé `K` du _Safe_ de l'utilisateur. Elle contient les _fils de document_ du contexte fixé par l'utilisateur et les documents attachés: ils ne sont pas du tout dernier état mais a minima dans l'état où ils ont été accédés la dernière fois sur ce appareil.
- l'utilisateur peut saisir des textes ou formulaires purement locaux et stocker des fichiers (comme des photos prises en mode avion): toutes ces informations pourront être utilisées pour mettre à jour des documents quand le réseau sera à nouveau disponible en sécurité.

**En mode _incognito_** la fermeture de l'application ne laisse pas le choix: les abonnements aux _fils de news_ sont tous supprimés, aucune notification ne parviendra plus sur cet appareil résultant de l'usage précédent de l'application.

### Mode _veille_
Les applications ouvertes ont un mode _veille_ optionnel: s'il est activé, l'application se met en veille en cas de non utilisation pendant quelques minutes (fixés selon le degré de paranoïa de l'utilisateur). Pour sortir de la veille un code est nécessaire et au second échec l'application se ferme.

# Glossaire technique

### Strong Hash: `PBKDF`
- **SH(s1, s2, SEP)** (Strong Hash): le SH s'applique à un couple de textes `s1 s2`, typiquement un login / mot de passe, mais aussi aux _passphrase_ en une ou deux parties. Il a une longueur de 32 bytes et est unique pour chaque couple de textes `s1 s2`. Il est _strong_ parce qu'incassable par force brute dès lors que le couple de textes ne fait pas partie des _dictionnaires_ des codes fréquemment utilisés. `SEP` est un caractère de séparation / remplissage qui allonge le couple `s1 + SEP + s2` à une taille minimale.

### Clés asymétriques C / D : cryptage / décryptage
Un couple de clés `Ca / Da` asymétriques généré par A:
- `Ca` est une clé publique de **cryptage**: elle est utilisée par B pour crypter un objet qui ne pourra être décrypté que par A.
- `Da` est la clé privée de **décryptage**: elle est utilisée par A pour décrypter un objet qui a été crypté par B en utilisant `Ca`.

### Clés asymétriques S / V : signature / vérification
Un couple de clés `Sa / Va` asymétriques généré par A:
- `Va` est une clé publique de **vérification**: elle est utilisée par B pour _vérifier que la signature S d'un texte X_ a bien été générée par A en utilisant `Sa`.
- `Sa` est la clé privée de **signature**: elle est utilisée par A pour _générer la signature S d'un texte challenge X_.

-----------------------------------------------------------------------

# Décompter les consommations

Une application peut être gratuite pour certains de ses utilisateurs (voire tous) et payante pour d'autres selon l'usage qu'ils en font. 

Mais même dans le cas où l'application est toujours gratuite pour tous, il peut être cependant nécessaire / intéressant de décompter des unités d' œuvre d'utilisation afin le cas échéant de les limiter.

La mise en œuvre de **contrats** permet,
- d'effectuer des décomptes d'usage par des _utilisateurs_, 
- d'en gérer des limites,
- si souhaité d'établir des _factures / paiements / régularisations_.

## Contrats et utilisateurs

Un _utilisateur_ est identifié par l'application et tout utilisateur est toujours rattaché **à un instant donné** à un _contrat_. Au cours de sa vie un _utilisateur_ peut être détaché de son contrat précédent et être rattaché à un autre.

Un _contrat_ est un document d'identifiant aléatoire généré à sa création visant à suivre et gérer les consommations des utilisateurs qui lui sont rattachés: les consommations sont décomptées pour le mois _courant_ et le mois précédent (immuable par principe).

> Dans le cas général un _contrat_ concerne un _petit collectif d'utilisateurs_ une famille, une équipe, une petite association ... mais éventuellement un seul utilisateur.

Le décompte des consommations peut entraîner des contraintes, par exemple:
- rejet d'opération pour dépassement de certains seuils,
- ralentissement en cas de dépassement,
- blocage éventuel de certaines opérations pour certains utilisateurs.

## Unités d'œuvre des contrats (UC)

Chaque application déclare ses _unités d'œuvre_ selon deux catégories: les unités de _stock_ et les unités de _calcul / travail_.

Chaque unité a ses compteurs spécifiques: elles ne se moyennent ni se s'additionnent pas entre elles. Définir si c'est nécessaire des unités génériques englobantes.

Les unités sont décomptées,
- pour chaque _utilisateur_.
- globalement pour le _contrat_ (somme des valeurs pour chaque utilisateur).

### Unités de _stock_
Leur existence a un coût par le seul effet du temps qui passe. Par exemple:
- S1: nombre de commandes en cours.
- S2: `volume en Mo` de fichiers stockés.

Elles sont décomptées par _mois d'existence_, mais leur nombre pouvant varier à tout instant une moyenne pondérée du temps passé (à la milliseconde) est calculée. _Exemple_: 
- 30 commandes en cours pendant 10 jours (300),
- puis 10 commandes en cours pendant 20 jours (200),
- correspondent à 16,6 commandes par mois ((300 + 200) / 30).

### Unités de _calcul / travail_
Elles _coûtent_ effectivement lorsqu'elles sont sollicitées. Par exemple:
- C3: **nombre** de lecture de documents D1 et D2.
- C4: **volume en Mo** des fichiers de type F1 téléchargés.

Décompter ces unités sur un mois (par exemple le volume des fichiers téléchargés) revient à faire leur **somme** sur un mois (somme des téléchargements exécutés dans le mois).

#### Niveau des nombres d'unités
Un _niveau_ est codifié par deux chiffres `ab`, la _valeur_ d'un niveau étant `a * 10**b`.

Le niveau `10` vaut 1, `22` vaut 200, `52` vaut 500, `73` vaut 7000, `99` vaut 9,000,000,000.

La donnée d'un _niveau_ permet de fixer un ordre de grandeur d'un seuil pour une unité donnée.

## Maximum applicables à un contrat

Pour chaque UC déclarée, le contrat définit un _niveau maximum_, par exemple:
- S1-nombre de commandes en cours : `32` (soit 300)
- C4-volume en Mo des fichiers de type F1 téléchargés : `56` (soit 5000000 en l'occurrence 5Go).

Pour les unités de _stock_: le maximum fixé ne peut pas être dépassé, l'opération demanderesse tombe en exception.
- _sauf_ si l'opération est marquée _privilégiée_,
- _sauf_ si le maximum est certes dépassé mais en baisse.
- le **niveau d'alerte** est le pourcentage de dépassement au delà de 80% (0 en deçà).

Pour les unités de _calcul_ c'est le nombre moyen d'unités accumulé en 30 jours sur M et M-1:
  - le 10 du mois, le compte pour M est affecté du coefficient 1/3, celui de M-1 pour 2/3.
- le **niveau d'alerte** est le pourcentage de dépassement au delà de 80% (0 en deçà).
- l'application calcule une **durée de ralentissement de l'opération** (d'attente) fonction du niveau d'alerte afin de freiner l'excès de calcul, voire de bloquer l'opération (si elle n'est pas _privilégiée_).

### Au niveau de chaque _utilisateur_
Un niveau par UC est également fixé par _utilisateur_: 
- l'action de blocage / ralentissement et le niveau d'alerte sont les plus restrictifs des deux _utilisateur / contrat_.

## Décompte continu des unités sur un contrat: arrêté mensuel
Le décompte est établi en nombre pour toutes les unités effectivement consommées dans le mois courant et le mois précédent: il est disponible,
- par utilisateur,
- par sommation, globalement pour le contrat.
- pour les _unités de stock_ le nombre n'est pas entier puisqu'il s'agit d'une moyenne au temps passé effectivement (en milliseconde).

> A la fin de chaque mois un arrêté mensuel est calculé et mémorisé comme _mois précédent_ désormais invariant: l'application peut archiver les factures sur la profondeur d'historique de son choix avec un format adapté au traitement statistique.

**Pour chaque UC le décompte exact est _forfaitisé_**, converti / arrondi au _niveau_ le plus proche supérieur. Par exemple, la valeur 6542 est convertie en 7000 (code `73`), le _niveau forfaitaire_ immédiatement supérieur.

**Un historique sur N mois** des décomptes _forfaitisés_ par UC est conservé.

### Tarif: prix unitaire pour chaque UC
Un _tarif_ donne le prix unitaire pour chaque UC.

L'application du tarif à tous les décomptes _arrondis_ des UC au mois courant et au mois précédent donne un montant monétaire:
- _provisoire_ pour le mois courant,
- _définitif_ pour le mois précédent.

> Quand un _contrat_ est utilisé d'une manière assez stable dans le temps, les factures ont des chances d'être identiques d'un mois sur l'autre.

## Débits et crédits, solde
La _facture_ d'un mois comporte une liste de _débits / crédits_: chacun a,
- un _code_ qui indique sa nature: par exemple _paiement reçu_,
- une _référence_ ou _commentaire_ permettant d'en savoir plus dans l'application.
- un montant positif ou négatif.

La première ligne est un _débit_ correspondant à la consommation effective du _contrat_.

### Ajustement global de l'application
Une fois la consommation ainsi calculée, l'application peut introduire une ligne finale d'ajustement, au débit ou au crédit:
- remise sur volume,
- remise / pénalité selon le type de contrat,
- le cas échéant **remise égale au montant de la consommation** ce qui revient à de la gratuité. 

Le _solde_ en fin de mois correspond au solde en fin du mois précédent, plus les crédits, moins les débits.

En cours de mois un _solde provisoire_ est calculable simplement:
- depuis le solde au mois précédent,
- la consommation en cours _forfaitisée_,
- la ligne d'ajustement spécifique de l'application.
- la balance des débits / crédits du mois.

### Politique vis à vis d'un solde négatif
L'application fixe sa politique vis à vis d'un _contrat_ présentant un solde _provisoire_ négatif. Par exemple:
- tolérance si le solde en début de mois était positif,
- tolérance pour les opérations _privilégiées_,
- alerte selon le nombre de jours estimés avec _solde positif_: simple calcul du nombre de jours pendant lequel le solde restera positif si la consommation se maintient au même niveau que la moyenne du mois courant et du mois précédent.
- rejet de l'opération.

## Authentification d'utilisateurs via leur contrat

**OBSOLÈTE**
Tout utilisateur rattaché à un contrat _peut_ y être associé à une ou plusieurs **passphrases** qui l'authentifient: 
- une _passphrase_ donnée ne peut être associée qu'à un seul contrat à un instant donné et à un seul _utilisateur_ dans le contrat.
- dans son contrat, un utilisateur peut avoir _plusieurs passphrases_ associées lui permettant autant de solutions d'authentification.

> Une seule action est dans ce cas nécessaire pour authentifier un _utilisateur_ d'identifiant inconnu (mais en fournissant une de ses passphrases) et obtenir son _contrat_.

Hormis le cas ci-dessus, on accède à un contrat par son identifiant et on peut y retrouver tous ses utilisateurs enregistrés par leur identifiant.

### Suivi des consommations dans les opérations

Sauf exception toute opération du serveur commence par fixer un _contrat par défaut de l'opération_ à qui imputer ses consommations d'UC. 

Au cours de l'opération, d'autres _contrats_ peuvent être cités à qui imputer certaines consommations spécifiques selon la logique de l'application. Par exemple le stockage et l'utilisation de documents partagés par un **groupe** d'utilisateurs peut être imputé au _contrat du groupe_ plutôt qu'aux _contrats_ des utilisateurs eux-mêmes.

### Remarque: début et fin d'opération
Le début d'une opération va, en général, 
- lire le _contrat_ depuis la base et authentifier l'utilisateur demandeur de l'opération,
- en cas de basculement sur le mois suivant, calculer la facturation et repositionner les compteurs.

En cours d'opération, les compteurs du mois courant sont mis à jour (si nécessaire) et le _solde provisoire_ peut être réévalué.

La fin d'une opération va, en général, enregistrer en base la mise à jour du _contrat_.

Un _contrat_ a un mois de dernière mise à jour: un traitement périodique à partir du N d'un mois effectue a minima une opération sur chaque contrat dont le dernier mois de mise à jour n'est pas le mois courant afin d'éviter de perdre des facturations sur les _contrats_ peu utilisés.

Un traitement périodique peut aussi collecter un historique sous forme de fichier CSV pour analyse externe.

## Détail du document `Contrat`
Propriétés:
- code statistique: pour déclenchement sélectif d'actions / reports
- dernier mois facturé: pour déclencher le calcul de la facture
- montant consommation (moyenne M M-1): 
- indicateur de solde négatif

--------------

# Contributions diverses en attente

### Désérialisation de la propriété `data` du document
Elle consiste à retourner une _map_ nom, valeur des propriétés du document, dont celles d'identification et la version.

La couche applicative est en charge de créer une instance de la classe appropriée depuis cette _map_ en utilisant le type du document et si nécessaire d'autres propriétés de _data_ pour des sous-classes héritant d'une classe racine correspondant au type de document.

### Lecture d'un fichier
Elle peut s'effectuer de deux manières:
- en retournant le contenu binaire du fichier dans la couche applicative,
- en retournant une URL d'accès sécurisé valable un certain temps, typiquement à transmettre à une application externe.

### Cohérence _forte_ dans un fil, _faible_ entre fils
L'état d'un fil retourné par une requête est _fortement cohérent_: cette configuration a existé vraiment à un moment donné.

Mais deux demandes faites pour deux fils, forcément à des moments différents, retourne deux états de fils qui ont pu ne jamais exister conjointement: il en résulte une _cohérence faible_ entre fils, un état qui globalement peut être fonctionnellement incohérent temporairement.

> On pourrait certes grouper dans la même requête des demandes concernant plusieurs fils: toutefois le volume correspondant retourné peut être important et la transaction correspondante de collecte être longue et induire des blocages techniques de la base de données. Il y a applicativement un compromis à choisir entre _force de la cohérence entre arbres_ et lourdeur technique.

## Authentification _double_
(questions, pertinence)

Faut-il prévoir d'obliger à une authentification depuis un appareil #1 exigeant une confirmation sur un appareil #2 (favori ou non).
- que se passe-t-il quand l'appareil #1 n'est déclaré _favori_ ?
- et si l'utilisateur N'A PAS d'appareil #2 (au moins sous la main) ?
- si #2 n'est PAS favori, pour valider le login de #1 il lui faut un couple (s1, s2) qu'il vient a priori déjà de donner sur #1 ?

### Le répertoire des _profils_ des utilisateurs
Ce répertoire n'est accédé que par les applications terminales.

Tout utilisateur peut s'y faire enregistrer son _profil_.
- le profil est crypté, seul son utilisateur peut donner aux applications terminales les clés de décryptages qu'il est seul à connaître.
- disposer d'un _profil_ n'est requis que pour un utilisateur souhaitant enregistrer un ou des _appareils favoris_ et de bénéficier des avantages associés (voir plus avant).

Le _profil_ d'un utilisateur est identifié par un `userid`, un code généré aléatoirement à son inscription, et peut contenir les rubriques suivantes:
- la liste de **ses _appareils favoris_** comportant des données cryptographiques.
- la liste de **ses _préférences_**: données _pré-saisies_ souvent demandées par les applications ou choix simples (langue préférée, mode _sombre / clair_ ...).
- la liste de **ses _sessions favorites_**. En choisissant une session favorite, un utilisateur se retrouve à l'ouverture d'une application avec le _desktop de l'application_ initialisé avec les documents visibles adaptés à son besoin et les _credentials_ correspondant.

> Un _credential_ est une petite structure de données ayant un _type_ donné et donnant les autorisations d'accès pour un objet précis: par exemple le type `CRDCO` donne une autorisation à un _consommateur donné attaché à un point de livraison donné_ en citant ses initiales et son mot de passe. Les accès aux documents et le droit d'invoquer des opérations requièrent un ou des _credentials_.

### Fils et _credentials_
Le _credential_ attaché à un fil gouverne le droit à en lire les documents et à s'y abonner. 

Les autorisations de création / mise à jour sont gérées par l'application selon des règles applicatives plus riches.

Chaque _type de fil_ est associé à un _type de credential_:
- les paramètres du credential sont mentionnées comme propriétés identifiantes du type de fil.
- par exemple le fil `CMDGP` est identifié par `gp.livr`.
- son _credential_ associé sera par exemple `CREDGP` identifié par `gp`.
- pour accéder à un _fil_ d'une livraison d'un groupement, il faut avoir le _credential_ de ce groupement.

> Des documents de ce fil, par exemple les _cartons_, apparaissent aussi dans un autre fil relatif au point-de-livraison (`CMDGC` identifié par `gc.gp.livr`). Ce second fil sera associé à un _credential_ `CREDGC` identifié par `gc`. Les _cartons_ seront donc accessibles soit en ayant un _credential_ `CMDGP`, soit un _credential_ `CMDGC`, avec en conséquence des notifications de deux ordres avec des autorisations différentes.

## Les _activités_ définies dans une application

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

## Choisir et exercer une activité dans une session d'une application terminale
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
