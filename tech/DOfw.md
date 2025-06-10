---
layout: page
title: Réflexions à propos d'un framework "Orienté document"
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

# Les _applications_, leurs magasins, leurs variantes
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

# Documents, fichiers et _fils_ traçant leurs évolutions

## Document
Selon le standard JSON:
- deux **structures** de données sont utilisables:
  - des listes ordonnées,
  - des maps, associations _clé (string) / valeur_.
  - chaque terme d'une structure peut être une structure ou une donnée primitive.
- les données primitives sont des _string, number, boolean_.

Un document est un agrégat de données structurées en JSON dont la racine est une map dont chaque nom / clé donne la valeur d'une propriété qui peut être une structure ou une donnée élémentaire.

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
Un _credential_ attaché à un fil est une petite structure de données gouvernant un droit à lire le fil et à s'y abonner. 

Les autorisations de création / mise à jour sont gérées par l'application selon des règles applicatives plus riches.

Chaque _type de fil_ est associé à un _type de credential_:
- les propriétés identifiantes du credential sont mentionnées comme propriétés identifiantes du type de fil.
- par exemple le fil `CMDGP` est identifié par `gp.livr`.
- son _credential_ associé sera par exemple `CREDGP` identifié par `gp`.
- très simplement pour accéder à un _fil_ d'une livraison d'un groupement, il faut avoir le _credential_ de ce groupement.

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
- **soit l'appareil qu'il s'apprête à utiliser est _le sien_**: un peu plus généralement c'est un appareil,
  - qu'il utilise régulièrement, que se soit le sien ou celui d'un proche,
  - c'est un appareil de confiance: il peut y laisser quelques informations cryptées et espérer raisonnablement les retrouver plus tard.
- **soit l'appareil qu'il s'apprête à utiliser ne lui est pas familier**, il est partagé par des utilisateurs inconnus comme au cyber-café ou est celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelques informations que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour raccourcir ses saisies.

### Appareil _favori_
C'est un appareil qui _peut_ être utilisé par d'autres que soi-même. Typiquement dans un cadre familial ou un couple, l'appareil n'est pas strictement _personnel_.

Le _login_ à un tel appareil est _protégé_ par un mot de passe ou tout autre dispositif que seuls des proches connaissent, à moins que l'appareil leur soit prêté déverrouillé. 

_Plusieurs personnes peuvent plus ou moins occasionnellement s'en servir_: d'où le principe d'y avoir possiblement plusieurs _contextes personnels_, chacun étant repéré par un libelle parlant, un _alias_ de l'utilisateur.
- un browser comme Firefox a une notion d'utilisateur: on peut basculer d'un utilisateur à un autre (sans pour autant avoir changé de connexion au niveau de l'OS). Chacun a ses sites favoris, son historique de navigation et ses mots de passe enregistrés.
- Thunderbird, le gestionnaire de mails locaux, supporte de gérer plusieurs _profils_, chacun avec ses comptes mails.

#### Des _contextes_ plus ou moins bien défendus
Ce n'est pas parce qu'on partage un appareil avec un proche qu'on a envie de partager avec lui ses informations confidentielles.

Or dans les cas cités ci-dessus, la confidentialité est plutôt _lâche_:
- Thunderbird ne demande rien: on choisit son profil, sans mot de passe ou quoi que ce soit. Les boîtes mail sont de toutes les façons en clair dans le file-system, confidentialité _intra-familiale_ zéro.
- Chrome s'ouvre sur le compte _courant_: si vous ne vous déconnectez pas **explicitement** avant de fermer le browser, il s'ouvre la fois suivante sur votre compte, ses mots de passe, ses historiques et ses favoris. Si vous vous déconnectez Chrome est strict sur la connexion et demandera même à votre téléphone si c'est vraiment vous qui essayez de vous connecter: il suffit d'y répondre OUI et c'est bon (même si vous vous êtes fait voler votre téléphone en état déverrouillé, Google est content).

#### Déclarer son ou ses _alias_ sur un appareil _favori_
Il est possible de déclarer un appareil comme _favori_ en fixant un _alias_ (par exemple `bob`) et un code PIN. Voir le détail plus avant.

Quelques informations _locales_ sont alors mémorisés sur l'appareil dans une micro base de données dont l'espace est gérée par le _browser_. Le contenu est crypté, illisible en _debug_ et  même l'accès (malaisé) par le _file-system_ de l'OS ne permet pas à un _hacker_ d'accéder à son contenu.

Pour un utilisateur lancer une application depuis un appareil _favori_ a plusieurs avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant de petites bases de données locales spécifiques du profil comme _cache_ de documents.
- **possibilité d'accéder en mode _avion_** sans accès au réseau.
- **identification plus rapide** mais sûr par usage d'un code PIN.

### Lancement d'une application sur un appareil _favori_
La liste des quelques profils enregistrés est présentée: l'utilisateur choisit le sien et donne son code PIN.

L'application ouvre sa page d'accueil où l'utilisateur peut choisir une de ses sessions _favorites_ ouvrant directement accès à ses documents grâce aux _credentials_ pré-enregistrés.

En fermant l'application, l'utilisateur peut choisir:
- de laisser ses _fils de news_ activés: des notifications apparaîtront, même quand l'application sera fermée et même si ce n'est plus le même utilisateur qui dispose de l'appareil. Les _notifications_ ne font qu'annoncer des changements sans en donner les détails et jamais d'informations confidentielles. 
- de fermer ses _fils de news_: aucune notification ne parviendra plus sur l'appareil relativement à cette application. L'utilisateur peut le prêter à quelqu'un d'autre sans risque ... mais lui-même ne recevra plus de notifications (il faut choisir).

### Lancement d'une application EN MODE AVION sur un appareil _favori_
La liste des quelques profils enregistrés est présentée: l'utilisateur choisit le sien et donne l'une de ses deux phrases longues d'accès (et non pas son code PIN).

L'application ouvre sa page d'accueil où l'utilisateur peut choisir une de ses sessions _favorites_ ouvrant directement accès à ses documents enregistrés au cours d'une session antérieure pas en mode _avion_.

En mode avion, il n'y a pas de réseau, pas de _fils de news_: les dossiers peuvent être consultés mais pas mis à jour. L'utilisateur peut saisir des textes ou formulaires purement locaux et stocker des fichiers (comme des photos prises en mode avion): toutes ces informations sont cryptées dans le stockage local et pourront être utilisées pour mettre à jour des documents quand le réseau sera à nouveau disponible en sécurité.

### Lancement d'une application sur un appareil _anonyme_
Si l'utilisateur s'est enregistré, il donne l'une de ses deux phrases longues d'accès. L'application ouvre sa page d'accueil où l'utilisateur peut choisir une de ses sessions _favorites_ ouvrant directement accès à ses documents grâce aux _credentials_ pré-enregistrés.

Si l'utilisateur n'est pas enregistré, l'application propose un dialogue d'accueil où il devra indiquer:
- son intention, avec quel rôle il souhaite accéder aux documents,
- quels documents il souhaite accéder et fournit le ou les _creddentials_ associés.
- le cas échéant il choisira les _fils de news_ qui l'intéresse et donnera le cas échant les _credentials_ associés.

> Pour l'application, un utilisateur _non enregistré_ a les mêmes possibilités qu'un utilisateur enregistré. S'il n'est pas enregistré il doit fournir plus d'informations pour faire valoir ses droits d'accès, bref les données qui justement sont pré-remplies pour une session déclarée _favorite_ pour l'utilisateur.

La fermeture de l'application ne laisse pas le choix sur un appareil _anonyme_: les abonnements aux _fils de news_ sont tous supprimés, aucune notification ne parviendra plus sur cet appareil résultant de l'usage précédent de l'application.

### Mode _veille_
Les applications ouvertes ont un mode _veille_ optionnel: s'il est activé, l'application se met en veille en cas de non utilisation pendant quelques minutes (fixés selon le degré de paranoïa de l'utilisateur). Pour sortir de la veille un code est nécessaire et au second échec l'application se ferme.

# Données d'un service
Un _service donné pour un prestataire donné_ dispose de deux entités de stockage dédiées:
- **UNE Base de données** pour gérer les documents et les fils de documents selon un mode _transactionnel_ (ACID).
- **UN Storage de fichiers**, non géré par des transactions mais dont la sécurité transactionnelle peut être assurée par la base de données avec un protocole léger pré-validation / validation. Il stocke des _fichiers_ identifiés par leur _path_: la présence de caractères `/` dans un path définit une sorte d'arborescence. Le _contenu_ de chaque fichier est une suite d'octets opaque.

Le Storage permet de disposer d'un volume pratiquement 10 fois plus importants à coût identique par rapport à la base de données: de nombreuses applications ont des données historiques / mortes ou d'évolutions sporadiques qui s’accommodent bien d'un support sur Storage.

> La Base et le Storage sont gérées et accédées exclusivement par le prestataire du service.

#### Le répertoire des applications terminales _abonnées_
Chaque application sur un appareil a un _token_ qui l'identifie de manière unique. Ce répertoire contient les _abonnements_ en cours des applications.
- Le répertoire détecte les applications n'ayant pas été lancées depuis un certain temps.
- pour chaque application le répertoire conserve la liste des abonnements en cours aux _fils de documents_.

Quand un document évolue, le répertoire retrouve toutes les applications abonnées à un _fil_ auquel le document est attachés et effectue une publication de notifications vers elles.

> Chaque application terminale est en conséquence susceptible de s'abonner éventuellement auprès de plus d'un prestataire si toutes les organisations de son domaine d'intérêt ne sont pas toutes gérées par le même prestataire.

# Glossaire technique

- **SH(s1, s2)** (Strong Hash): le SH s'applique à un couple de textes `s1 s2`, typiquement un login / mot de passe, mais aussi aux _passphrase_ en une ou deux parties. Il a une logueur de 32 bytes et est unique pour chaque couple de textes `s1 s2`. Il est _strong_ parce qu'incassable par force brute dès lors que le couple de textes ne fait pas partie des _dictionnaires_ des codes fréquement utilisés.
- **PP-x** : couples de clés publique / privée (Pub / Priv).
- **K-x** : clés AES de 32 bytes.

Soit `PP-S` le couple de clés généré par un serveur:
- `Pub-S` est une clé publique: toutes les applications l'obtiennent libremenet par un simple GET au serveur.
- `Priv-S` est la clé privée du serveur et fait partie des _secrets_ du logiciel serveur.

Soit `PP-ti` un couple de clés généré par une application terminale `ti` pour une conversation donnée avec le serveur `S`:
- `Pub-ti` est transmise dans les requêtes au serveur.
- Le serveur peut génèrer une clé `K-S-ti` à parir du couple `Pub-ti / Priv-S`: il s'en sert pour crypter ses réponses à l'application terminale ou toute donnée dont il veut que seule `ti` puisse les lire.
- L'application terminale peut générer une clé `K-ti-S` à partir du couple `Pub-S / Priv-ti`. Comme `K-ti-S` et `K-S-ti` sont égales, l'application terminale peut s'en servir pour décrypter les réponses / données cryptées pour elle par l'application serveur (et nul autre ne peut le faire).

Les applications terminales connaissent la clé publique du serveur Pub-S. Quand l'une d'elle veut échanger des données confidentielles avec le serveur:
- elle génère un couple PP-t,
- elle envoie ses requêtes au serveur en fournissant Pub-t.
- elle décryptera les réponses / données cryptées par le serveur par la clé Pub-S.

# Base de données partagée par tous les prestataires

**Cette base unique concerne toutes les applications et tous les prestataires.**
- Tous les prestataires peuvent y enregistrer les organisations dont ils traitent les données.
- Toutes les applications terminales peuvent solliciter un quelconque des prestataires afin d'accéder concernant un de leurs utilisateurs.

Les données de cette base sont:
- Le **répertoire des organisations par application**.
  - il liste pour toutes les applications le prestataire (son URL) gérant chaque organisation.
  - tous les prestataires y enregistrent les organisations dont ils traitent les données.
- Le **répertoire des utilisateurs**. Tout utilisateur qui s'y enregistre (c'est une facilité pas une obligation) y dispose d'une _fiche personnelle_ confidentielle et sécurisée gérée par les applications terminales. Il y a plusieurs rubriques dans la _fiche personnelle_ d'un utilisateur (voir plus avant):
  - des _préférences_,
  - des _credentials_,
  - des _sessions favorites_,
  - des _appareils favoris_.

> Un _credential_ est une petite donnée structurée qui renferme des données d'authentification comme un couple login / mot de passe (ou tout autre dispositif).

## Répertoire des _organisations par application_
Une application terminale détient la liste des _prestataires_ fournissant les services centraux, leur _code_ et leur _URL d'accès_. 
- L'ajout ou le retrait d'un prestataire provoque une nouvelle version de l'application dont l'installation est automatique.

Une session d'une application terminale peut concerner plusieurs organisations, à l'instar du randonneur faisant partie de plusieurs associations selon l'endroit où il randonne.

L'application terminale a en conséquence besoin de savoir pour chaque organisation concernée à quel prestataire (URL) il doit faire appel.

Ce répertoire contient la liste des triplets `{ application, prestataire, organisation }` déclarés par les prestataires:
- ils peuvent en exporter des listes sélectives, en particulier pour une application / prestataire, la liste des organisations gérées. 
- une application terminale peut faire appel à n'importe quel prestataire afin de récupérer le prestataire traitant une organisation donnée

**Remarque** : chaque prestataire peut ensuite gérer **dans _sa_ base de données**, un document relatif à l'organisation comportant:
- un **statut** : est-elle ouverte, restreinte en lecture seule (archive), fermée jusqu'à nouvel ordre.
- une **courte liste de _news_** données par l'administrateur.
- les applications terminales peuvent s'abonner aux modifications de ce document.

## Répertoire des _fiches personnelles_ des utilisateurs
Chaque utilisateur **_PEUT_**, et non _DOIT_, s'enregistrer dans ce répertoire et y disposer de sa _fiche personnelle_ afin de raccourcir les saisies d'information, dont les _credentials_ à fournir pour accéder aux applications de son choix. Il y est identifié par un USERID généré aléatoirement et sans signification.

Une fiche personnelle est cryptée dans la base de données de sorte que seul l'utilisateur puisse la lire par l'intermédiaire des applications terminales qu'il sollicite en leur donnant (indirectement) la clé de décryptage. 
- Ces applications n'exportent jamais le texte en clair d'une fiche sur le réseau.
- _L'impression_ de cette fiche (en clair) peut être toutefois explicitement demandée par l'utilisateur qui est alors responsable de ce qu'il en fait.

### Les applications peuvent-elles connaître le USERID ?
Une application terminale bien entendu, **à condition** que l'utilisateur se soit enregistré.
- elle peut communiquer dans un _credential_ ce USERID à un serveur accompagné du `SH(s1, s2)` prouvant que l'utilisateur a bien fourni cette clé d'accès `(s1, s2)`.
- à la première présentation de ce USERID, typiquement à l'enregistrement de l'utilisateur, le serveur peut vérifier auprès du répertoire des utilisateurs que ce USERID est bien authentifié et va conserver le couple `(USERID, sha(SH(s1, s2))` dans la base de l'application.
- les authentifications ultérieures se feront seulement à partir de la base locale.

Les applications **peuvent** en conséquence utiliser le répertoire des utilisateurs pour les authentifier **MAIS** ceci oblige les utilisateurs à s'enregistrer dans ce répertoire pour utiliser l'application ce qui pose un problème déontologique: les applications peuvent établir des liens logiques entre elles.

Une solution consiste à permettre à un  utilisateur d'une application qui souhaite disposer d'un USERID,
- soit d'en créer un spécifique de l'application avec une authentification spécifique,
- soit, au choix de l'utilisateur, d'utiliser son USERID du répertoire des utilisateurs, externe aux applications.

### Rubriques d'une _fiche personnelle_
Il y a plusieurs rubriques dans la _fiche personnelle_ d'un utilisateur:
- des _préférences_,
- des _credentials_,
- des _sessions favorites_,
- des _appareils favoris_.

### Préférences
Une _préférence_ est une donnée nommée pour laquelle l'utilisateur a donné une ou des valeurs par défaut / préférées:
- langue préférée,
- mode sombre / clair,
- nom, e-mail, adresses, numéros de téléphone ...
- etc.

Quand une application a besoin de l'une de ces informations, elle propose à l'utilisateur en pré-saisie la ou l'une des valeurs inscrites en _préférences_ si elle y en a. L'utilisateur peut en sélectionner une ou en saisir une autre (à enregistrer ou non en préférence).

Deux préférences par défaut sont toujours conservées: les deux couples de phrases `(s1, s2)`, habituelle et de secours, qui ouvrent à l'utilisateur l'accès à sa _fiche personnelle_.

> A noter que pour _lire_ ces phrases (voire les modifier), il faut en avoir fourni une des deux.

### _Credentials_: droits d'accès
Un _credential_ est un droit d'accès pour lire / agir sur des données d'une application. Le type le plus standard de _credential_ est un couple _login / mot de passe_, mais des formes plus sophistiquées existent _passphrase_ ... (voir le chapitre sur l'authentification _double_).

Pour une application donnée, il existe plusieurs **types** de _credential_, conférant chacun des natures d'autorisations différentes. Par exemple pour l'application "circuitscourts",
- le **type `PL`** est le _credential_ d'un _point-de-livraison_:
  - il est identifié par le code `gc` d'un point-de-livraison.
  - il est unique, il n'y a qu'un credential reconnu par point.
- **le type `CO`** est le _credential_ d'un _consommateur_:
  - il est identifié par `gc co` identifiant un consommateur attaché à un point-de-livraison.
  - il est **multiple**: pour un `gc co`. Il y est défini une valeur de  _credential_ associée aux initiales du membre de la famille du consommateur. Chacun a par commodité son propre mot de passe de manière à pouvoir le cas échéant en bloquer facilement un sans bloquer les autres, ou plus simplement avoir une valeur courante et une de secours en cas d'oubli de la première.

**Le _jeton_ associé à un _credential_** est un texte généralement assez opaque, typiquement d'un texte `SH(s1, s2)` ou d'un mot de passe `SH(mp, mp)`. Ce sont les serveurs qui sont en charge de vérifier la validité d'un _credential_, par exemple en comparant la valeur fournie par l'application terminale avec son SHA enregistré en base de données.

> Sauf rares exceptions, une opération d'un serveur exige un, voire plusieurs, _credentials_ qui sont vérifiés avant de traiter effectivement l'opération.

Un _credential_ peut être enregistré dans la _fiche personnelle_ de l'utilisateur avec un libellé court, connu seulement de l'utilisateur: `mon accès conso à JP`. Énigmatique dans l'absolu, ce texte est signifiant pour l'utilisateur lui donnant accès à ses documents de _consommateur_ dans le cadre d'un point-de-livraison qui lui est familier (plus que le code aléatoire correspondant): il a saisi, son code de point de livraison, son code de consommateur, ses initiales (ou rien) et le mot de passe.

> L'enregistrement des _credentials_ d'un utilisateur dans sa _fiche personnelle_lui permet de simplement cliquer dans une courte liste pour le fournir plutôt que d'avoir à se rappeler et à saisir un mot de passe long: c'est l'accès à sa _fiche personnelle_ qui est sécurisée pour un utilisateur, ceci protégeant **tous** ses _credentials_ enregistrés.

### _Sessions favorites_ d'un utilisateur
Lorsqu'un utilisateur ouvre une application terminale il commence une _session_: en général il ne peut pas faire grand-chose avant d'avoir déclaré a minima, 
- a) son intention, qu'est-ce qu'il veut y faire, 
- b) un _credential_ démontrant son droit à accéder aux documents et aux actions associées.

Un utilisateur peut aussi débuter une session avec plus de droits et un périmètre d'action plus large au cours de laquelle il aura:
- a) un accès _consommateur_ dans le point-de-livraison où il est enregistré.
- b) deux accès _point-de-livraison_ pour les deux organisations où il intervient pour aider à gérer des livraisons.
- c) un accès _groupement_ parce qu'il est également l'assistant d'un groupement de producteur qui n'est pas autonome.

> Exactement comme un utilisateur de Discord peut avoir accès à plusieurs _serveurs_ pour autant de sujets d'intérêt.

#### Auto-enregistrement d'un utilisateur
Lorsqu'un utilisateur a ouvert une session sans l'avoir sélectionnée depuis sa _fiche personnelle_, typiquement parce qu'il n'en n'a pas encore, l'application lui propose de s'enregistrer. Si l'accepte il saisit:
- les deux couples `s1 s2` de clés d'accès (un habituel et un de secours).
- il donne un libellé à sa session actuelle.

Sa _fiche personnelle_ est créée, 
- elle est cryptée, 
- ses préférences sont initialisées avec ses clés d'accès, 
- le ou les credentials de sa session sont enregistrés, 
- les paramètres de sa session sont enregistrée en tant que première session favorite.

En cours d'une session, l'utilisateur peut ouvrir de nouveaux accès pour de nouveaux rôles après avoir fourni le cas échéant de nouveaux _credentials_. Les paramètres de l'état courant de la session peuvent être enregistrés comme _session favorite_, une nouvelle ou en remplaçant une antérieure.

Lors d'une prochaine ouverture de l'application l'utilisateur, après avoir donné une de ses clés d'accès à sa fiche personnelle, n'a plus qu'à cliquer sur l'une de ses sessions favorites pour avoir à disposition tous les documents correspondants sans avoir eu à en citer d'identifiants ni à fournir de credentials.

> Au lieu d'une logique organisée autour de l'identification de _personnes_ (plus ou moins virtuelles), c'est l'utilisateur qui sélectionne une _session_ où il a plusieurs rôles. Chaque _ensemble de  documents et actions associés_ est comme enfermé dans un coffre, tout utilisateur en connaissant la combinaison peut prétendre y accéder avec la possibilité de gérer plusieurs combinaisons par coffres.

#### Remarques techniques
- l'application terminale lors de l'enregistrement d'un utilisateur allonge `s1` en `s1+` par un texte de remplissage quand sa longueur est inférieure au maximum mais supérieure au minimum (refus si inférieur à la longueur minimale). L'unicité du `SH(s1+, s1+)` est vérifiée.
- l'application terminale d'enregistrement allonge `s2` en `s2+` de même, mais le texte de remplissage est généré en fonction de `s1`.
- l'unicité de `s1` est vérifiée par l'enregistrement du `SHA(SH(s1+, s1+))`.
- l'unicité de `s1 s2` est vérifiée par l'enregistrement du `SHA(SH(s1+, s2+))`.

#### Accès par l'utilisateur à sa _fiche personnelle_
Quand un utilisateur est enregistré, l'ouverture d'une application lui propose de démarrer une des sessions favorites listées dans sa fiche personnelle: 
- lui demande l'un de ses couples d'accès `s1 s2`. L'application en construit les couples `SH(s1+, s1+)` et `SH(s1+, s2+)`. 
- le serveur peut par `SHA(SH(s1+, s1+))` accéder à l'entrée `USERID` pour cet utilisateur et vérifier la validité de `SH(s1+, s2+)` pour ce `USERID`. Il peut retourner,
  - la clé `Kp` cryptée par `s1 + s2` (et qu'il est incapable de décrypter faute de connaître `s1` et `s2`).
  - le set des _préférences_ de l'utilisateur crypté par `Kp`.
  - la liste des _sessions favorites_ et des _credentials_ enregistrés cryptée par `Kp`.

> Quand il n'a pas utilisé de session favorite enregistrée, l'utilisateur peut créer sa session depuis une des _sessions template_ codée de l'application, en fournir les paramètres (le code de son point-de-livraison, son code consommateur) et indiquer quel credential de sa liste il veut utiliser (ou le saisir, par exemple ses initiales et mot de passe). La session peut être enregistrée comme favorite pour une ouverture en un clic la prochaine fois.

L'application terminale disposant en mémoire de la fiche de l'utilisateur en _clair_ peut:
- mettre à jour les _préférences_ de l'utilisateur et les enregistrer cryptées par la clé `Kp` qu'il vient de récupérer.
- pour chaque _session favorite_ et _credential_,
  - proposer de l'effacer le cas échéant,
  - adapter son _libellé_ facilitant le choix de l'utilisateur.

> L'utilisateur peut avoir plusieurs usages plus ou moins fréquents pour accéder à ses applications dans des circonstances diverses: l'utilisation de sa _fiche personnelle_ lui permet de n'avoir qu'un couple `s1 s2` à se souvenir. 

> Dans ce répertoire la fiche d'un utilisateur est anonyme, inconnu des GAFAM, ne contient aucune information personnelle, ni nom, ni adresse e-mail, ni numéro de mobile (sauf à les avoir volontairement saisies en _préférences_ mais elles sont cryptées). Son enregistrement dans ce répertoire est inviolable, pour autant que les couples `s1 s2` en clés principales et de secours, qu'il a choisi soient respectueux d'un minimum de règles simples.

### _Appareils favoris_ de l'utilisateur
Un appareil _de confiance / personnel_ est un appareil où l'utilisateur considère qu'il peut stocker des données locales permanentes.

Exécuter une application sur un _appareil favori_ de l'utilisateur présente des avantages significatifs:
- **réduction de l'usage du réseau comme du nombre d'accès à la base de données:** de nombreux documents peuvent être déjà présents dans la base de données locales de l'application. La mise à niveau de ceux-ci est _incrémentale_ ne chargeant que ceux ayant changé ou nouveaux.
- **possibilité d'avoir des sessions en mode _avion_**, sans aucun accès au réseau (voir plus loin les restrictions associées).
- **authentification rapide depuis un code PIN court** au lieu de devoir fournir une de ses deux clés longues `s1 s2` pour accéder à sa `fiche personnelle`.

#### Stockage local à l'appareil
Une **micro base locale des alias** stocke quelques données relatives aux utilisateurs ayant déclaré l'appareil comme _favori_. Elle est hébergée / gérée par le browser dans un espace spécifique du _domaine_ de l'application terminale.

La base a une table ayant une ligne par _alias_ d'utilisateur comportant:
- `alias` : l'alias choisi par un des utilisateurs.
- `ka` : une clé de 32 bytes générée à la création de l'entrée. Elle est cryptée _mollement_ par une clé détenue dans le source de l'application terminale (donc lisible en debug avec un peu de fatigue).
- `fp` : la _fiche personnelle_ de l'utilisateur cryptée par sa clé `Kp`.
- `ckp` : le couple de 2 cryptages de la clé `Kp` de l'utilisateur par respectivement les deux clés `(s1 + s2)`, la principale et celle de secours.

#### Déclarer un appareil comme favori
La liste des _alias_ des utilisateurs ayant utilisé cet appareil comme favori est présentée: l'utilisateur peut ainsi déterminer s'il doit,
- déclarer cet appareil comme favori.
- si c'était déjà fait changer son code PIN.
- supprimer les entrées des _alias_ qui ne l'inspirent pas.

Pour déclarer l'appareil comme favori ou refixer son code PIN, l'utilisateur doit fournir,
- l'alias de son choix, sélectionné dans la liste ou inventé à l'instant,
- **un code PIN d'au moins 8 signes**,
- une de ses deux clés longues `s1 s2` qui permet à l'application de retrouver sa fiche personnelle.

> Si l'alias avait déjà une entrée, l'application terminale va essayer le code PIN proposé: en cas d'échec, l'entrée de l'alias dans le _répertoire des alias_ est supprimée.

L'application terminale:
- récupère depuis l'entrée de répertoire accessible par `(s1, s2)`,
  - `fp` : la _fiche personnelle_ de l'utilisateur cryptée par la clé `Kp` de l'utilisateur. Disposant de s1 s2, l'application terminale,
    - obtient `Kp`,
    - décrypte la fiche personnelle avec `Kp`.
  - `userid` : de l'utilisateur.
  - `ckp` : le couple de 2 cryptages de la clé `Kp` de l'utilisateur par respectivement les deux clés `(s1 + s2)`, la principale et celle de secours.
- génère aléatoirement une clé `Ka` qui est identifiante de l'alias.
- enregistre une entrée dans le _répertoire des alias_:
  - `aliasid` : le SHA de `Ka`, clé d'accès dans ce répertoire.
  - `userid` : de l'utilisateur ayant déclaré cet alias.
  - soit `x` le _SH(code PIN allongé, `Ka`)_.
  - `shax` : le SHA de `x`.
  - `kpx` : le cryptage de `Kp` par `x`.
  - `err` : 0. Nombre de tentatives infructueuses d'accès au code PIN.
  - `lm` : dernier mois d'accès, en l'occurrence le mois de création.
enregistre dans la base locale des alias une entrée avec : `alias ka fp ckp`.

In fine l'application terminale dispose en mémoire de la _ficher personnelle_ en clair de l'utilisateur, obtenue par la saisie de `(s1 s2)`.

#### Ouverture d'une application sur un appareil déclaré _favori_
L'utilisateur saisit son code PIN et désigne son _alias_ dans la liste des utilisateurs habituels de l'appareil obtenue en lisant la base locale des alias.

L'application terminale:
- dispose,
  - du code de l'alias, 
  - du code PIN, 
  - de la clé `Ka` associée.
  - de `x`, le _cryptage du code PIN (allongé) par `Ka`_
- soumet une requête 1 au serveur avec en arguments: `sha(x)` et `aliasid`: le `sha(Ka)`:
  - lit l'entrée `a` du _répertoire des alias_ par sa clé `aliasid`.
  - enregistre dans `a.lm` le mois courant (si sa valeur a changé).
  - compare `a.shax` et `sha(x)` reçu en argument:
      - en cas d'inégalité, incrémente le compteur d'erreur `a.err` et s'il est supérieur à 1 supprime l'entrée `a` du _répertoire des alias_ .
      - en cas d'égalité, retourne `a.userid` et `a.kpx` 
- obtient `Kp` en décryptant `a.kpx` par `x`.
- soumet une requête 2 au serveur avec userid en argument et en récupère en retour `fp` la _fiche personnelle_ de l'utilisateur par sa clé `userid`.
- décrypte la fiche par `Kp` et en extrait `ckp`.
- stocke dans la base locale des alias de l'appareil, dans l'entrée correspondante de l'alias de l'utilisateur, `fp` (cryptée) et `ckp` (ce qui les met à jour).

> **Remarque**: la clé `Kp` n'a jamais été en clair dans un serveur.

In fine l'application terminale dispose en mémoire de la _ficher personnelle_ en clair de l'utilisateur, obtenue par la saisie du code PIN.

L'utilisateur et l'application terminale se retrouvent dans les mêmes conditions que si l'utilisateur avait fourni un couple de clés longues `s1, s2`, la fiche personnelle est en clair en mémoire. Depuis un appareil favori, l'utilisateur a seulement saisi un code PIN plus court que `(s1, s2)` et désigné un alias local.

> Le code PIN ne peut jamais être décrypté, ni avec seulement les données du _répertoire des alias_, ni seulement avec les données de la base locale de l'appareil.

> L'alias est local et sert seulement à l'utilisateur à retrouver facilement sa ligne dans la liste présentée à so choix. Son texte peut être n'importe quoi et être modifié à loisir.

#### Sécurité de l'accès par _alias / code PIN_ sur un appareil favori
L'entrée dans le _répertoire des alias_ étant détruite par le serveur au second échec, aucune attaque par force brute n'est possible à distance.

Les attaques possibles restent celles effectuées, depuis l'appareil, depuis le serveur ou depuis les deux conjointement.

##### Attaque depuis l'appareil
Le code PIN **N'EST PAS** stocké localement sur l'appareil: un voleur / hacker ne peut donc pas le retrouver. Le code PIN n'est présent que:
- en clair dans la tête de l'utilisateur (qui certes doit éviter de l'inscrire au feutre sur son appareil),
- par le sha de son cryptage par la clé Ka dans le _répertoire des alias_.

> Le seul moyen d'attaque serait de casser un des deux couples de codes `(s1 s2)`, ce qui est impossible si `s1` et `s2` sont à peu près bien choisis. L'existence d'un code PIN ne fragilise pas l'attaque depuis un appareil.

##### Par attaque depuis le serveur
L'administrateur du serveur protège l'accès à la base de données contenant les _répertoires des utilisateurs et des alias_. Les données de celle-ci sont cryptées par une clé d'administration. Pour _décrypter les enregistrements de la base_ il faut donc,
- a) avoir un accès en lecture à la base: remarque, le prestataire hébergeur de la base de données l'a.
- b) avoir la clé de cryptage de l'administrateur: remarque, le prestataire hébergeur de la base de données ne l'a pas.

En supposant que l'administrateur de la base de données dispose aussi de la clé de cryptage des données dans la base, le hacker peut tester par force brute des codes PIN `px`:
- calcul de `x` son SH(`px`, `Ka`) et vérification que `x` décrypte `kpx`.
- mais il n'a pas `Ka`, clé de 32 bytes tirée aléatoirement, donc inatteignable par force brute.

##### Par attaque conjointe: vol de l'appareil + complicité de l'administrateur
Cette fois la clé `Ka` est accessible, dans le debug de l'application terminale en exécution sur l'appareil. 

Le hacker peut tester par force brute des codes PIN `px`:
- calcul de `x` son SH(`px`, `Ka`) et vérification que `x` décrypte `kpx`.

Avec un code PIN `1234` et autres vedettes des mots de passe friables, l'effort ne devrait pas durer longtemps.

Toutefois UN SEUL essai d'un code demande un temps calcul important, le Strong Hash n'est _strong_ que parce qu'il exige du temps calcul non parallélisable et inapte à bénéficier de processeurs dits _graphiques_.

Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il est quasi incassable dans des délais humains: pour être mnémotechnique il va certes s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. mais il y a N façons de saisir `allons enfants de la pa`, avec ou sans séparateurs, des chiffres au milieu, des alternances de minuscules / majuscules. Il est difficilement concevable de coder l'inventivité des variantes, sans compter le nombre énorme de variantes possibles à exécuter à partir d'une seule _idée_ de texte de longueur inconnue.

En conséquence pour casser un code PIN sur un appareil favori, un hacker doit:
- connaître le login / mot de passe de l'appareil,
- l'emprunter et y lancer l'application pour en obtenir les `Ka`.
- avoir la complicité de l'administration technique du serveur,
- avoir de gros moyens informatiques.

Ces conditions constituent un handicap sérieux ... et demandent beaucoup d'argent et / ou l'usage de la force physique sur des humains. Si cette option est envisageable, il est moins coûteux de _persuader_ l'utilisateur de donner son code PIN (tant qu'il est vivant).

> SI l'hypothèse d'une collusion possible entre, les administrateurs ET des voleurs capables de dérober un appareil et d'y ouvrir une session, est considérée comme plausible, **soit** il ne faut pas déclarer d'appareils favoris, renoncer au mode avion et alourdir ses sessions sur l'appareil, **soit** il faut choisir un code PIN _dur_ à plus de 15 signes (ce qui reste vivable) qui sera incassable.

## Bases de données locale _cache_ sur un poste personnel
Sur un poste _favori_, le profil `bob` détient une petite base de données locale **par application** portant le nom de l'application suivi d'un hash de la clé `Ka`. Elle contient:
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

La base de données locale d'une application contient les _fils de document_ du contexte fixé par l'utilisateur et les documents attachés: certes ils ne sont pas du tout dernier état mais a minima dans l'état où ils ont été accédés la dernière fois sur ce appareil.

La base de données est cryptée par la clé `Kp` et l'application doit se la procurer:
- l'accès par un code PIN est impossible, il n'y a pas de réseau pour obtenir la clé `Kp` cryptée par la clé `Ka` lisible localement.
- l'application demande à l'utilisateur de saisir un de ses couples d'accès `(s1, s2)` et peut ainsi obtenir `Kp` depuis la base locale des alias.

# Les _activités_ définies dans une application


# Annexe: le Use Case _circuit court_

## Vision générale: les _documents_

Un **point-de-livraison regroupe des consommateurs** et est identifié par son code `gc`. 
- **Document FGC** : fiche de renseignement donnant des informations de contact, son ou ses mots de passe, liste des groupements de producteurs auxquels il peut commander. 

Un **consommateur** est identifié par son code dans son point-de-livraison: `gc co`. 
- **Document FCO** : fiche de renseignement donnant des informations de contact, son mot de passe, un statut de blocage.

Un **groupement de producteurs** est identifié par son code `gp`. 
- **Document FGP** : fiche de renseignement donnant des informations de contact, son ou ses mots de passe, liste des groupes de consommateurs qui peuvent lui émettre des commandes.

Un **producteur** est identifié par son code dans son groupement: `gp pr`. 
- **Document FPR** : fiche de renseignement donnant des informations de contact et son mot de passe.

Une **référence de produit** est identifiée par son code dans son producteur: `gp pr rp`.

Le **calendrier des livraisons d'un groupement de producteurs** est identifié par le code du groupement `gp`.
- **Document CALG** : il donne la liste des livraisons déclarées avec pour chacune:
  - son de code `livr`, date de livraison (théorique et immuable).
  - les dates-heures d'ouverture, de clôture des commandes, d'expédition et d'archivage. Des règles fixent comment et quand ces dates peuvent être changées, en particulier les unes par rapports aux autres.
  - la liste des points-de-livraison (groupes de consommateurs) livrés.

Une **livraison d'un groupement** est identifiée par le groupement livrant et son numéro de livraison: `gp livr`.
- **Document LIVRG**:
  - _date-heure d'ouverture_: on ne peut pas commander avant cette date. Les conditions de vente (prix / poids des produits) sont fixées mais peuvent subir des variations après ouverture. Dans ce cas les conditions à l'ouverture et les conditions courantes existent toutes les deux.
  - _date-heure de clôture des commandes_: on ne peut plus commander après cette date-heure.
  - _date-heure d'expédition_. Début du chargement des camions. Les conditions prix / poids des produits ne peuvent plus changer après cet instant, des paiements effectifs et définitifs pouvant s'engager.
  - _dates-heures de livraison_: il y a une date-heure et une adresse de livraison par point de livraison au groupe de consommateur.
  - _dates-heures de distribution_: il y a une date-heure et une adresse de distribution par point de livraison au groupe de consommateur, les produits peuvent commencer à être récupérés.
  - _date-heure d'archivage_: plus rien ne peut changer, les rectifications ultérieures sont traitées manuellement, la livraison est archivée.

_Remarque_: il y a des redondances entre le calendrier des livraisons **CALG** et les livraisons **LIVRG**. Le calendrier est en avance, mais dès qu'une livraison est _détaillée_, le détail est reporté dans le calendrier (s'il y a lieu).

Un **bon de commande d'un consommateur** est identifié par `gc co gp livr`.
- Il contient les informations:
  - relatives à la commande (ce qu'il souhaitait).
  - relative à la distribution (ce qu'il a eu).
- **Document BCC**: il donne pour chaque produit la quantité qui peut être en unité ou en poids au Kg. Les unités peuvent être des _demi_ (une demi caisse) selon les produits (pas de _demi_ paquet de café). Pour la distribution, le _poids_ peut être donné par une liste de poids de paquets individualisés (des poulets par exemple).

Un **bon de commande d'un groupement** est identifié par `gc gp livr`.
- Il contient les informations:
  - relatives à la commande: la somme des commandes des consommateurs.
  - relative à la distribution: ce qui a été déchargé du camion.
- **Document BCG**:
  - **généré / mis à jour à chaque mise à jour d'un BCC** d'un consommateur du groupe. Redondance partielle, il a des informations propres.
  - il donne pour chaque produit la quantité qui peut être en unité ou en poids au Kg. Les unités peuvent être des _demi_ (une demi caisse) selon les produits (pas de _demi_ paquet de café). Pour la livraison, le _poids_ peut être donné par une liste de poids de paquets individualisés (des poulets par exemple).

Un **carton d'un producteur pour la livraison à un groupe** est identifié par `gp pr livr gc`.
- **Document CART**: 
  - généré / mis à jour à chaque mise à jour d'un `BCC` du groupe pour une ligne concernant un produit de ce producteur. 
  - il donne par produit du producteur la somme des quantités commandées dans le groupe. Ce document est une redondance générée / mise à jour à chaque mise à jour.

_Remarque 1_: quand une commande est réceptionnée, pour chaque produit (en général commandé mais pas forcément), figure une quantité livrée ou un poids livré.
- pour certains produits, des poulets par exemple, la quantité livrée est le nombre N de poulets et le poids livré est une liste de N poids individuels des poulets. Quand il y a un poids total c'est, soit temporaire en estimation avant obtention des poids individuels, soit la somme des poids individuels.

_Remarque 2_: le rapprochement entre les informations de commande et celles de livraison (réception) permet de détecter des problèmes à résoudre:
- des quantités excédentaires ou insuffisantes: il faudra pour chaque consommateur ajuster la quantité distribué.
- des paquets individualisés (des poulets) au déchargement non attribués et des consommateurs n'ayant pas reçu leurs paquets.
- des produits livrés mais pas commandés: il faudra les répartir sur des consommateurs, le cas échéant un consommateur virtuel représentant le point-de-livraison lui-même.

Le **catalogue général des produits** d'un groupement est identifié par `gp`.
- **Document CATG**. Il donne pour chaque produit:
  - un _descriptif permanent_ ne pouvant pas changer après déclaration, sauf le libellé. Un produit _à l'unité_ ne peut pas devenir _au poids_ (il faut définir un autre produit).
    - son code et sa référence (_code-barre_).
    - un libellé descriptif,
    - si c'est un produit _sec_, _frais_ ou _surgelé_.
    - des indicateurs de label / qualité, taux de TVA applicable, 
    - si le produit est vendu à l'unité, au poids et / ou par _caisse_ / _demi-caisse_.
  - une _liste chronologique_ de dates à laquelle les conditions de ventes du produit ont changé:
    - sa disponibilité,
    - son prix unitaire,
    - ses poids _net_ et _brut_: le poids _net_ est celui sans l'emballage (ce que mange le consommateur), son poids _brut_ inclut l'emballage (ce que ça pèse approximativement dans le camion).

Le **catalogue d'une livraison** d'un groupement est identifié par `gp livr`.
- **Document CATL**. Il donne pour une livraison, la liste des produits avec pour chacun sa condition de vente.
  - la catalogue d'une livraison est _calculé_ avant ouverture de la livraison depuis le catalogue général.
  - il peut être amendé ponctuellement jusqu'à l'ouverture de la commande.
  - après ouverture de la commande, quand une condition de vente d'un produit change, les deux conditions existent: celle _actuelle_ et celle _à l'ouverture de la commande_.
  - les conditions de vente ne peuvent plus changer après la date-heure d'expédition (des paiements définitifs ont pu avoir lieu).

Le **répertoire général** des groupes et groupements n'a pas d'identifiant (c'est un singleton):
- **Document RG**:
  - pour chaque groupe, une _carte de visite_ du groupe.
  - pour chaque groupement, une _carte de visite_ du groupement.

Le **répertoire des consommateurs** d'un groupe est identifié par le code du groupe `gc`.
- **Document RCO** : pour le groupe lui-même et pour chaque consommateur il donne une fiche de contact.

Le **répertoire des producteurs** d'un groupement est identifié par le code du groupement `gp`.
- **Document RPR** : pour le groupement lui-même et pour chaque producteur il donne une fiche de contact.

Le **chat d'une livraison** est identifié par le groupement livrant et la date de livraison `gp livr`.
- **Document CHL**: c'est une suite chronologique de news alimentée par le groupement. Tous les groupes de consommateurs sont concernés. Un point-de-livraison (groupe de consommateurs) peut y déposer aussi des news (avec modération).

Le **chat d'une distribution** est identifié par le groupement livrant, la date de livraison et le point-de-livraison distributeur `gp livr gc`.
- **Document CHD**: c'est une suite chronologique de news alimentée par les consommateurs.

Le **chat d'un point-de-livraison (groupe de consommateurs)** est écrit par les consommateurs indépendamment de toute livraison et est identifié par `gc`.
- **Document CHCO**: c'est une suite chronologique de news alimentée par les consommateurs. Les groupements de producteurs peuvent émettre, avec modération, des news.

Le **chat d'un groupement de producteurs** est écrit par le groupement et les producteurs du groupement indépendamment de toute livraison et est identifié par `gc`.
- **Document CHPR**: c'est une suite chronologique de news alimentée par le groupement et les producteurs. Les groupes de consommateurs peuvent émettre, avec modération, des news.

> La partie de gestion des paiements est omise dans ce use-case tout comme la gestion et la consultation historique des livraisons archivées.

## Point de vue d'un consommateur
Un consommateur souhaite voir:
- **les calendriers généraux des livraisons**: vision de _reporting_, il n'a pas à en suivre en temps-réel les variations. Zoom à la demande sur chaque calendrier.
- **la liste des livraisons ouvertes avec leur statut et dates-heures**: vison _synchronisée_ présentant pour chaque livraison la _synthèse de sa commande_ : nombre de références, somme des poids brut et somme des prix.
  - il peut _zoomer sur une livraison et avoir une vision _synchronisée_ de la livraison:
    - le détail de sa commande,
    - la synthèse de la commande pour son groupe: en effet pour chaque produit un consommateur aime à savoir globalement ce que les autres membres du groupe ont commandé.
- **les catalogues généraux des produits de chaque groupement**: vision _reporting_ avec zoom à la demande de chaque catalogue.
- **les catalogues des produits spécifiques des livraisons ouvertes**: vision _reporting_ avec zoom à la demande sur le catalogue applicable à une livraison.
- **le chat de son groupe**: vision _synchronisée_.
- **les chats des livraisons ouvertes**: vision _synchronisée_.
- **le répertoire des groupes et groupements**: vision _reporting_ donnant les _cartes de visite_ avec zoom sur certains répertoires:
  - pour _le répertoire des consommateurs de son groupe_ la fiche de contact du consommateur sauf les données _confidentielles_ qui ne sont visibles que pour lui-même.
  - pour -le répertoire d'un groupement_ la fiche contact (réduite) du groupement et les _cartes de visite_ des producteurs.

### Fils de news notifiés quand l'application n'est pas ouverte
- chat de son groupe.
- chats des livraisons ouvertes de son groupe.
- ses commandes sur les livraisons ouvertes. Ce dernier fil peut être utile pour un _consommateur_ pour lequel il y a plusieurs utilisateurs susceptibles de commander: famille, proches, voisins... Il permet de voir apparaître des notifications quand un de ces utilisateurs a modifié une commande.

> Il ne suit pas par fils de news les évolutions tarifaires, les évolutions des dates, etc. C'est l'animateur du groupe qui en fera les informations de synthèses sur le chat du groupe.

### Point de vue d'un groupe (un de ses animateurs) (A SUIVRE).

### _Fils de synchronisation_ des documents
Chaque document est accessible par son identifiant.

Certains documents peuvent être _synchronisés_: pour cela il faut définir dans quels _fils de synchronisation_ chaque document est rattaché:
- chaque application terminale déclare à quels _fils_ elle est abonnée de manière à recevoir une notification circonstanciée quand un document rattaché à ce fil a changé.
- chaque document peut être rattaché à au plus DEUX fils de synchronisation.

#### Liste des documents, définition de leurs index
- RG: répertoire général des groupes et groupements
- RC: répertoire des consommateurs: gc
- RP: répertoire des producteurs: gp

- FGC: fiche d'un groupe. gc
- FCO: fiche d'un consommateur: gc co
  - index 1 : gc
  - index 2 : co
- FGP: fiche d'un groupement: gp
- FPR: fiche d'un producteur: gp pr
  - index 1 : gp
  - filter : pr
- CATG: catalogue des produits d'un groupement: gp
- CATL: catalogue d'une livraison: gp livr

- CALG: calendrier d'un groupement: gp
- LIVRG: livraison d'un groupement: gp livr
  - index 1 : gp

- BCC: bon de commande d'un consommateur: gc co gp livr
  - index 1 : gc gp livr
  - index 2 : gc co
- BCG: bon de commande d'un groupement: gc gp livr
  - index 1 : gc gp
  - index 2 : gp livr
- CART: carton d'un producteur pour la livraison à un groupe: gp pr livr gc
  - index 1 : gc gp livr
  - index 2 : gp pr
  - index 3 : gp livr

- CHL: chat d'une livraison: gp livr
  - index 1 : gp
- CHD: chat d'une distribution: gp livr gc
  - index 1 : gp livr
  - index 2 : gp gc
- CHCO: chat d'un groupe de consommateurs: gc
- CHPR: chat d'un groupement de producteur: gp

#### Liste des _fils_
La description d'un type de fil donne:
- la liste de ses propriétés identifiantes.
- pour chaque document pouvant faire partie du fil:
  - son type,
  - le numéro de l'index (0, 1, 2) définissant ses propriétés d'appartenance. Par convention 0 pour l'id complète.
  - s'il peut y avoir un filtre pour éviter les notifications non pertinentes:
    - le numéro du paramètre de filtre (en général 1),
    - le numéro de l'index dans le document pour retrouver la propriété filtrée.

Fil `#CMDGC` : `gc.gp.livr` - commande d'un groupe à un groupement - Filtre de notification possible: `co`
- `BCG` : 0 - C'est un singleton dans le fil
- `CART` : 1
- `BCC` : 1 , 1/2 - Filtrage de notification possible avc l'index 2 (`co`) donnant la valeur du paramètre de filtre 1 (`co`)

Fil `#CMDOV` : `gc.gp` - commandes ouvertes d'un groupe à un groupement
- `BCG` : 1

Fil `#CALGP` : `gp` - calendrier des livraisons d'un groupement
- `CALG` : 0 - un singleton pour le fil
- `LIVRG` : 1

Fil `#CMDGP` : `gp.livr` - commandes des groupes à un groupement
- `CHD` : 1
- `BCG` : 2
- `CART` : 3

Fil `#CMDPR` : `gp.pr` : commandes ouvertes d'un producteur
- `CART` : 2

Fil `#RGC` : `gc` - Filtre de notification possible: `co`
- `RG` : - rien, RG est un singleton pour l'organisation
- `RC` : 0 - c'est un singleton pour le fil
- `CHCO`: 0 - c'est un singleton pour le fil
- `FGC` : 0 - c'est un singleton pour le fil
- `FCO` : 1 , 1/2

Fil `#RGP` : `gp` - Filtre de notification possible: `pr`
- `RG` : - rien, RG est un singleton pour l'organisation
- `RP` : 0 - c'est un singleton pour le fil
- `CHPR` : 0 - c'est un singleton pour le fil
- `FGP` : 0 - c'est un singleton pour le fil
- `FPR` : 1 , 1/1

Fil `#CHL` : `gp` - Filtre de notification possible: `gc`
- `CHL` : 1
- `CHD` : 1 , 1/2

### Abonnement à un fil, _filtre_
Pour s'abonner à un fil il faut fixer:
- son code et son path exact: `#CMDGC/gc1.gp1.livr1`
- les types de documents cités dans l'abonnement sont seuls considérés : `[BCG, CART, BCC]`
- la ou les valeurs de filtres à appliquer pour éviter une notification non pertinente: `co1` (qui s'appliquera aux documents BCC dont l'index 2 est égal à `co1`).

### _Template de session_ et extensions dynamiques
Un _template de session_ a un code `@GROUPE` et une liste de propriétés identifiantes:
- par exemple `@GROUPE [gc]`
- un ou plusieurs types de _credential_. Pour chacun, toutes ses propriétés identifiantes doivent être citées dans l'identifiant du template. Le type de _credential_ `CREDGC` a pour identifiant `gc`.

Un _template de session_ est constitué d'un ou plusieurs _fils_ dont le path est entièrement fixé depuis les propriétés identifiantes du template, pour chaque fil, quel est le type de credential à appliquer: 
- par exemple le fil `#RGC.gc [RG, RP, CHCO, FGC, FCO]` - type de _credential_ `CREDGC`.

> L'exemple précédent est celui d'un _template de session_ avec un seul fil. Dans la réalité il y a en général plusieurs fils à ouvrir, sous contrôle d'un (a minima en général), voire plusieurs (plus rarement) _credentials_.

**Quand dans une session un utilisateur veut _ouvrir une session suivant un template_** il doit fournir,
- a) son type @`GROUPE` et tous ses paramètres (`gc1`): `@GROUPE/gc1`.
- b) le (ou les) _credential_ requis par le template dont le serveur vérifiera qu'il lui donne le droit d'accéder aux fils de ce périmètre. 
  - par exemple _le_ ou _un des_ mots de passe enregistré pour le groupe `gc1`.

Le _succès_ de la connexion est l'abonnement de l'application terminale au(x) fil(s) du groupe `gc`.

#### _Template de session_ pour un groupe: `gc` - liste des abonnements
Un _template de session_ autour d'un groupe `gc` contient:
- un fil _principal_, `#RGC.gc [RG, RP, CHCO, FGC, FCO]` : le document `RG` donne une liste calculable des groupements `gp`, à qui il peut commander, les fiches du groupement et des consommateurs, le chat du groupement.
- N fils _secondaires_ `#CALGP/gp [CALG, LIVRG]`, un par `gp` de la liste calculée ci-dessus.
- N fils _secondaires_ `#CHL/gp [CHL, CHD {gc}]`, un par `gp` de la liste calculée ci-dessus.

En cours de session une application peut fixer une livraison _courante_ `gp.livr`, elle s'abonne au fil `#CMDGC/gc.gp.livr [BCG, CART, BCC]` (elle pourrait également en avoir plusieurs selon la logique applicative souhaitée).

#### Un _template_ partiellement déterminé
La construction du _template_ `#GROUPE` **n'est donc entièrement déterminée** par le seul identifiant `gc` du template: elle dépend d'une liste de valeurs calculée depuis le document `RG`.

Il peut y avoir deux visions de ce template, statique et dynamique.

##### La vision _statique_ 
Un second paramètre de `$GROUPE` est décrit comme une liste `*gp` qui s'écrit `@GROUPE/gc1, *gp=RG.lstgp`. 
- Un pré-traitement lit le document `RG` et calcule la liste des `gp` passée en paramètre par sa méthode `lstgp`. 
- Le template devient alors complètement déterminé après ce traitement d'initialisation. 
- Dans ce cas, l'évolution éventuelle de `RG` pouvant possiblement changer `*gp` n'est pas censée recalculer le _template_ et la liste des fils qui en dépendent.

##### La vision _dynamique_
La description est semblable à la vision _statique_, mais de plus à chaque fois que le document `RG` est détecté changé sur l'écoute du fil `#RGC` (ce qui dans cet exemple est improbable en pratique), il faut réinitialiser les fils:
- se désabonner de tous les fils `#CALGP #CHL` dont le `gp` n'est plus dans la nouvelle liste: ce sont des groupements qui n'existent plus ou ne livrent plus ce point-de-livraison,
- s'abonner à tous les fils `#CALGP #CHL` pour les `gp` de la nouvelle liste n'étant pas dans l'ancienne: nouveaux groupements ou groupements nouvellement disposés à livrer le point-de-livraison.

L'application terminale étant notifiée des évolutions de `RG` elle peut simplement émettre une requête au serveur pour faire recalculer / changer ce groupe d'abonnements. Sauf qu'en l'occurrence, le serveur doit disposer de la liste actuelle des `gp` pour établir la différence avec la nouvelle qu'il est capable d'obtenir. Ceci introduit un concept de _fil secondaire d'un principal_:
- pour un _principal_ connaître la liste de ses secondaires,
- pour un _principal_, garder ses paramètres à calculer, à la fois en disposant du nom du calcul et en conservant dans l'instance du fil lui-même les résultats du dernier calcul d'initialisation.

#### _Template de session_ pour un consommateur: `gc co` - liste des abonnements
Ce _template_ autour d'un groupe `gc co` contient un fil _principal_ `#RGC.gc [RG, RP, CHPR, FPR]`.

Comme ci-avant il y aura des fils secondaires dépendant associé à chaque `gp` à qui il peut commander: 
- N fils `#CALGP/gp [CALG, LIVRG]`, 1 pour chaque `gp` récupéré du document `RG`.
- N fils `#CHL/gp filtre:gp.gc [CHL, CHD]`

Quand une exécution d'une application fixe une livraison courante `gp.livr`, elle s'abonne au fil `#CMDGC/gc.gp.livr filtre:gc.co, [BCG, BCC]`, (`BCC` est filtré par son index 2 sur `gc.co`)

#### Abonnements pour un groupement: `gp` (A REVISER)
Abonnement à `#RGC [RG, RC]` : donne une liste des groupes `gc` qui peuvent commander.

Abonnement au fil `#CALGP/gp [CALG, LIVRG]`

Quand il a fixé une livraison `livr`, abonnement à `#CMDGP/gp.livr [CHD, BCG, CART]`.

Abonnement au fil `#CHL/gp [CHL, CHD]`

Abonnements à `#CHPR/gp`.

#### Abonnements pour un producteur: `gp pr`
(TODO)

-----------------------------------------------------------------------

# Contributions diverses en attente

### Désérialisation de la propriété `data` du document
Elle consiste à retourner une _map_ nom, valeur des propriétés du document, dont celles d'identification et la version.

La couche applicative est en charge de créer une instance de la classe appropriée depuis cette _map_ en utilisant le type du document et si nécessaire d'autres propriétés de _data_ pour des sous-classes héritant d'une classe racine correspondant au type de document.

### Lecture d'un fichier
Elle peut s'effectuer de deux manières:
- en retournant le contenu binaire du fichier dans la couche applicative,
- en retournant une URL d'accès sécurisé valable un certain temps, typiquement à transmettre à une application externe.

### Fin d'une transaction
A la fin d'une transaction le framework connaît la liste des documents créés / modifiés / supprimés et donc des fils associés.

En consultant les abonnements des sessions pour chaque fil, il en résulte pour chaque application terminale (identifiée par son _token_) intéressée, une _liste de notifications_ formée des _fils changés_.
- celles qui ne correspondent pas au _token_ de l'application terminale ayant émis la requête, sont notifiées par des messages _webpush_.
- celle initiatrice de la requête reçoit cette liste de notifications en retour de la requête, donc très vite et par un canal raccourci ce qui lui permettra de mettre à jour son affichage au plus tôt.

#### Cohérence _forte_ dans un fil, _faible_ entre fils
L'état d'un fil retourné par une requête est _fortement cohérent_: cette configuration a existé vraiment à un moment donné.

Mais deux demandes faites pour deux fils, forcément à des moments différents, retourne deux états de fils qui ont pu ne jamais exister conjointement: il en résulte une _cohérence faible_ entre fils, un état qui globalement peut être fonctionnellement incohérent temporairement.

> On peut certes grouper dans la même requête des demandes concernant plusieurs fils: toutefois le volume correspondant retourné peut être important et la transaction correspondante de collecte être longue et induire des blocages techniques de la base de données. Il y a applicativement un compromis à choisir entre _force de la cohérence entre arbres_ et lourdeur technique.

## Décompte des consommations 
(En réflexion)

## Authentification _double_
(questions, pertinence)

Faut-il prévoir d'obliger à une authentification depuis un appareil #1 exigeant une confirmation sur un appareil #2 (favori ou non).
- que se passe-t-il quand l'appareil #1 n'est déclaré _favori_ ?
- et si l'utilisateur N'A PAS d'appareil #2 (au moins sous la main) ?
- si #2 n'est PAS favori, pour valider le login de #1 il lui faut un couple (s1, s2) qu'il vient a priori déjà de donner sur #1 ?
