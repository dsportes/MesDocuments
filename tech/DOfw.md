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

Le changement de version est en général automatique mais peut être opérée manuellement.

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

## LA base de données partagée par tous les prestataires
Elle contient:
- Le **répertoire des organisations par application** qui liste pour chaque application et chaque organisation le prestataire gestionnaire.
- Le **répertoire des _profils_ des utilisateurs**.

### Répertoire des _organisations par application_
Toute application terminale détient la liste des _prestataires_ fournissant les services centraux, leur _code_ et leur _URL d'accès_. 
- L'ajout ou le retrait d'un prestataire provoque une nouvelle version des applications concernées (l'installation est automatique).

Une session d'une application terminale peut concerner plusieurs organisations, à l'instar du randonneur faisant partie de plusieurs associations selon l'endroit où il randonne. Pour chaque organisation concernée  elle obtient de ce répertoire le prestataire gestionnaire.

Ce répertoire contient la liste des triplets `{ application, prestataire, organisation }` déclarés par les prestataires:
- ils peuvent en exporter des listes sélectives, en particulier pour une application / prestataire, la liste des organisations gérées. 
- une application terminale peut faire appel à n'importe quel prestataire afin de récupérer le prestataire traitant une organisation donnée.

**Remarque** : chaque prestataire peut ensuite gérer **dans _sa_ base de données**, un document relatif à l'organisation comportant:
- un **statut** : est-elle ouverte, restreinte en lecture seule (archive), fermée jusqu'à nouvel ordre.
- une **courte liste de _news_** données par l'administrateur.
- les applications terminales peuvent s'abonner aux modifications de ce document.

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

La propriété `del` contient le jour de suppression quand la document est en état  _zombi_, supprimé logiquement mais pas purgé physiquement.

### Stockage d'un document d'un type donné
Le document est stocké dans une table (SQL) ou une collection (NOSQL) spécifique du type de document.

**L'ensemble des propriétés** est sérialisé dans un champ dénommé `_data_`: ce contenu est désérialisable dans les applications terminales et les serveurs.

En base de données, les propriétés **visibles de la base de données** sont:
- `id` : clé primaire ou path.
- `version`.
- `del`.
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

Une application terminale qui a gardé en mémoire la dernière image d'un fil qui lui lui a été transmise, peut à réception d'un nouvel état de ce fil, demander à un serveur la liste des documents de version postérieure à celle qu'elle détenait et en effectuer la mise à jour dans sa mémoire. Cette mise à jour est :
- _optimale_: elle n'est demandée QUE si un des documents d'un type qui intéresse l'application a changé. De plus le filtrage s'effectuant sur la propriété indexée `version`, seul l'index est sollicité (ce qui pour certaines bases NOSQL est gratuit) la _lecture effective_ n'étant pas faite pour les documents non modifiés.
- _incrémentale_: seuls les documents ayant changé depuis la version connue de l'application terminale sont lus et transmis.

### Fils et _credentials_
Le _credential_ attaché à un fil gouverne le droit à en lire les documents et à s'y abonner. 

Les autorisations de création / mise à jour sont gérées par l'application selon des règles applicatives plus riches.

Chaque _type de fil_ est associé à un _type de credential_:
- les paramètres du credential sont mentionnées comme propriétés identifiantes du type de fil.
- par exemple le fil `CMDGP` est identifié par `gp.livr`.
- son _credential_ associé sera par exemple `CREDGP` identifié par `gp`.
- pour accéder à un _fil_ d'une livraison d'un groupement, il faut avoir le _credential_ de ce groupement.

> Des documents de ce fil, par exemple les _cartons_, apparaissent aussi dans un autre fil relatif au point-de-livraison (`CMDGC` identifié par `gc.gp.livr`). Ce second fil sera associé à un _credential_ `CREDGC` identifié par `gc`. Les _cartons_ seront donc accessibles soit en ayant un _credential_ `CMDGP`, soit un _credential_ `CMDGC`, avec en conséquence des notifications de deux ordres avec des autorisations différentes.

### Traitements dans un serveur: attachement / mise à jour d'un document dans un fil
- récupération des `version` `vi` de tous les fils dans lequel le document est à insérer / mettre à jour.
- la version du document `v` est le maximum des `vi` + 1.
- dans chacun de ces fils:
  - la `version` est mise à `v`.
  - dans `_data_` la `version` pour le type du document est mise à `v`.

### Traitement des _suppressions_
Pour que la mise à jour soit incrémentale dans une sous-collection d'un type de documents, un document ne peut pas être simplement _supprimé_: en demandant la liste des documents ayant changé il n'apparaîtrait pas, serait considéré comme inchangé et sa suppression non détectée.

Le document supprimé va être traité comme une mise jour particulière:
- sa `version` est mise jour (comme pour une mise jour normale).
- ses propriétés indexables sont mises à null.
- sa propriété `del` donne le jour de suppression.
- son _data_ est mis à null.

Le document est en état _zombi_, supprimé logiquement.

> _Remarque_: rien ne l'empêche de renaître plus tard.

La possibilité d'obtention d'une mise à jour incrémentale depuis un état détenu à la date `d` est bornée par la possibilité de disposer des suppressions.
- si elles sont gardées, même _zombi_ avec une taille minimale, sans limite de temps, la base peut être encombrée de zombis.
- en fixant un délai d'un an par exemple, les mises à jour depuis un état de plus d'an sont traitées comme une demande _intégrale_ avec la fourniture de tous les documents existants. C'est à l'application terminale de déterminer, si besoin est, les suppressions de documents depuis l'état à la date `d` qu'elle connaît et le nouvel état complet.

Le serveur va à l'occasion d'une suppression d'un document regarder les `cleandate` du ou des fils auxquels est rattaché le document: si ces dates ont plus de 18 mois, il va purger effectivement les documents zombis depuis plus d'an de ces fils (en testant leur propriété `del`). Il mettra à jour la ou les `cleandate` du ou de ces fils.

De cette façon les documents supprimés sont purgés au fil du temps mais avec des opérations distantes de six mois au moins.

# Abonnements d'une application à des _fils_

**Un _fil_ fait aussi office de _news_, d'alerte:** quand un document change (un bon de commande), le ou les fils auxquels il est attaché (la livraison de samedi)sont informés et le fil a noté qu'un bon de commande a changé. 

**Si des applications s'étaient abonnées à ce fil**, leurs utilisateurs vont voir apparaître sur leurs écrans une _notification_ indiquant _qu'un bon de commande a changé pour la livraison de samedi_. Si l'application d'un utilisateur était lancée et que la page courante montrait cette livraison, l'application est allé chercher les bons de commande attachés au fil de la livraison de samedi ayant une version plus récente que celle que l'application a en mémoire: la page est mise à jour à l'écran sans intervention de l'utilisateur.

Suivant ce paradigme, une application présente à son utilisateur trois concepts:
- des **_fils d'information_** annonçant des évolutions de documents ou de collections de documents qui l'intéresse: l'arrivée de nouveaux échanges sur un _chat_ (un document), une évolution tarifaire (un tarif vu comme une collection de documents). Ces fils **annoncent** par des notifications courtes une évolution de certains documents, mais n'en donne q'un minimum d'information.
- des **fils de documents synchronisables**: les documents attachés à un fil synchronisé sont systématiquement maintenus à jour dans l'application dans un état le plus proche techniquement possible de l'état des documents sur le serveur.
- des **_rapports_**: ce sont vues calculées à un instant donné et qui ne changent qu'à redemande du même rapport.

**Les documents synchronisés dans une application** le restent a minima tant que l'application est **au premier plan**:
- l'application peut décider de ne plus maintenir cette synchronisation quand elle passe **en arrière plan**: c'est une économie de ressources et comme en pratique l'utilisateur ne voit d'une application en arrière plan que les _popups_ de notification, maintenir à jour un volume important de documents synchronisés n'a pas forcément d'intérêt.
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
  - ainsi que les fils déterminés en phase 4 auxquels l'application appelante était abonnés.

## Gestion des documents
Un module gère une _mémoire cache_ des documents les plus récemment demandés et mis à jour, du moins pour les types de documents spécifiés. 

Ainsi la demande par une opération en phase 2 d'un document qui se trouve en _cache_,
- soit délivre le document s'il n'est pas exigé à être absolument dans sa version la plus récente mais tolère une _certaine ancienneté_,
- soit s'assure que c'est bien la dernière version: si c'est le cas seul un index a été lu, sinon le document est lu et conservé en _cache_.

Le module gère également une _mémoire cache de documents_ pour chaque opération en cours.
- les documents lus y sont stockés dans leur forme _objet compilé_ (et non le format sérialisé de stockage en base).
- les documents supprimés et créés y sont aussi stockés.

A la fin de l'opération le module gère les _fils_ impactés par les documents modifiés / créés / détruits, et les met à jour dans la base. Ces _fils_ seront utilisés en phase 4 de l'opération pour notifier les applications abonnées.

## Requête de collecte des documents d'un fil
Quand une application terminale souhaite disposer des documents mis à jour pour un fil donné, ce module détermine en fonction du fil transmis:
- si la mise à jour peut être _incrémentale_ ou si elle doit être _intégrale_,
- quels documents sont à retourner en fonction des versions détenues par l'application terminale.

## Providers d'accès à la base de données
Un provider présente un interface indépendant de la base de données gérée. 
- pour chacun des services de cet interface, il implémente l'accès effectif à la base qu'il gère:
  - mise en forme / sérialisation des documents,
  - cryptages / hachages éventuels des _data_ et propriétés indexées,
  - gestion des transactions commit / rollback.

Pour un serveur donné, le (ou les ?) _providers_ requis sont importés.

Un _provider_ d'accès à **LA** base commune hébergeant les répertoires des services est disponible afin de masquer la technologie effective utilisée dans cette base.

> Typiquement en _test_ l'usage du provider _SQLIte_ simplifie le développement plutôt que ceux qui seront utilisés effectivement en production (_Postgresql_, _Firebase_...).

## Providers d'accès au _storage_
Les quelques services généraux d'accès sont développés pour chaque type de storage souhaité (AWS-S3, GCP, File-system ...).

## Principe de _déclaration_ statique
La structure des documents et leur rattachements aux fils se fait par _déclaration statique_ dans des modules de configuration. Il en est de même pour les options techniques de cryptage / hachage par type de document.

Toutefois dans ces déclarations il peut apparaître des noms de fonctions applicatives spécifiques qui sont à développer par codage et non plus par déclaration.

## Modules utilitaires
Un module de cryptographie évite de gérer les subtilités du paramétrages des algorithmes.

> Vis à vis des applications terminales, les serveurs ne connaissent **QUE** les concepts de documents et fils de documents: abonnements et mises à jour incrémentales. Le reste de la logique applicative passe par l'usage des opérations.

# Un utilisateur, ses appareils et ses applications

Un utilisateur qui veut utiliser une application (Web-PWA ou non) est placé devant deux cas de figure:
- **soit l'appareil qu'il s'apprête à utiliser est _familier_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - c'est un appareil de confiance: il peut y laisser quelques informations cryptées et espérer raisonnablement les retrouver plus tard.
- **soit l'appareil qu'il s'apprête à utiliser ne lui est pas familier**, il est partagé par des utilisateurs inconnus comme au cyber-café ou est celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelques informations que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour raccourcir ses saisies.

### Appareil _favori_
Pour un utilisateur lancer une application depuis un appareil _favori_ a plusieurs avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant de petites bases de données locales spécifiques du profil comme _cache_ de documents.
- **possibilité d'accéder à l'application en mode _avion_** sans accès au réseau.
- **identification plus rapide**, mais sûr, par usage d'un code PIN.

Même _favori_ un appareil _peut_ être utilisé par d'autres que soi-même, même dans un cadre familial ou de couple, l'appareil n'est pas strictement _personnel_.

Le _login_ à un appareil est _protégé_ par un mot de passe ou tout autre dispositif que seuls des proches connaissent, à moins que l'appareil leur soit prêté déverrouillé.

#### Des _contextes personnels_ habituellement plus ou moins bien défendus
Un browser comme Firefox a une notion d'utilisateur: on peut basculer d'un utilisateur à un autre (sans pour autant avoir changé de connexion au niveau de l'OS). Chacun a ses sites favoris, son historique de navigation et ses mots de passe enregistrés.

Thunderbird, le gestionnaire de mails locaux, supporte de gérer plusieurs _profils_, chacun avec ses comptes mails.

Mais ce n'est pas parce qu'on partage un appareil avec un proche qu'on a envie de partager avec lui ses informations confidentielles.

Or dans les cas cités ci-dessus, la confidentialité est plutôt _lâche_:
- Thunderbird ne demande rien: on choisit son profil, sans mot de passe ou quoi que ce soit. Les boîtes mail sont de toutes les façons en clair dans le file-system, confidentialité _intra-familiale_ zéro.
- Chrome s'ouvre sur le compte _courant_: si vous ne vous déconnectez pas **explicitement** avant de fermer le browser, il s'ouvre la fois suivante sur votre compte, ses mots de passe, ses historiques et ses favoris. Si vous vous déconnectez Chrome est strict sur la connexion et demandera même à votre mobile si c'est vraiment vous qui essayez de vous connecter: il suffit d'y répondre OUI et c'est bon. Même si vous vous êtes fait voler votre mobile en état déverrouillé, Google est content.

#### Déclarer son ou ses _alias_ sur un appareil _favori_
Un utilisateur déclare un appareil comme _favori_ en fixant,
- un _alias_ par exemple _Bob sur le mobile d'Alice_.
- un _code PIN_ d'au moins 8 signes. Voir le détail plus avant.
- une phrase d'authentification de deux termes `s1 s2`.

Si l'utilisateur était déjà enregistré par cette phrase dans le répertoire central des _profils_, un nouvel appareil favori sera ajouté à son _profil_.

Si aucun utilisateur n'est enregistré par cette phrase dans le répertoire central des _profils_:
- il lui est demandé de confirmer cette phrase.
- il lui est attribué, 
  - un `userid` aléatoirement,
  - aléatoirement une clé de cryptage personnelle nommée par la suite `Kp` et enregistrée cryptée dans son _profil_.
- son _profil_ est créé dans le répertoire avec son premier appareil _favori_:
  - `s1` et `s2` ont au moins 16 signes.
  - `s1` ne doit pas avoir été déclaré par un autre utilisateur ayant un profil enregistré dans le répertoire.
  - `s1` et `s2` ne sont **PAS** lisibles dans le répertoire (voir plus loin comment ils sont stockés _hachés_).

**Quelques informations sont aussi mémorisés sur l'appareil lui-même** dans une micro base de données `LocalProfiles` dont l'espace est gérée par le _browser_. Le contenu en est crypté, illisible en _debug_ et  même l'accès (malaisé) par le _file-system_ de l'OS ne permet pas à un _hacker_ d'accéder à son contenu.

### Lancement d'une application depuis un appareil _favori_
La liste des quelques profils enregistrés est présentée: l'utilisateur choisit le sien par exemple _Bob sur le mobile d'Alice_ et donne **son code PIN**.

L'application ouvre sa page d'accueil où l'utilisateur peut choisir,
- soit une de ses _sessions favorites_ donnant directement accès à des fils de documents dont les identifiants et _credentials_ sont enregistrés dans la _session favorite_.
- soit d'accéder à ses données en donnant les identifications et _credentials_ requis (ce qu'il veut faire, ou, au nom de qui ...).

En fermant l'application, l'utilisateur peut choisir:
- de laisser ses _fils de news_ activés: des notifications apparaîtront, même quand l'application sera fermée et même si ce n'est plus le même utilisateur qui dispose de l'appareil. Les _notifications_ ne font qu'annoncer des changements sans en donner les détails et évitent de délivrer des informations confidentielles. 
- de fermer ses _fils de news_: aucune notification ne parviendra plus sur l'appareil relativement à cette application. L'utilisateur peut le prêter à quelqu'un d'autre sans risque ... mais lui-même ne recevra plus de notifications (il faut choisir).

### Lancement d'une application EN MODE AVION sur un appareil _favori_
La liste des quelques profils enregistrés est présentée: l'utilisateur choisit le sien et donne sa **phrase d'authentification de deux termes `s1 s2`**. 
- Remarque: il ne donne **PAS** son code PIN qui doit être confronté aux données de son _profil_ dans le _répertoire des profils_ ce qui nécessite un accès Internet.

L'application ouvre sa page d'accueil où l'utilisateur peut choisir une de ses sessions _favorites_ ouvrant directement accès à ses documents enregistrés au cours d'une session antérieure pas en mode _avion_.

En mode avion, il n'y a pas de réseau, pas de _fils de news_: les dossiers peuvent être consultés mais pas mis à jour. L'utilisateur peut saisir des textes ou formulaires purement locaux et stocker des fichiers (comme des photos prises en mode avion): toutes ces informations sont cryptées dans le stockage local et pourront être utilisées pour mettre à jour des documents quand le réseau sera à nouveau disponible en sécurité.

### Lancement d'une application sur un appareil _anonyme_
L'utilisateur **PEUT** fournir une phrase d'authentification de deux termes `s1 s2`.

Si l'utilisateur était déjà enregistré par cette phrase dans le répertoire central des _profils_, il bénéficie des données enregistrées dans son _profil_.

Si aucun utilisateur n'est enregistré par cette phrase dans le répertoire central des _profils_, il lui est demandé de confirmer cette phrase et un _profil_ est créé pour lui dans le répertoire des profils avec la génération et l'enregistrement de son `userid` et de sa clé `Kp`.

Si l'utilisateur n'avait pas de _profil_ enregistré, ou simplement pas souhaité saisir sa phrase `s1 s2`, l'application propose un _desktop_ où il devra indiquer:
- son intention, avec quel rôle il souhaite accéder aux documents,
- quels documents il souhaite accéder et fournir le ou les _credentials_ associés.
- le cas échéant il choisira les _fils de news_ qui l'intéresse et donnera le cas échéant les _credentials_ associés.

> Pour l'application, un utilisateur _sans profil_ a les mêmes possibilités qu'un utilisateur ayant un _profil enregistré_. Il doit saisir plus d'informations pour faire valoir ses droits d'accès, bref les données qui sont pré-remplies pour une session déclarée _favorite_ pour l'utilisateur.

La fermeture de l'application ne laisse pas le choix sur un appareil _anonyme_: les abonnements aux _fils de news_ sont tous supprimés, aucune notification ne parviendra plus sur cet appareil résultant de l'usage précédent de l'application.

### Mode _veille_
Les applications ouvertes ont un mode _veille_ optionnel: s'il est activé, l'application se met en veille en cas de non utilisation pendant quelques minutes (fixés selon le degré de paranoïa de l'utilisateur). Pour sortir de la veille un code est nécessaire et au second échec l'application se ferme.

### Accès d'un utilisateur à son _profil_
Depuis n'importe quelle application terminale, après avoir fourni son code PIN (si c'est un appareil _favori_) ou sa phrase d'authentification `s1 s2`, l'utilisateur peut afficher en clair les rubriques de son _profil_.
- en particulier il peut voir la liste de ses _appareils favoris_ avec leur alias et supprimer ceux jugés inutiles.
- il peut effectuer des mises à jour de ses _préférences_.
- il peut supprimer des _sessions favorites_.

# Glossaire technique

- **SH(s1, s2)** (Strong Hash): le SH s'applique à un couple de textes `s1 s2`, typiquement un login / mot de passe, mais aussi aux _passphrase_ en une ou deux parties. Il a une longueur de 32 bytes et est unique pour chaque couple de textes `s1 s2`. Il est _strong_ parce qu'incassable par force brute dès lors que le couple de textes ne fait pas partie des _dictionnaires_ des codes fréquemment utilisés.
- **PP-x** : couples de clés publique / privée (Pub / Priv).
- **K-x** : clés AES de 32 bytes.

Soit `PP-S` le couple de clés généré par un serveur:
- `Pub-S` est une clé publique: toutes les applications l'obtiennent librement par un simple GET au serveur.
- `Priv-S` est la clé privée du serveur et fait partie des _secrets_ du logiciel serveur.

Soit `PP-ti` un couple de clés généré par une application terminale `ti` pour une conversation donnée avec le serveur `S`:
- `Pub-ti` est transmise dans les requêtes au serveur.
- Le serveur peut générer une clé `K-S-ti` à partir du couple `Pub-ti / Priv-S`: il s'en sert pour crypter ses réponses à l'application terminale ou toute donnée dont il veut que seule `ti` puisse les lire.
- L'application terminale peut générer une clé `K-ti-S` à partir du couple `Pub-S / Priv-ti`. Comme `K-ti-S` et `K-S-ti` sont égales, l'application terminale peut s'en servir pour décrypter les réponses / données cryptées pour elle par l'application serveur (et nul autre ne peut le faire).

Les applications terminales connaissent la clé publique du serveur Pub-S. Quand l'une d'elle veut échanger des données confidentielles avec le serveur:
- elle génère un couple PP-t,
- elle envoie ses requêtes au serveur en fournissant Pub-t.
- elle décryptera les réponses / données cryptées par le serveur par la clé Pub-S.

### Les applications peuvent-elles utiliser le `userid` d'un profil d'utilisateur ?
Une application terminale **à condition** que l'utilisateur se soit enregistré, dispose du _profil_ en clair de l'utilisateur, donc de son `userid`.
- elle peut le communiquer dans un _credential_ à un serveur accompagné du `SH(s1, s2)` prouvant que l'utilisateur a bien fourni sa clé d'accès `(s1, s2)`.
- à la première présentation de ce `userid`, typiquement à l'enregistrement de l'utilisateur, le serveur va conserver le couple `(userid, sha(SH(s1, s2))` dans la base de l'application.
- les authentifications ultérieures se feront seulement à partir de la base du prestataire et non de la base commune à tous.
- ultérieurement l'utilisateur peut aussi bien s'authentifier directement auprès du serveur à partir de `(s1, s2)` que d'un accès depuis un appareil favori et la saisie d'un code PIN.

Les applications **peuvent** en conséquence utiliser le répertoire des profils des utilisateurs pour les authentifier **MAIS** ceci oblige les utilisateurs à enregistrer leur _profil_ dans ce répertoire pour utiliser l'application ce qui peut pose un problème déontologique car des liens logiques pourraient être établis entre elles.

Une solution consiste à permettre à un utilisateur d'une application qui souhaite disposer d'un `userid`,
- soit d'en créer un spécifique de l'application avec une authentification spécifique,
- soit, au choix de l'utilisateur, d'utiliser son `userid` du répertoire des profils des utilisateurs, externe aux applications.

# Détail d'un _profil_
Il y a plusieurs rubriques dans le _profil_ d'un utilisateur:
- des _préférences_,
- des _sessions favorites_,
- des _appareils favoris_.

> Dans le répertoire des profils l'utilisateur est anonyme, inconnu des GAFAM, ne contient aucune information personnelle, ni nom, ni adresse e-mail, ni numéro de mobile (sauf à les avoir volontairement saisies en _préférences_ mais elles sont cryptées). Son enregistrement dans ce répertoire est inviolable, pour autant que les couples `s1 s2` en clés principales et de secours, qu'il a choisi soient respectueux d'un minimum de règles simples.

### Propriétés racines
Par sécurité un utilisateur _peut_ déclarer **deux** phrases d'authentification:
- `s1 s2` : phrase _principale_, `sh11` est le SH(s1, s1), `sh12` est le SH(s1, s2)
- `s1s s2s` : phrase de secours pouvant être invoquée en cas d'oubli de la première. `sh11s` est le SH(s1s, s1s), `sh12s` est le SH(s1s, s2s)

- `userid` : code aléatoire généré à l'inscription.

- `Kp` : clé personnelle de cryptage du _profil_ crypté dans le répertoire des profils par `sh12`.
- `sha11`: SHA de `sh11`. C'est un index d'accès au _profil_ dans le répertoire des profils et permet de vérifier l'unicité de `s1`.
- `s1s2` : couple `s1 s2`. Dans le répertoire des profils il est crypté par `Kp`.

Symétriquement il est défini `Kps sha11s s1s2s` à partir de la phrase d'authentification de secours.

> **Remarque:** pour pouvoir lire les phrases d'authentification `s1 s2` et `s1s s2s`, l'utilisateur devra soit avoir fourni l'une des deux, soit avoir fourni un code PIN depuis un appareil favori. Dans ces conditions l'utilisateur peut changer l'une ou l'autre de ses phrases d'authentification (après double saisie de vérification).

### Préférences
Une _préférence_ est une donnée nommée pour laquelle l'utilisateur a donné une ou des valeurs par défaut / préférées:
- langue préférée,
- mode sombre / clair,
- nom, e-mail, adresses, numéros de téléphone ...
- etc.

Quand une application a besoin de l'une de ces informations, elle propose à l'utilisateur en pré-saisie la ou l'une des valeurs inscrites en _préférences_ si elle y en a. L'utilisateur peut en sélectionner une ou en saisir une autre (à enregistrer ou non en préférence).

### _Sessions favorites_ d'un utilisateur
Lorsqu'un utilisateur ouvre une application terminale il commence une _session_: en général il ne peut pas faire grand-chose avant d'avoir déclaré a minima, 
- a) son intention, qu'est-ce qu'il veut y faire, 
- b) un _credential_ démontrant son droit à accéder aux documents et aux actions associées.

Un utilisateur peut aussi débuter une session avec plus de droits et un périmètre d'action plus large au cours de laquelle il aura:
- a) un accès _consommateur_ dans le point-de-livraison où il est enregistré.
- b) deux accès _point-de-livraison_ pour les deux organisations où il intervient pour aider à gérer des livraisons.
- c) un accès _groupement_ parce qu'il est également l'assistant d'un groupement de producteur qui n'est pas autonome.

> Exactement comme un utilisateur de Discord peut avoir accès à plusieurs _serveurs_ pour autant de sujets d'intérêt.

En cours d'une session, l'utilisateur peut ouvrir de nouveaux accès pour de nouveaux rôles après avoir fourni le cas échéant de nouveaux _credentials_. Les paramètres de l'état courant de la session peuvent être enregistrés comme _session favorite_, une nouvelle ou en remplaçant une antérieure.

Lors d'une prochaine ouverture de l'application, après avoir donné son code PIN (sur appareil favori) ou sa phrase d'authentification `s1 s2`, l'utilisateur n'a plus qu'à cliquer sur l'une de ses sessions favorites pour avoir à disposition tous les documents correspondants sans avoir eu à en citer d'identifiants ni à fournir de credentials.

> Au lieu d'une logique organisée autour de l'identification de _personnes_ (plus ou moins virtuelles), c'est l'utilisateur qui sélectionne une _session_ où il a plusieurs rôles. Chaque _ensemble de  documents et actions associés_ est comme enfermé dans un coffre, tout utilisateur en connaissant la combinaison peut prétendre y accéder (avec la possibilité de gérer plusieurs combinaisons par coffres).

#### Remarques techniques
- l'application terminale lors de l'enregistrement d'un utilisateur allonge `s1` en `s1+` par un texte de remplissage quand sa longueur est inférieure au maximum mais supérieure au minimum (refus si inférieur à la longueur minimale). L'unicité du `SH(s1+, s1+)` est vérifiée.
- l'application terminale d'enregistrement allonge `s2` en `s2+` de même, mais le texte de remplissage est généré en fonction de `s1`.
- l'unicité de `s1` est vérifiée par l'enregistrement du `SHA(SH(s1+, s1+))`.
- l'unicité de `s1 s2` est vérifiée par l'enregistrement du `SHA(SH(s1+, s2+))`.

##### Accès par l'utilisateur à son _profil_
Quand un utilisateur est enregistré, l'ouverture d'une application terminale lui propose de d'utiliser son _profil_: 
- lui demande l'un de ses couples d'accès `s1 s2`. L'application en construit les couples `SH(s1+, s1+)` et `SH(s1+, s2+)`. 
- l'application terminale soumet une requête à un serveur qui:
  - peut par `SHA(SH(s1+, s1+))` accéder à l'entrée `userid` pour cet utilisateur,
  - vérifier la validité de `SH(s1+, s2+)`. Dans ce cas il retourne le _profil_ crypté à l'application terminale.
- l'application terminale peut décrypter la clé `Kp` cryptée par `s1 + s2` (ce que le serveur ne pouvait pas faire faute de connaître `s1` et `s2`) et décrypter toutes les données du _profil_.

## Stockage local dans un appareil _favori_
Une **micro base locale des alias** stocke quelques données relatives aux utilisateurs ayant déclaré l'appareil comme _favori_. Elle est hébergée / gérée par le browser dans un espace spécifique du _domaine_ de l'application terminale.

La base a une table ayant une ligne par _alias_ d'utilisateur comportant:
- `alias` : l'alias choisi par un des utilisateurs (par exemple _Bob sur le mobile d'Alice_).
- `ka` : une clé de 32 bytes générée à la création de l'entrée. Elle est cryptée _mollement_ par une clé détenue dans le source de l'application terminale (donc lisible en debug avec un peu de fatigue).
- `fp` : le _profil_ de l'utilisateur cryptée par sa clé `Kp`.
- `ckp` : le couple de 2 cryptages de la clé `Kp` de l'utilisateur par respectivement les deux clés `(s1 + s2)`, la principale et celle de secours.

#### Déclarer un appareil comme favori
La liste des _alias_ des utilisateurs ayant utilisé cet appareil comme favori est présentée: l'utilisateur peut ainsi déterminer s'il doit,
- déclarer cet appareil comme favori.
- si c'était déjà le cas, changer son code PIN.
- supprimer les entrées des _alias_ qui ne l'inspirent pas.

Pour déclarer l'appareil comme favori ou refixer son code PIN, l'utilisateur doit fournir,
- l'alias de son choix, sélectionné dans la liste ou inventé à l'instant,
- **un code PIN d'au moins 8 signes**,
- une de ses deux clés longues `s1 s2` qui permet à l'application de retrouver son _profil_ dans le répertoire des profils.

> Si l'alias avait déjà une entrée, l'application terminale va essayer le code PIN proposé: en cas d'échec, l'entrée de l'alias dans le _profil_ est supprimée.

L'application terminale:
- récupère depuis l'entrée de répertoire accessible par `(s1, s2)`,
  - `fp` : le _profil_ de l'utilisateur crypté par la clé `Kp` de l'utilisateur. Disposant de `s1 s2`, l'application terminale,
    - obtient `Kp`,
    - décrypte le profil avec `Kp`.
  - `ckp` : le couple des 2 cryptages de la clé `Kp` de l'utilisateur par respectivement les deux clés `(s1 + s2)`, la principale et celle de secours.
- génère aléatoirement une clé `Ka` qui est identifiante de l'alias.
- enregistre une entrée dans le _profil_:
  - `aliasid` : identifiant : le SHA de `Ka`. Soit `x` le _SH(code PIN allongé, `Ka`)_.
  - `shax` : le SHA de `x`.
  - `kpx` : le cryptage de `Kp` par `x`.
  - `err` : 0. Nombre de tentatives infructueuses d'accès au code PIN.
  - `lm` : dernier mois d'accès, en l'occurrence le mois de création.

In fine l'application terminale dispose en mémoire du _profil_ en clair de l'utilisateur, obtenue par la saisie de `(s1 s2)`.

#### Ouverture d'une application par un code PIN
L'utilisateur saisit son code PIN et désigne son _alias_ dans la liste des utilisateurs habituels de l'appareil obtenue en lisant la base locale des alias.

L'application terminale:
- dispose,
  - du code de l'alias, 
  - du code PIN, 
  - de la clé `Ka` associée.
  - de `x`, le _cryptage du code PIN (allongé) par `Ka`_
- soumet une requête au serveur avec en arguments: `sha(x)` et `aliasid`: le `sha(Ka)`:
  - la requête lit l'enregistrement `a` par l'index `aliasid` (sa clé primaire est `userid.aliasid`).
  - enregistre dans `a.lm` le mois courant (si sa valeur a changé).
  - compare `a.shax` et `sha(x)` reçu en argument:
    - en cas d'inégalité, incrémente le compteur d'erreur `a.err` et s'il est supérieur à 1 supprime l'entrée `a` du _profil_ .
    - en cas d'égalité, retourne le _profil_ (crypté) et `a.kpx`.
- obtient `Kp` en décryptant `a.kpx` par `x`.
- décrypte le profil par `Kp` et en extrait `ckp`.
- stocke dans la base locale des alias de l'appareil, dans l'entrée correspondante de l'alias de l'utilisateur, `fp` (crypté) et `ckp` (ce qui les met à jour).

> **Remarque**: la clé `Kp` n'a jamais été disponible en clair dans un serveur.

In fine l'application terminale dispose en mémoire du _profil_ en clair de l'utilisateur, obtenue par la saisie du code PIN.

L'utilisateur et l'application terminale se retrouvent dans les mêmes conditions que si l'utilisateur avait fourni un couple de clés longues `s1 s2`, le _profil_ est en clair en mémoire. Depuis un appareil favori, l'utilisateur a seulement saisi un code PIN plus court que `s1, s2` et désigné un alias local.

> Le code PIN ne peut jamais être décrypté, ni avec seulement les données du _répertoire des alias_, ni seulement avec les données de la base locale de l'appareil.

> L'alias est local et sert seulement à l'utilisateur à retrouver facilement sa ligne dans la liste présentée à so choix. Son texte peut être n'importe quoi.

#### Sécurité de l'accès par _alias / code PIN_ sur un appareil favori
L'entrée dans un _profil_ étant détruite par le serveur au second échec, aucune attaque par force brute n'est possible à distance.

Les attaques possibles restent celles effectuées, depuis l'appareil, depuis le serveur ou depuis les deux conjointement.

##### Attaque depuis l'appareil
Le code PIN **N'EST PAS** stocké localement sur l'appareil: un voleur / hacker ne peut donc pas le retrouver. Le code PIN n'est présent que:
- en clair dans la tête de l'utilisateur (qui certes doit éviter de l'inscrire au feutre sur son appareil),
- dans son profil dans le _répertoire des profils_ par le sha de son cryptage par la clé `Ka`.

> Le seul moyen d'attaque serait de casser un des deux couples de codes `(s1 s2)`, ce qui est impossible si `s1` et `s2` sont à peu près bien choisis. L'existence d'un code PIN ne fragilise pas l'attaque depuis un appareil.

##### Par attaque depuis le serveur
L'administrateur du serveur protège l'accès à la base de données contenant le _répertoire des profils_. Les données sont cryptées par une clé d'administration. Pour _décrypter les enregistrements de la base_ il faut,
- a) avoir un accès en lecture à la base: remarque, le prestataire hébergeur de la base de données l'a.
- b) avoir la clé de cryptage de l'administrateur: remarque, le prestataire hébergeur de la base de données ne l'a pas.

En supposant que l'administrateur de la base de données dispose aussi de la clé de cryptage des données dans la base, le hacker peut tester par force brute des codes PIN `px`:
- calcul de `x` son SH(`px`, `Ka`) et vérification que `x` décrypte `kpx`.
- mais il n'a pas `Ka`, clé de 32 bytes tirée aléatoirement, donc inatteignable par force brute.

##### Par emprunt de l'appareil déverrouillé + complicité de l'administrateur
Cette fois la clé `Ka` est accessible, dans le debug de l'application terminale en exécution sur l'appareil. 

Le hacker peut tester par force brute des codes PIN `px`:
- calcul de `x` son SH(`px`, `Ka`) et vérification que `x` décrypte `kpx`.

Avec un code PIN `1234` et autres vedettes des mots de passe friables, l'effort ne devrait pas durer longtemps.

Toutefois UN SEUL essai d'un code demande un temps calcul important, le Strong Hash n'est _strong_ que parce qu'il exige du temps calcul non parallélisable et inapte à bénéficier de processeurs dits _graphiques_.

Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il est quasi incassable dans des délais humains: pour être mnémotechnique il va certes s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. mais il y a N façons de saisir `allons enfants de la pa`, avec ou sans séparateurs, des chiffres au milieu, des alternances de minuscules / majuscules. Il est difficilement concevable de coder l'inventivité des variantes, sans compter le nombre énorme de variantes possibles à exécuter à partir d'une seule _idée_ de texte de longueur inconnue.

Pour casser un code PIN sur un appareil favori, un hacker doit:
- connaître le login / mot de passe de l'appareil,
- l'emprunter et y lancer l'application pour en obtenir les `Ka`.
- avoir la complicité de l'administration technique du serveur,
- avoir de gros moyens informatiques.

Ces conditions constituent un handicap sérieux ... et demandent beaucoup d'argent et / ou l'usage de la force physique sur des humains. Si cette option est envisageable, il est moins coûteux de _persuader_ l'utilisateur de donner son code PIN (tant qu'il est vivant).

> SI l'hypothèse d'une collusion possible entre, les administrateurs ET des voleurs capables de dérober un appareil et d'y ouvrir une session, est considérée comme plausible, **soit** il ne faut pas déclarer d'appareils favoris, renoncer au mode avion et alourdir ses sessions sur l'appareil, **soit** il faut choisir un code PIN _dur_ à plus de 15 signes (ce qui reste vivable) qui sera incassable.

## Bases de données locale _cache_ sur un poste personnel
Sur un poste _favori_, le profil `Bob sur le mobile d'Alice` détient une petite base de données locale **par application** portant le nom de l'application suivi d'un hash de la clé `Ka`. Elle contient:
- des copies (forcément retardées par principe) de documents de l'application.
- des copies également retardées de _fils de documents_.
- trois index définissent à quels fils, chaque document est attaché.
- les contenus des documents et des fils sont cryptés par la clé `Kp` de l'utilisateur.

Quand une application est lancée elle va déterminer en fonction du souhait de l'utilisateur sur la page d'accueil, quels _fils de documents_ contiennent les documents à charger en mémoire:
- pour chacun l'application lit le contenu du fil détenu dans la base locale et demande au serveur de lui retourner le dernier état s'il est plus récent que celui obtenu de la base locale.
- l'application peut ainsi,
  - a) charger depuis la base locale les documents actuellement déclarés attachés au fil,
  - b) si nécessaire au vu des versions respectives, demander au serveur tous les documents attachés à ce fil de version postérieure.
  - c) mette à jour dans la base locale, les documents et le fil.

En effectuant cette opération pour tous les fils constituant le contexte de travail de la session, l'application,
- a) dispose en mémoire des fils nécessaires et des documents attachés,
- b) a mis à jour la base de données locales, qui pour cette session ouverte par l'utilisateur, est à jour.

## Le mode _avion_
Il est possible sur un poste _favori_ où l'utilisateur a ouvert récemment l'application et accédé à une de ses sessions favorites. Dans le _use-case circuitscourts_, par exemple pour un _consommateur_ ou le responsable des livraisons d'un groupement authentifiés par un identifiant et une clé d'autorisation (mot de passe pour simplifier).

La base de données locale d'une application contient les _fils de document_ du contexte fixé par l'utilisateur et les documents attachés: ils ne sont pas du tout dernier état mais a minima dans l'état où ils ont été accédés la dernière fois sur ce appareil.

La base de données est cryptée par la clé `Kp` et l'application doit se la procurer:
- l'accès par un code PIN est impossible, il n'y a pas de réseau pour obtenir la clé `Kp` cryptée par la clé `Ka` lisible localement.
- l'application demande à l'utilisateur de saisir un de ses couples d'accès `(s1, s2)` et peut ainsi obtenir `Kp` depuis la base locale des alias.

# Les _activités_ définies dans une application

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


-----------------------------------------------------------------------

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

# Décompter les consommations

Une application peut être gratuite pour certains de ses utilisateurs (voire tous) et payante pour d'autres selon l'usage qu'ils en font. 

Mais même dans le cas où l'application est toujours gratuite pour tous, il peut être cependant nécessaire de décompter des unités d'utilisation afin le cas échéant de les limiter.

La mise en œuvre de **contrats** permet,
- d'effectuer des décomptes d'usage par des _utilisateurs_, 
- d'en gérer des limites,
- si souhaité d'établir des _factures / paiements / régularisations_.

## Contrats et utilisateurs

Un _utilisateur_ est identifié par l'application et tout utilisateur est rattaché à un instant donné à un _contrat_. Au cours de sa vie un _utilisateur_ peut être détaché de son contrat précédent et être rattaché à un autre.

Un _contrat_ est un document d'identifiant aléatoire généré à sa création visant à suivre et gérer les consommations des utilisateurs qui lui sont rattachés: les consommations sont décomptées pour le mois _courant_ et le mois précédent (immuable par principe).

> Dans le cas général un _contrat_ concerne un _petit collectif d'utilisateurs_ une famille, une équipe, une petite association ... mais éventuellement un seul utilisateur.

Le décompte des consommations peut entraîner des contraintes, par exemple:
- rejet d'opération pour dépassement de certains seuils,
- ralentissement en cas de dépassement,
- blocage éventuel de certaines opérations pour certains utilisateurs.

## Unités d'œuvre

Chaque application déclare ses _unités d'œuvre_ selon deux catégories: les unités de _stock_ et les unités de _calcul / travail_.

Chaque unité a ses compteurs spécifiques: elles ne se moyennent ni se s'additionnent entre elles. Définir si c'est nécessaire des unités génériques englobantes.

Les unités sont décomptées,
- pour chaque _utilisateur_.
- globalement pour le _contrat_ (somme des valeurs pour chaque utilisateur).

### Unités de _stock_
Leur existence a un coût par le seul effet du temps qui passe. Par exemple:
- S1: nombre de commandes en cours.
- S2: volume de fichiers stockés.

Elles sont décomptées par _mois d'existence_, mais leur nombre pouvant varier à tout instant une moyenne pondérée du temps passé (à la milliseconde) est calculée. _Exemple_: 
- 30 commandes en cours pendant 10 jours (300),
- puis 10 commandes en cours pendant 20 jours (200),
- correspondent à 16,6 commandes par mois ((300 + 200) / 30).

### Unités de _calcul / travail_
Elles _coûtent_ effectivement lorsqu'elles sont sollicitées. Par exemple:
- C3: nombre de lecture de documents D1 et D2.
- C4: volume en Mo des fichiers de type F1 téléchargés.

Décompter ces unités sur un mois (par exemple le volume des fichiers téléchargés) revient à faire leur **somme** sur un mois (somme des téléchargements exécutés dans le mois).

## Forfaits: profil et niveau

Chaque application définit quelques **profils** de consommation (A à D par exemple):
- chaque profil correspond à un nombre maximum d'unités de chaque type. Par exemple:
  - profil A: 100 unités de stock S1, 20 unités S2, 5 unités C4
  - profil B: 10 unités S1, 50 unités S2, 50 unités C4
- un _profil_ correspond à un usage typique de consommation pour un profil d'utilisateur.

Chaque application définit aussi plusieurs **niveaux** de forfaits (XS S M L XL XXL par exemple): chacun correspond,
- à un coût forfaitaire,
- à un coefficient multiplicateur, par exemple 1 pour XS, 16 pour L, 64 pour XL.

La conjonction _profil / niveau_ définit ainsi un nombre forfaitaire d'unités de chaque type. Par exemple:
- 800 unités C4 pour un forfait B/L.
- 1600 unités S1 pour un forfait A/L.

## Application d'un forfait à un contrat

Un _contrat_ a un forfait défini en _profil_ et _niveau_ qui peuvent changer au cours du mois:
- le _profil_ peut changer sans conséquence sur la facturation.
- le _niveau_ peut changer aussi: la facturation du mois est calculée sur le niveau le plus proche au dessus du niveau moyen pondéré par le temps passé à chaque niveau.

Un _contrat_ s'appliquant à un collectif de N membres, ce facteur multiplicatif est spécifié: par exemple 8*A/L

### Maximum pour chaque unité
Le forfait du contrat détermine un maximum pour chaque unité: par exemple pour le forfait 8*A/L, 12800 unités de stock S1 et 640 unités de calcul C4.

Pour les unités de _stock_:
- le maximum fixé ne peut pas être dépassé, l'opération demanderesse tombe en exception.
- sauf si l'opération est marquée _privilégiée_,
- sauf si le maximum est certes dépassé mais en baisse.
- le **niveau d'alerte** est le pourcentage de dépassement au delà de 80% (0 en deçà).

Pour les unités de _calcul_:
- le nombre d'unité pris en compte est pris sur M et M-1:
  - le 10 du mois, le compte pour M est affecté du coefficient 1/3, celui de M-1 pour 2/3.
- le **niveau d'alerte** est le pourcentage de dépassement au delà de 80% (0 en deçà).
- l'application calcule une **durée de ralentissement de l'opération** (d'attente) fonction du niveau d'alerte afin de freiner l'excès de calcul, voire de bloquer l'opération (si elle n'est pas _privilégiée).

### Au niveau de chaque _utilisateur_
Un forfait _profil / niveau_ est également fixé par _utilisateur_, par exemple A/L: l'action de blocage / ralentissement et le niveau d'alerte sont les plus restrictifs des deux _utilisateur / contrat_.
- en l'occurrence par exemple pour S1 1600 pour _l'utilisateur_ OU 12800 pour le _contrat_.

## Facturation d'un contrat
Le décompte est établi en unités ou fractions d'unités d'œuvre pour toutes les unités effectivement consommées dans le mois courant et le mois précédent: il est disponible,
- par utilisateur,
- par sommation, globalement pour le contrat.

La facturation du mois courant,
- ne concerne globalement **que** le contrat,
- est établi en _unité monétaire_,
- est basée uniquement sur **le niveau _moyen_ de forfait** pour le mois (qui peut évoluer au cours du mois). Si compte tenu des changements en cours de mois, il apparaît que le _niveau de forfait moyen_ est L, la somme correspondante dans le tarif pour L est appliquée. Si le niveau est constant dans le mois, c'est celui-ci qui s'applique.

Le dernier mois facturé est conservé. L'application a ainsi le moyen d'archiver les factures sur la profondeur d'historique de son choix avec un format adapté au traitement statistique.

> En cours de mois la _facture est provisoire_, elle correspond au niveau de forfait moyen depuis le début du mois.

## Tarifs monétisant chaque niveau

L'application est maîtresse de sa politique de _monétarisation_ de chaque niveau de forfait en définissant un tarif monétaire (dans l'unité monétaire de son choix) en face de chaque niveau sous la forme (m1 N m2)
- N : niveau pivot: en-dessous de N appliquer m1 sinon m2.

Le tarif "Standard" est obligatoire et ne comporte qu'un montant m1.

L'application fixe les règles pour chaque contrat du choix libre ou contraint du niveau de contrat et du tarif applicable.
Ainsi une application peut par exemple:
- laisser libre pour ses adhérents identifiés la création de contrat "Gratuit": (0 M 5c) un tarif à 0 pour les niveaux inférieurs ou égaux à M, et de 5c au-delà.
- laisser un _Administrateur_ créer un par un des contrats "Contrôlé": un tarif à 0 pour des niveaux inférieurs ou égaux à M et un tarif moyen pour les niveaux supérieurs.
- laisser libre la création de contrats "Standard", un tarif normal sans limite de niveau.

## Débits et crédits, solde
La _facture_ d'un mois comporte une liste de _débits / crédits_: chacun a,
- un _code_ qui indique sa nature: par exemple _paiement reçu_,
- une _référence_ ou _commentaire_ permettant d'en savoir plus dans l'application.
- un montant positif ou négatif.

La première ligne est un _débit_ (possiblement à 0) correspondant à la facturation du forfait.

Le _solde_ en fin de mois correspond au solde en fin du mois précédent, plus les crédits, moins les débits.

En cours de mois un _solde provisoire_ est calculable simplement:
- depuis le solde au mois précédent,
- le niveau de forfait et le tarif applicable au contrat dans le mois,
- la balance des débits / crédits du mois.

### Politique vis à vis d'un solde négatif
L'application fixe sa politique vis à vis d'un _contrat_ présentant un solde _provisoire_ négatif. Par exemple:
- tolérance si le solde en début de mois était positif,
- tolérance pour les opérations _privilégiés_,
- alerte selon le nombre de jours estimés avec _solde positif_: simple calcul du nombre de jours pendant lequel le solde restera positif si le niveau du forfait reste inchangé.
- rejet de l'opération.

## Authentification d'utilisateurs via leur contrat

Tout utilisateur rattaché à un contrat _peut_ y être associé à une ou plusieurs **passphrases** qui l'authentifient: 
- une _passphrase_ donnée ne peut être associée qu'à un seul contrat à un instant donné et à un seul _utilisateur_ dans le contrat.
- dans son contrat, un utilisateur peut avoir _plusieurs passphrases_ associées lui permettant autant de solutions d'authentification.

> Il est en conséquence possible en une seule action d'authentifier un _utilisateur_ d'identifiant inconnu (mais en fournissant une de ses passphrases) et d'obtenir son _contrat_.

Hormis le cas ci-dessus, on accède à un contrat par son identifiant et on peut y retrouver tous ses utilisateurs enregistrés par leur identifiant.

### Suivi des consommations dans les opérations

Sauf exception toute opération du serveur a un _contrat par défaut_ à qui imputer ses consommations de ressources. 

Au cours de l'opération, d'autres _contrats_ peuvent être cités à qui imputer certaines consommations spécifiques selon la logique de l'application. Par exemple le stockage et l'utilisation de documents partagés par un **groupe** d'utilisateurs peut être imputé au _contrat du groupe_ plutôt qu'aux _contrats_ des utilisateurs eux-mêmes.

### Remarque: début et fin d'opération
Le début d'une opération va, en général, 
- lire le _contrat_ depuis la base et authentifier l'utilisateur demandeur de l'opération,
- en cas de basculement sur le mois suivant, calculer la facturation et repositionner les compteurs.

En cours d'opération, les compteurs du mois courant sont mis à jour (si nécessaire) et le _solde provisoire_ peut être réévalué.

La fin d'une opération va, en général, enregistrer en base la mise à jour du _contrat_.

Un _contrat_ a un mois de dernière mise à jour: un traitement périodique à partir du N d'un mois effectue a minima une opération sur chaque contrat dont le dernier mois de mise à jour 
n'est pas le mois courant afin d'éviter de perdre des facturations sur les _contrats_ peu utilisés.

Un traitement périodique peut aussi collecter un historique sous forme de fichier CSV pour analyse externe.
