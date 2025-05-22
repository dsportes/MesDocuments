---
layout: page
title: Réflexions à propos d'un framework "Orienté document"
---

Dans le langage courant on emploie le mot _application_ (Facebook, TikTok ...) pour désigner en réalité un système applicatif dont l'architecture schématique peut se résumer en deux niveaux:
- des _serveurs centraux_ détiennent les données et effectuent les calculs sollicités par des demandes émises par ...
- des _applications_ s'exécutant sur les terminaux / appareils des utilisateurs et sollicitant les serveurs centraux.

### Installation de l'application sur un appareil / terminal (_device_)
Un PC, une tablette, un mobile sont des _appareils / terminaux_ munis d'un moyen de communication avec un humain (écran, clavier, souris ...).

Selon la variante technique choisie, il faut,
- soit dans un browser _ouvrir la page Web_ de l'application.
- soit _l'installer_ sur chaque appareil d'où un utilisateur souhaite s'en servir.

#### Application de type Web : PWA _Progressive Web Application_
Depuis un browser l'utilisateur appelle une URL d'un _magasin d'applications_ qui ouvre une page contenant l'application:
- l'application peut être directement utilisable depuis cette page.
  - L'utilisateur peut déclarer un _raccourci sur son bureau_ (ou dans son browser) vers cette page afin d'éviter la saisie de l'URL de l'application. 
  - Certains OS (comme iOS) des appareils ne permettent pas une utilisation directe d'une telle page Web et oblige à une installation, au demeurant simple, de l'application depuis cette page.
- l'application _peut ou doit_ (selon le browser utilisé et l'OS de l'appareil) être _installée_ par le browser. Elle apparaît désormais comme une application locale de l'appareil avec une icône de lancement, typiquement sur le bureau.

#### Application de type _mobile_
L'utilisateur l'installe depuis le ou un des magasins d'application supportés par l'OS du mobile.

> Il n'y a ensuite quasiment pas de différence perceptible par l'utilisateur à l'utilisation de l'application, il clique sur une icône pour l'ouvrir (la lancer).

> On peut installer une application Web-PWA sur un mobile: elle est considérée comme application PWA (et non comme _mobile_).

### Serveurs
Les applications en exécution sur leur appareil envoient à des **serveurs centraux** des demandes de mise à jour et consultation des données de l'application.

Le terme générique de _serveurs_ recouvre des variantes techniques non perceptibles de l'extérieur:
- **Serveurs permanents**: plusieurs processus sont en exécution en permanence afin de traiter les requêtes qui leurs parviennent sur l'URL du pool de processus et ont été routées vers l'un ou l'autre.
- **Cloud Function**: un _serveur éphémère_ du _Cloud_ est lancé pour traiter une demande de service reçue sur son URL:
  - la demande est traitée et le serveur éphémère reste actif un certain temps pour traiter d'autres demandes. Un serveur éphémère peut traiter plusieurs dizaines de demandes en parallèle.
  - en l'absence de nouvelles demandes, un serveur éphémère reste en attente, entre 3 et 60 minutes pour fixer les idées, puis s'interrompt.
  - si le flux des demandes sature la capacité d'un serveur éphémère, un deuxième, voire un troisième etc. sont lancés.

> Ces choix de déploiement technique ne sont pas détectables par les applications qui envoient des demandes au serveur.

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
- enfin ils peuvent différer selon le prix de leurs prestations et leur qualité (temps de réponse, disponibilité, restrictions de volume, ...).

> Si le nom d'un service comme `randos` traduit sa fonctionnalité générale, un _service précis_ est désigné par le couple du **nom de service / nom du prestataire** le proposant : `randos/Rouge`. Ce couple se traduit par une URL qui permet de solliciter _ce_ service spécifiquement délivré par _ce_ prestataire.

# Les _applications_, leurs magasins, leurs variantes
Le logiciel d'une application est disponible dans un magasin: dans le cas d'une application Web le magasin est un site Wen statique dont chaque URL correspond à une application.

Pour afficher et gérer ses randos, il faut en conséquence installer l'application `randos` depuis un magasin. Pour un nom donné d'application, il peut exister le cas échéant plus d'une variante:
- `randos-mobile` par exemple peut se limiter au sous-ensemble des fonctionnalités utiles pendant la rando et sous un format simplifié adapté à consulter un écran de petite taille dans des conditions de luminosité pas optimale.
- `randos` par exemple propose toutes les fonctionnalités de l'inscription à la consultation d'historique en supposant une surface de lecture plus importante (PC ou tablette).

C'est à chaque utilisateur de choisir la **variante** qu'il veut installer (voire les variantes) quand plusieurs sont disponibles et selon chacun de ses appareils.

Les _variantes_ de l'application terminale `randos` ont pour caractéristiques de faire appel au service `randos` d'un des prestataires le proposant.

> Il n'est pas exclu qu'une application _terminale_ donnée soit restrictive vis à vis du choix du prestataire de service. Ce peut être le cas quand un prestataire de service _haut de gamme_ propose aussi dans un magasin une variante plus complète de l'application terminale complétée d'écrans accédant aux prestations supplémentaires qu'il offre.

# Organisations: les services sont _multi-tenant_
Un service comme `randos`, peut à la manière de Discord, proposer d'héberger les applications d'associations de randonneurs distinctes: chaque organisation / _tenant_ dispose de _son_ espace de données propre complètement étanche à celui des autres.

Un service `boutiques` va proposer de gérer plusieurs boutiques, pas une seule, mais de manière à ce que les données de chacune soient totalement isolées de celle des autres.

Les données d'un service d'un prestataire sont stockées dans deux _mémoires persistantes_ (voir plus loin):
- **UNE base de données** logiquement strictement partitionnée par organisation, sans aucun lien ou référence à des données / documents d'une organisation par une autre.
- **UN _storage_ de fichiers**, comme un directory de fichiers classiques, avec une **racine** par organisation.

### Pour un service donné, UNE organisation donnée n'est hébergée que par UN prestataire
Pour un service `randos` proposés par les prestataires **Rouge** et **Bleu**, une organisation donnée _val-de-bièvre_ est _hébergée_ chez **Rouge** ou chez **Bleu** mais pas dans les deux.

UNE base centrale unique pour `randos` indique pour chaque organisation le prestataire qui l'héberge (l'URL d'appel du service).

> Une organisation peut _migrer_ d'un prestataire à un autre: ce transfert technique des données est génériquement possible, sauf quand un prestataire a des données additionnelles absentes chez l'autre.

### Une application terminale peut accéder à plus d'une organisation
Dans le cas de l'application `randos`, un utilisateur donné peut parfaitement faire partie de plus d'une association de randonneurs: une pour ses randonnées près de chez lui, une autre pour les randonnées de montagne et une troisième pour les treks lointains. Depuis la même application il peut basculer d'une organisation à une autre.

Un gestionnaire de boutiques peut par exemple gérer trois boutiques différentes (trois organisations) avec des rôles différents pour chacune.

Les utilisateurs de Discord font souvent partie de plusieurs groupes, qui s'ignorent entre eux, ayant des sujets d'intérêt totalement différents.

L'utilisateur qui ouvre sur son poste son application `randos` peut disposer de pages de synthèse lui montrant ce qui est important pour chacune des associations auxquelles il participe. Pour agir effectivement sur l'une d'entre elles, il basculera sur une page d'accueil spécifique de l'association sélectionnée et ses actions de mises à jour ne porteront que sur celle-là.

# L'exécution d'une application sur un appareil
Sur un appareil donné, on ne peut pas lancer plus d'une exécution d'une application donnée, par exemple une seule application `randos`.

> Dans le cas d'une application Web-PWA, chaque browser (Firefox, Chrome ...) est vu comme un **appareil différent**: on peut avoir s'exécutant au même instant sur son PC, une application sous Firefox ET une application sous Chrome.

**Une application sur UN appareil** peut avoir trois états:
- être en exécution au **premier plan**. Sa fenêtre est affichée et a le _focus_ (elle capte les actions de la souris ou du clavier). Pour un mobile c'est celle (ou l'une des deux) visibles.
- être en **arrière plan** : elle a été lancée mais est recouverte par d'autres.
  - sur un browser, c'est un autre onglet qui a le focus ou la fenêtre du browser est en icône: mais l'utilisateur peut cliquer sur son onglet pour l'amener au premier plan ou sur l'icône du browser dans la barre d'icônes pour l'afficher.
  - sur un mobile elle est cachée mais peut être ramenée au premier plan quand l'utilisateur la choisit dans sa liste des applications _ouvertes mais cachées_.
- être **non lancée**: son exécution n'a pas encore été demandée, ou a été active puis fermée.

### Une application peut _envoyer_ des requêtes aux serveurs
C'est l'application qui appelle un serveur identifié par son URL: le serveur traite la requête et retourne un résultat.
- requêtes et réponses peuvent être volumineuses.

### Une application peut _écouter_ des notifications émises par les serveurs
Une application donnée sur un appareil donné est identifiée par un _jeton_ qui une sorte de numéro de téléphone universel: tout serveur ayant connaissance de ce jeton peut envoyer des _notifications_ à l'application correspondante sur le poste correspondant.

Une notification ressemble à un SMS:
- son texte est _court_ (certes plus long que celui d'un SMS).
- on ne répond pas _directement_ à une notification: le serveur émetteur ne sait rien de la suite donnée, ou non, par l'application destinataire.

> Toutefois l'application peut évidemment en tenir compte et effectuer des traitements et des requêtes ultérieures aux serveurs.

**Quand l'application destinatrice d'une notification est au PREMIER PLAN:**
- elle peut afficher un court message dans une petite fenêtre _popup_ (voire émettre un son ...) pour alerter l'utilisateur,
- elle peut effectuer le traitement simple ou complexe adapté aux données portées par la notification.

**Quand l'application destinatrice est en ARRIÈRE PLAN:**
- elle peut (ou l'OS de l'appareil ou le browser dans lequel elle s'exécute) peut afficher en _popup_ la notification ce qui alerte l'utilisateur,
- si l'utilisateur clique sur cette _popup_, l'application correspondante repasse au premier plan.

**Quand l'application destinatrice N'EST PAS en exécution:**
- l'OS de l'appareil ou le browser dans lequel elle est enregistrée, peut afficher en _popup_ la notification ce qui alerte l'utilisateur,
- si l'utilisateur clique sur cette _popup_, l'application est lancéee.

## Des applications _écoutantes_ réagissant au flux d'informations poussées
Les applications **_sourdes_** classiques ne peuvent afficher des écrans que sur sollicitation de l'utilisateur: l'écran ne se remet à jour que suite à une action de l'utilisateur:
- si ce dernier ne fait rien, l'écran ne change pas et affiche des données plus ou moins anciennes qui ont déjà été modifiées par l'action d'autres utilisateurs, du temps qui passe, etc.

Les applications **_écoutantes_** peuvent remettre à jour leurs écrans et données détenues localement même sans action d'un utilisateur simplement en fonction des _notifications_ poussées vers elles par les serveurs.

# Le paradigme _fils de news_ / _collections de documents synchronisés_
Les _données d'une application_ sont structurées comme des collections de documents, chaque document pouvant rassembler un volume significatif d'informations structurées de manière relativement complexe.

Suivant ce paradigme, une application présente à son utilisateur trois concepts:
- des **_fils d'information_** annonçant des évolutions de documents ou de collections de documents qui l'intéresse: l'arrivée de nouveaux échanges sur un _chat_ (un document), uné évolution tarifaire (un tarif vu comme une collection de documents). Ces fils **annoncent** par des notifications courtes une évolution de certains documents, mais n'en donne q'un minimum d'information.
- des **_collections de documents synchronisés_**: les documents de la collection sont systématiquement maintenus à jour dans l'application dans un état le plus proche techniquement possible de l'état des documents sur le serveur.
- des **_rapports_**: ce sont vues calculées à un instant donné et qui ne changent pas, sauf bien entendu à redemander le même rapport.

**Les documents synchronisés dans une application** le restent a minima tant que l'application est **au premier plan**:
- l'application peut décider de ne plus maintenir cette synchronisation quand elle passe **en arrière plan**: c'est une économie de ressources et comme en pratique l'utilisateur ne voit d'une application en arrière plan que les _popups_ de notification, maintenir à jour un volume important de documents synchronisés n'a pas forcément d'intérêt.
- en repassant au premier plan, elle demandera aux services de lui fournir les mises à jour survenues sur les documents synchronisés depuis l'arrêt de cette synchronisation.

### Quand l'application n'est plus en exécution
Quand une application est en exécution elle est _abonnée_ à des _fils d'information_.

Quand son exécution s'arrête, sauf décision explicite de l'utilisateur, ces abonnements restent actifs, du moins un certain temps:
- les notifications correspondantes continueront à s'afficher en _popups_, l'OS ou le browser de l'application s'en chargeant.
- l'utilisateur reste informé des _news_ auxquelles il était abonné.
- un clic sur un ces _popups_ ouvre l'application ce qui lui permet de connaître en détail les documents ayant changé.

### Des _fils_ plus ou moins riches
Pour assurer la synchronisation d'une collection de documents, le _fil_ correspondant est riche: il peut y avoir beaucoup de documents modifiés. Les **_fils de synchronisation_** ne donnent lieu à des _popups_ que sur des critères très restrictifs gérés par l'application afin de ne pas submerger l'utilisateur.

Les **_fils de news_** sont a contrario beaucoup plus sobres: ils correspondent à quelques documents / collections bien ciblés et pas à tous ceux qui seraient nécessaires à une synchronisation complète de ces documents.

> Ce paradigme ne peut pas être mis en œuvre dans toute sa généralité: comment sont définis les _fils de news_ et les _documents synchronisés_ peut aboutir à une impossibilité technique de mise en œuvre, ou à un coût de développement prohibitif, ou à un coût calcul insupportable. La solution générique décrite ci-après correspond à une restriction de ces concepts permettant de faire fonctionner des applications avec un minimum d'effort, tant de développement que de calcul.

# Un utilisateur, ses appareils et ses applications

Un utilisateur qui veut utiliser une application (Web-PWA ou non) est placé devant deux cas de figure:
- **l'appareil qu'il s'apprête à utiliser est _le sien_**: un peu plus généralement c'est un appareil,
  - qu'il utilise régulièrement, que se soit le sien ou celui d'un proche,
  - c'est un appareil de confiance: il peut laisser quelques informations personnelles localement dessus.
- **l'appareil qu'il s'apprête à utiliser ne lui est pas familier**, il est partagé par un grand nombre d'utilisateurs comme au cyber-café ou est celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelques informations que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement.

### Appareil _personnel_: ses profils
En pratique dans bien des cas c'est un appareil qui peut être utilisé par d'autres que soi-même. Typiquement dans un cadre familial ou un couple, l'appareil n'est pas strictement _personnel_.

En général un tel appareil est _protégé_ par un mot de passe ou tout autre dispositif que seuls des proches connaissent (à moins d'ailleurs que l'appareil leur soit prêté avec une session ouverte). Il n'empêche que _plusieurs personnes peuvent plus ou moins occasionnellement s'en servir_: d'où le principe d'y avoir possiblement plusieurs _profils_.
- un browser comme Firefox a une notion d'utilisateur: on peut basculer d'un utilisateur à un autre (sans pour autant avoir changé de connexion au niveau de l'OS), ce qui va conférer à chacun ses sites favoris, son historique de navigation et ses mots de passe enregistrés.
- Thunderbird, le gestionnaire de mails locaux, supporte de gérer plusieurs _profils_, chacun avec ses comptes mails.

### Des _profils_ plus ou moins bien défendus
Ce n'est pas parce qu'on partage un appareil avec un proche qu'on a envie de partager ses informations confidentielles.

Or dans les cas cités ci-dessus, la confidentialité est plutôt _lâche_:
- Thunderbird ne demande rien: on choisit son profil, sans mot de passe ou quoi que ce soit. Les boîtes mail sont de toutes les façons en clair dans le file-system, confidentialité zéro.
- Chrome s'ouvre sur le compte _courant_: si vous ne vous déconnectez pas **explicitement** avant de fermer le browser, il s'ouvre la fois suivante sur votre compte, ses mots de passe, ses historiques et ses favoris. Si vous vous déconnectez il est strict sur la connexion et demandera même à votre téléphone si c'est vraiment vous qui essayez de vous connecter: il suffit d'y répondre OUI et c'est bon (même si vous vous êtes fait voler votre téléphone en état déverrouillé, Google est content).

### Profil personnel local à un appareil
C'est un petit stockage local de données destiné à:
- faciliter et accélérer le démarrage des applications qui vont y trouver des données d'autorisation, le cas échéant multiples et potentiellement fastidieuses à fournir.
- permettre d'utiliser, avec des restrictions, une application en _mode avion_, sans aucun accès à Internet, à partir des documents synchronisés lors d'une utilisation antérieure.

Ce stockage est **crypté**: même l'accès par le _file-system_ ne permet pas à un _pirate_ d'accéder aux informations.

La clé de cryptage est dérivée d'une **phrase** que seul le détenteur du profil connaît: après deux échecs sur la donnée de celle-ci, le _profil_ s'efface (pas les données sur les serveurs des applications).

### Lancement d'une application sur un appareil _personnel_
La liste des quelques profils enregistrés est présentée:
- l'utilisateur choisit le sien et donne sa phrase de protection.
- si c'est la bonne phrase, l'application s'ouvre en ayant une série d'autorisations préremplies, ses groupes de dossiers synchronisés, ses fils de news ... Il peut:
  - en désactiver certains, temporairement ou non,
  - en activer de nouveaux qui compléteront son profil.

En fermant l'application, l'utilisateur peut choisir:
- de laisser ses _fils de news_ activés: des notifications apparaîtront, même quand l'application sera fermée et même si ce n'est plus vous qui avez l'appareil en main. Toutefois ses _fils_ ne font qu'annoncer des changements sans en donner les détails et jamais d'informations classées confidentielles. 
- de fermer ses _fils de news_: aucune notification ne parviendront plus sur l'appareil relativement à cette application. On peut le prêter à quelqu'un d'autre sans risque ... mais soi-même on ne recevra plus de notifications (il faut choisir).

### Lancement d'une application EN MODE AVION sur un appareil _personnel_
La liste des quelques profils enregistrés est présentée:
- l'utilisateur choisit le sien et donne sa phrase de protection.
- si c'est la bonne phrase, l'application va s'ouvrir en disposant des _dossiers synchronisés_ qui l'ont été lors de la dernière utilisation connectée.
- en mode avion, il n'y a pas de réseau, pas de _fils de news_: les dossiers peuvent être consultés mais pas mis à jour. Toutefois l'utilisateur dispose d'une mémoire locale où il peut saisir des textes ou formulaires purement locaux et stocker des fichiers (comme des photos prises en mode avion): toutes ces informations sont cryptées dans le stockage local.

### Lancement d'une application sur un appareil _anonyme_
L'application se lance sans aucune autorisation activée: elle ne va guère afficher que des informations d'une grande banalité mais surtout va permettre de saisir les autorisations nécessaires pour,
- accéder à des groupes de dossiers synchronisés ce qui va lui permettre de les consulter et de le mettre à jour selon les droits ouverts par l'autorisation fournie.
- ouvrir des _fils de news_ également en citant les autorisations nécessaires.

> Pour l'application, les possibilités sur un appareil _anonyme_ sont les mêmes que sur un appareil _personnel_: l'utilisateur a seulement fourni plus d'effort pour faire valoir ses droits d'accès (alors que son _profil_ les a préremplis pour un appareil _personnel_).

La fermeture de l'application ne laisse pas le choix: tous les _fils de news_ sont désactivés, aucune notification ne parviendra plus sur cet appareil résultant de l'usage précédent de l'application.

### Mode _veille_
Les applications ouvertes ont un mode _veille_ optionnel: s'il est activé, l'application se met en veille en cas de non utilisation pendant quelques minutes (fixés selon le degré de paranoïa de l'utilisateur). Pour sortir de la veille un code PIN est nécessaire et au second échec l'application se ferme.







### Base de données d'un _serveur_
**UNE base de données** est attachée à chaque _serveur_ identifié par son URL, qu'il soit exécuté par un processus unique ou un nuage de processus.

### Storage d'un _serveur_
**UN storage** est attaché à chaque _serveur_. Il stocke des _fichiers_ identifiés par leur _path_: la présence de caractères `/` dans un path définit une sorte d'arborescence.

Le _contenu_ de chaque fichier est une suite d'octets opaque.

## Organisations
**Un serveur gère une ou plusieurs organisations:** chacune a ses données distinctes, en base de données comme en storage. 

Les données sont partitionnées par _organisation_.

Sauf exception ci-après, chaque requête concerne UNE seule organisation et n'accède donc qu'aux seules données de _son_ organisation.

> Certaines requêtes **d'administration** ont pour cible _le répertoire des organisations_ et la gestion de celles-ci. Elles ne ciblent pas au départ _une_ organisation spécifique mais peuvent ensuite exécuter un traitement portant sur l'une ou plusieurs d'entre elles.

## Comptes d'une organisation
Une organisation a, en général, plusieurs **comptes**, le cas échéant beaucoup.

Chaque requête au serveur s'exécute (en général) **sous contrôle d'un compte** ce qui va déterminer:
- quelles données de l'organisation (en base de données / storage) sont accessibles par la requête,
- quels traitements peuvent être exécutés sous quelles conditions.

#### Un _compte_ n'est PAS assimilable à une _personne_
Certains comptes correspondent à des _rôles_ ou _personnes morales_: comptable, administrateur, modérateur, représentant d'une famille, chef d'équipe ... ces comptes peuvent être référencées par des _personnes_ (physiques) différentes.

Certains comptes peuvent sembler avoir un rôle plus _personnel_ mais si on pense par exemple à un compte en banque, des _procurations_ peuvent être attribuées à plusieurs personnes physiques.

## Sessions d'une application
Quand une personne lance une première fois l'application sur un appareil où elle n'a jamais été lancée, l'application s'ouvre sur un écran _vierge_ de tout contexte.

A ce moment la personne ne peut qu'accéder qu'à des informations générales, la météo, une information générale sur l'état du serveur, un manuel d'aide en ligne ... bref guère plus que ce que délivre un site Web statique, et encore **pour autant que l'application ait prévu de fournir ces informations gratuitement**.

### Ouverture d'une _session_ : identification / authentification de l'utilisateur
Pour utiliser _vraiment_ l'application une personne doit s'identifier (dire qui elle est) et s'authentifier (le prouver).

Après cette opération, une **session** est ouverte.

La personne peut accéder à plus d'informations (toujours pour autant que l'application l'ait prévu), par exemple à des informations éventuellement payantes (puisque qu'elle identifiée) ou réservées aux personnes _identifiées_.

Dans le cas où l'application est payante, la personne peut accéder:
- à sa situation _comptable_: le solde de son compte, des informations sur ses abonnements d'espace et consommations de calcul.
- à la déclaration d'un paiement, ou à sa pré-annonce quand le paiement effectif est reçu par un autre canal anonymisé.

#### Réception de _notifications_
Dès lors qu'une session est ouverte elle _peut_ recevoir des _notifications_: par exemple si sa situation comptable évolue du fait d'actions déclenchées par ailleurs. 

### Connexion d'une _session_ à un _compte_
Une _session_ ouverte peut se _connecter_ à un ou plusieurs comptes: c'est la logique applicative du logiciel du serveur qui détermine comment et quelles personnes (physiques) peuvent ou non se connecter à un _compte_ d'une organisation et comment il est identifié.

Une connexion d'une session peut avoir deux statuts différents:
- **active**: des requêtes peuvent être émises pour déclencher des traitements sous contrôle du compte.
- **monitoring**: le compte est sous _surveillance_ les modifications des données de son périmètre d'intérêt vont déclencher des **notifications**.

Une seule connexion d'une session est _active_ à un instant donné, les autres sont en _monitoring_.

> Une session peut en conséquence travailler activement sur un compte et en avoir d'autres  sous _monitoring_ et recevoir des notifications.

Le _compte actif_ d'une session dispose dans celle-ci d'une mémoire de _documents_ copies retardées (ou non) de ceux détenus par le serveur central: les _notifications_ reçues du compte actif signalent l'existence d'une mise à jour (création ou suppression) de documents du _périmètre d'intérêt_ du compte.

> Cette copie distante synchronisée permet d'afficher à l'écran, sans intervention de l'utilisateur, un état le plus à jour possible des données du fait d'évolutions quelqu'en soit les origines.

#### Une session peut ne pas avoir de _compte actif_
Si l'essentiel des requêtes émises depuis une session concerne le _compte actif_, certaines sont indépendantes de tout comptes:
- les requêtes de _connexion_ qui ont justement pour objectif de déterminer quel compte doit être rattaché à la session (pour devenir actif ou simplement monitoré).
- certaines requêtes de lecture de données considérées comme _publiques_ dans l'organisation.

### Réception de _notifications_ dans une session
Chacune a pour source un compte auquel la session est connectée:
- si c'est le _compte actif_ la notification peut signaler une évolution de documents du _périmètre d'intérêt_, la session en demandera le nouvel état afin que sa mémoire locale en soit synchronisée.
- si c'est un _compte monitoré_ la notification signale un événement digne d'intérêt, sans déclencher pour autant une mise à jour de copies de documents dans la mémoire de la session.


### Fermeture de l'application SANS fermeture de la session
La fenêtre de l'application est fermée, elle n'est plus en exécution, mais sa **session** est toujours considérée comme ouverte par les comptes connectés: en conséquence l'appareil (ou le browser) recevra toujours des notifications liées aux évolutions des données des comptes associés à la session.

Si la personne derrière l'appareil à cet instant, clique sur une telle notification, l'application s'ouvrira et rétablira sa session (ce qui peut ou non constituer un problème).

### Fermeture EXPLICITE de la session en cours
Les comptes connectés dans une session peuvent toujours en cours de session être _déconnectés_ individuellement. 

Quand la session est **explicitement** fermée par l'utilisateur,
- tous les comptes connectés sont déconnectés.
- l'identification de la personne est effacée.
- l' application se retrouve _vierge_ dans le même état que si elle n'avait jamais été lancée.
- plus aucune notification n'apparaîtra pour cette application.

## Déconnexions _automatiques_ ?
Certaines applications sensibles peuvent prévoir une déconnexion automatique du compte actif (voire des autres) au bout d'un certain temps sans activité de l'utilisateur. 

Sans déconnexions automatiques l'appareil va  continuer d'afficher des notifications relatives à une personne (et les comptes qu'elle avait connectés) qui _était_ celle derrière l'appareil il y a quelque temps MAIS ne l'est peut-être plus. Comme un clic sur une telle notification rouvre la session, la nouvelle personne derrière l'appareil se retrouve avec une session ouverte par la précédente (et les autorisations qui vont avec).

## Code PIN de passage au premier plan suite à une notification
Quand l'application était _en arrière plan_ et se retrouve propulsé au _premier plan_ parce que l'utilisateur a cliqué sur une notification, ça peut faire longtemps qu'elle était en arrière plan: dans ce cas il doit saisir un code PIN avant d'obtenir le ré-affichage de la session ouverte. 

En cas d'échec (une ou deux fois au plus) la session est fermée par précaution.

De même si l'application était avec une session en cours et que l'utilisateur clique sur une notification,
- si le bon code PIN est fourni elle s'ouvre à nouveau sur la session laissée ouverte au moment de la fermeture,
- sinon la session est fermée, les connexions fermées et l'identification de l'utilisateur à refaire.

## Synthèse
Laisser sa session ouverte en fermant l'application permet:
- de voir s'afficher des notifications relatives aux données de celle-ci, d'être en monitoring des comptes connectés à la session.
- de la réactiver rapidement sans avoir à s'authentifier et se connecter à nouveau aux comptes.
- un code PIN permet d'éviter qu'une autre personne en profite.
- a contrario la personne _suivante_ sur l'appareil verra s'afficher en notifications des informations qui concernent la personne précédente (à éviter quelles soient trop précises / confidentielles).

Des déconnexions automatiques en cas d'inactivité, et / ou en cas de fermeture de l'application:
- empêche de voir s'afficher des notifications,
- empêche de réactiver sa session rapidement,
- procure une confidentialité rigoureuse sur l'appareil quelqu'en soit la personne qui l'a sous la main.

Le comportement dépend du type d'appareil sur lequel une session est ouverte:
- au cyber-café ou sur un PC ou mobile NON personnel, il faut fermer ses sessions.
- sur un appareil très _personnel_ dont le login est bien protégé, c'est plus pratique de les laisser ouvertes.

# Mémoire _cache_ locale de données sur un appareil / mode "avion"
Au lancement d'une application sur un appareil, l'utilisateur se trouve devant plusieurs possibilités:
- il y a du réseau et l'utilisateur le considère comme fiable (non écouté malicieusement):
  - si une session est _ouverte_, il peut saisir son code PIN et la reprendre.
- sinon s'identifier 

Au lancement d'une application sur un appareil, l'utilisateur va déclarer si cet appareil est _personnel_, soit qu'il lui appartient, soit qu'il le partage avec une ou quelques personnes de confiance.


# Arbres de documents
## Documents, type de document
Un document contient des données structurées selon sa _classe_: soit directement une _classe majeure_, soit une sous classe de sa classe majeure.

> Par exemple la classe majeure `Account` peut avoir des sous-classes `AccountAdmin` `AccountStandard`. Le **type d'un document** est le nom de sa classe majeure.

## Arbres
Un **arbre** est un conteneur structuré **de documents et de fichiers** matérialisé par une instance d'une classe `Tree` avec les propriétés suivantes:
- `treetype`: type d'arbre, un code identifiant la nature de l'arbre. La liste fermée des types est déclarée dans l'application.
- `tid` : ID de l'arbre (un string court libre).
- `version` : numéro séquentiel d'ordre de mise à jour du document le plus récent dans l'arbre.
- `versions` : donne pour chaque type de document de l'arbre le dernier numéro de version, soit _du_ document si c'est un singleton, soit du document de la collection _le plus récemment mis à jour_.
- `cleandate` : date du dernier jour où les documents supprimés de l'arbre depuis plus d'un certain temps ont été physiquement détruits.

Un arbre est persisté en base de données par un record ayant les propriétés ci-dessus sous un path / clé primaire `treetype / tid`.

#### Documents
- des **singletons**: chaque document est identifié dans son arbre par son **type de document**.
- des **collections typées** de documents. Chaque document est identifié relativement à son arbre par,
  - `doctype` : son type de document.
  - `did` : son ID de document dans l'arbre (un string court). Cet ID peut être par exemple de la forme `A1.B1.C1` définissant de facto une arborescence fonctionnelle.

Toutes les classes de documents sont des sous-classes des classes abstraites `Document` ou `Singleton` (elle-même sous-classe de `Document`). Elles ont les propriétés suivantes: 
- `treetype tid` : identifiant de leur arbre.
- `doctype did` : identifiant relatif à leur arbre.
- `version`: numéro d'ordre de mise à jour dans leur arbre.
- `deldate` : jour de suppression du document si le document n'existe plus.
- `data` : données du document sérialisées en binaire si le document existe.

Un arbre est persisté en base de données par un record ayant les propriétés ci-dessus sous un path / clé primaire `treetype / tid / doctype / did`.

#### Fichiers
Un fichier est persisté en deux parties:
- un descriptif attaché à un arbre persisté en base de données,
- son contenu effectif stocké dans un **Storage** (de type AWS/S3, Google Storage, etc.) sous le path `treetype / tid / fid` où `fid` est un identifiant universel unique attribué lors du stockage du contenu.

Le descriptif d'un fichier est accessible depuis son arbre par sa clé / path `treetype/tid/"file"/did` et a les propriétés suivantes: 
- `treetype tid` : identifiant de son arbre.
- `"file" did` : identifiant relatif à son arbre.
- `version`: numéro d'ordre de mise à jour dans son arbre.
- `deldate` : jour de suppression du fichier si le fichier n'existe plus.
- `data` : données définies par l'application du document et sérialisées en binaire si le document existe, par exemple _un nom de fichier, un type MIME, sa taille, son SHA, etc._
- `fid` : identifiant du fichier dans le Storage.

La mise à jour d'un fichier `treetype tid "file" id` dans son arbre est une opération double:
- le stockage d'un nouveau contenu. Une opération `preLoad` lui attribue son identifiant de Storage `fid2` et le charge dans le Storage. L'opération enregistre dans un record `toDelete` de la base de données l'identifiant `treetype tid fid2` du fichier avec sa date de stockage.
- la validation de ce changement dans le cadre d'une transaction de la base de données en remplaçant dans son descriptif le `fid` actuel par le nouveau `fid2` et en changeant sa version. Le record correspondant de `toDelete` est supprimé.

La suppression d'un fichier est aussi une opération double:
- validation dans une transaction de sa suppression dans son descriptif dans son arbre (propriété `delDate`). Un record `toDelete` de la base de données est enregistré avec l'identifiant du fichier et sa date de suppression.
- la suppression physique dans le Storage du fichier identifié, puis la suppression du record correspondant dans `toDelete`.

Les mises à jour comme les suppressions sont donc en deux phases et il se peut que suite à un incident une phase s'exécute et pas la seconde. En scannant périodiquement dans la base de données toDelete on récupère les identifiants des fichiers _fantômes_ à nettoyer du Storage et qui ne sont plus référencés dans la base de données (donc injoignables).

## Gestion des numéros de version et des suppressions
Depuis un arbre portant la version 12, 
- la mise à jour d'un document de cet arbre portera la version 13,
- la version de l'arbre sera elle-même portée à 13. 

Si dans une même transaction il y a plusieurs mises à jour, elles portent toutes le même numéro de version.

### Gestion des suppressions
En cas de suppression, le numéro de version augmente aussi (ça sera la dernière fois pour ce document):
- sa propriété `deldate` porte la date du jour.
- sa propriété `data` est nulle.

Un arbre conserve ainsi _un certain temps_, par exemple 200 jours, la liste de ses documents supprimées. Une copie ancienne de cet arbre de moins de 200 jours peut ainsi être rafraîchie incrémentalement: les documents mis à jour sont détectés depuis leur numéro de version ainsi que les documents supprimés depuis.

# Objectifs du framework
Cette couche logicielle a plusieurs objectifs:
- gérer la persistance des arbres de documents et fichiers dans la base de données et le storage.
- gérer la _synchronisation cohérente_ de copies distantes des arbres.
- notifier par _push_ des applications distantes des évolutions des arbres auxquels elles ont souscrites.

Le framework a deux niveaux d'action:
- **dans une Cloud Function _centrale_** ayant accès à la base de données et au Storage. Ce peut être un serveur HTTPS standard, ou une Cloud Function sous différentes variantes technique.
- **dans des applications de différentes natures techniques** faisant appel aux services des Cloud Functions centrales, typiquement une application Web, une application mobile, etc.

## Gestion de la persistance _centrale_ dans une Cloud Function
Le framework gère le stockage des documents et fichiers dans le couple base de données qui Storage détient _l'original_ des **documents et fichiers regroupés en arbres** comme expliqué ci-avant.
- cette gestion est sécurisée par des transactions ayant la propriété ACID du SGBD.
- les contenus des fichiers créés / modifiés par une transaction doivent avoir fait l'objet d'un `preLoad` en Storage AVANT mise à jour par une transaction. Le framework assure le nettoyage des contenus du Storage non référencés dans un arbre (suite à un incident technique).

### Création / mise à jour / suppression
Une couche logicielle d'API permet à la couche applicative de Cloud Functions d'accéder aux documents en mémoire sous forme d'objet de classes applicatives et de déclarer des créations / suppressions / mises à jour des documents en mémoire.

Les _arbres_ sont créés explicitement et supprimés explicitement: un arbre vide peut exister.

### Lecture d'un arbre
La lecture d'un arbre retourne les documents de l'arbre:
- une lecture **partielle** ne retourne que les documents dont les types sont listés dans la demande.
- une lecture **incrémentale** ne retourne que les documents dont la version est supérieure à celle passée en argument dans la demande.

Ces lectures sont _consistantes_, correspondent à un état cohérent des données dans la base de données.

### Lecture D'UN document
La demande spécifie l'identifiant du document.

### Désérialisation de la propriété `data` du document
Elle consiste à retourner une _map_ nom, valeur des propriétés du document, dont celles d'identification et la version.

La couche applicative est en charge de créer une instance de la classe appropriée depuis cette _map_ en utilisant le type de l'arbre, le type du document et si nécessaire d'autres propriétés de _data_.

### Lecture d'un fichier
Elle peut s'effectuer de deux manières:
- en retournant le contenu binaire du fichier dans la couche applicative,
- en retournant une URL d'accès sécurisé valable un certain temps, typiquement à transmettre à une application externe.

### L'écriture d'un fichier
En deux phases (deux transactions):
- préchargement du contenu du fichier en Storage:
  - directement en fournissant le contenu,
  - indirectement par retour d'une URL sécurisé permettant à une application externe d'effectuer _l'upload_ direct dans le Storage sans faire transiter le contenu du fichier dans la Cloud Function.

### Contrôle d'habilitation
Tous les appels de l'API se font dans le cadre d'une requête contrôlée par une transaction du SGBD.

Cette requête a systématiquement un contexte avec un objet `Account` géré par l'application:
- soit un document `Account` d'un arbre,
- soit un `Account` _anonyme_ donnant peu d'autorisations.

Cet objet est systématiquement consulté à chaque appel de l'API (lecture comme écriture) et peut lever une exception applicative pour refus de l'accès demandé.

### _Périmètre_ d'un `Account`
Un Account représente un utilisateur, humain ou non, identifiable et authentifiable.

L'objet Account est responsable de la gestion et détention du _périmètre_ d'intérêt de son utilisateur. C'est une liste d'identifiants d'arbres `treetype itd` donnant pour chacun la liste des types de documents qui l'intéresse.

A la fin d'une transaction le framework connaît:
- le périmètre retourné par l'objet Account sous le contrôle duquel la transaction s'est exécuté.
- la liste des documents créés / modifiés / supprimés. Chacun concerne un arbre et a un type de document.

Une liste est construite indiquant pour chaque arbre du périmètre dont un des objets a été mis à jour, les couples (type de document, version) contenant un des objets mis à jour.

Cette liste est retourné comme résultat de la requête: l'application distante ayant sollicité une opération reçoit donc en retour tous les **avis de changement** ayant affecté son _périmètre_. C'est ensuite à l'application de solliciter par des transactions ultérieures les mises à jour effectives des arbres concernés.

#### Cohérence _forte_ ans un arbre, _faible_ entre arbres
L'état d'un arbre retourné par une requête est _fortement cohérent_: cette configuration a existé vraiment à un moment donné.

Mais deux demandes faites pour deux arbres à des instants différents retourne deux états d'arbres qui ont pu ne jamais exister conjointement: il en résulte une _cohérence faible_ entre arbres, un éta qui globalement peut être fonctionnellement incohérent temporairement.

> On peut certes grouper dans la même requête des demandes concernant plusieurs arbres: toutefois le volume correspondant retourné peut être important et la transaction correspondante de collecte être longue et induire des blocages techniques de la base de données. Il y a applicativement un compromis à choisir entre _force de la cohérence entre arbres_ et lourdeur technique.

## Gestion de la synchronisation de copies distantes des arbres

Des applications distantes et multiples peuvent détenir des _copies partielles_ des arbres stockés en central:
- seulement certains arbres cités par leurs identifiants;
- dans ces arbres, seulement des types de documents souhaités.

Le framework met à disposition une couche logicielle aidant à maintenir à jour des copies locales d'arbres:
- en interprétant les notifications de changement des arbres qui proviennent,
  - soit d'un retour d'une opération de mise à jour sollicitée par l'application,
  - soit d'une notification de changement _poussée_ par une Cloud Function suite à des mises effectuées sur demande d'autres applications (typiquement des sessions Web d'autres utilisateurs).
- en sollicitant une Cloud Function pour obtenir les mises à jour des arbres qui ont été annoncés modifiés par ces notifications. En retour, les documents correspondants sont désérialisés et transformés en objets de classes applicatives puis transmis à l'application qui peut:
  - les ranger dans des mémoires applicatives, le cas échéant des mémoires réactives pouvant mettre à jour un état UI.
  - les stocker dans une base de données locales, typiquement une base IDB pour une application Web, une base SQLite pour une application mobile.

Le contenu des arbres ainsi stockés localement peut être consulté _offline_, quand l'application n'est pas connectée au réseau et ne fait pas appel aux Cloud Functions.

## Notifications _poussées_ par un _serveur_ aux applications clientes
Le troisième objectif est de conférer au _serveur_ la possibilité de _pousser des notifications_ vers des applications clientes:
- chaque application cliente peut recevoir des avis de modification du périmètre qui l'intéresse,
- elle peut ainsi,
  - avertir l'utilisateur par un message,
  - faire rafraîchir une copie plus ou moins partielle de son périmètre et afficher automatiquement l'état le plus récent de certaines données, même quand ces changements ont été issus d'autres sessions de travail d'autres utilisateurs.

# Applications clientes
## PWA : _progressive web app_
Une telle application s'exécute dans le contexte d'un browser mais a sa propre fenêtre:
- l'application (son logiciel) est identifiée par son URL.
- une instance d'application correspond à une installation dans un browser sur un device donné (par exemple une sous Chrome et une sur Edge).

Pour simplifier il n'y a à un instant donné au plus une instance d'application donnée sur un device donné.

> Rien n'interdit de mettre en ligne le même logiciel (au détail près de son `manifest`) sous 3 URLs: dans ce cas on peut ouvrir une fois ... 3 applications identiques (au nom près).

## Application Mobile (androïd)
Il n'y a au plus qu'une instance d'une application donnée sur un mobile donné. Elle a pu être lancée au démarrage du mobile et s'exécuter en _background_.

> Remarque: si une application s'exécute sous l'habilitation d'un _user authentifié_ elle doit prévoir:
- soit de connecter à un _user_ puis se déconnecter avant de se connecter à un autre.
- soit de gérer autant de contextes qu'il y a de users connectés en même temps dans l'application.

## Token
Une **instance d'application** distante (Web ou mobile) est identifiée par un `token` qui est une adresse à laquelle des messages pourront être poussés depuis un serveur (Cloud Function, ...) afin d'alerter l'instance de l'application sur l'évolution des documents de son, ou **ses** _périmètres_ d'intérêt.

Une instance d'application peut être en état,
- _foreground_: elle est visible par l'utilisateur et a le focus. Pour une application PWA sa fenêtre est sur le dessus et a le focus. Dans cet état l'application reçoit immédiatement les messages poussés et peuvent mettre à jour leurs copies locales de documents et leur UI.
- _background_: l'application n'est pas visible, cachée, voire même pas lancée. Dans cet état l'arrivée d'un message poussé provoque quand l'utilisateur clique sur la notification qui s'affiche,
  - si l'application est en exécution, son pop au premier plan,
  - si elle ne l'est pas son lancement.

## Authentification
Après avoir été lancée une application (PWA par exemple) peut soumettre des requêtes à son serveur (ou un de _ses_ serveurs) mais ne pourra pas accéder à beaucoup de données, la quasi totalité d'entre elles requérant un `Account` authentifié.

S'authentifier auprès d'un serveur consiste à lui soumettre des données de _credential_ (login / password, phrase secrète, etc.) celui-ci retournant à l'application en cas de succès un document `Account` portant a minima les propriétés:
- `id` : l'identifiant permanent du compte,
- `perimeter` : le périmètre du compte, une liste d'identifiants d'arbres `treetype itd` donnant pour chacun la liste des types de documents qui l'intéressent.

### Restrictions de périmètre
Le périmètre par défaut d'un `Account` est son périmètre le plus large, celui pour lequel l'utilisateur a le droit de consultation du maximum de documents.

Plusieurs applications peuvent se référer à un même `Account` mais avec des restrictions de périmètre:
- une application de mise à jour peut avoir le périmètre sans restriction,
- une application de monitoring peut avoir un périmètre réduit à certains types de documents, voire à des arbres ayant un certain profil.

Après authentification l'application peut si nécessaire:
- avoir un dialogue avec l'utilisateur afin de fixer un objet `options`.
- calculer un périmètre réduit depuis `Account` et `options`.

L'application va faire soumettre une _souscription_ auprès du serveur avec les données suivantes:
- son `token`: il sera utiliser par le serveur pour pousser des messages d'avis d'évolution de documents.
- **lURL de l'application** pour une application PWA.
- **son ou ses périmètres** avec pour chacun:
  - l'id de son `Account`,
  - le périmètre à notifier: pour chaque arbre `typetree tid` la liste des _types de documents_ à notifier.

La souscription est enregistré en base de données:
- sa clé primaire / path est son token,
- une date-heure est enregistrée afin de pouvoir purger les tokens inutilisés / non rafraîchis.

### Utilisation par le serveur
A la fin de chaque opération, le framework dispose de la liste des documents mis à jour avec leur arbre.
- pour chaque arbre mis à jour il obtient les souscriptions l'ayant dans leur périmètre et leurs types de documents surveillés,
- il peut établir pour chaque token un message signalant les mises à jour du ou des périmètres surveillés.

### Problème
Une application cliente pourrait se fabriquer un périmètre hors de ses limites.

Il faut faire calculer le périmètres sur le serveur et le retourner scellés / cryptés par la clé du serveur.

