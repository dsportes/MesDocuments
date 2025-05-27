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
C'est l'application qui appelle un serveur identifié par son URL: le serveur **traite la requête et retourne un résultat**.
- requêtes et réponses peuvent être volumineuses.

### Une application peut _écouter_ des notifications émises par les serveurs
Une application donnée sur un appareil donné est identifiée par un _jeton_ qui une sorte de numéro de téléphone universel: tout serveur ayant connaissance de ce jeton peut envoyer des _notifications_ à l'application correspondante sur le poste correspondant.

Une notification ressemble à un SMS:
- son texte est _court_ (certes plus long que celui d'un SMS).
- on ne répond pas à une notification: le serveur émetteur ne sait rien de la suite donnée, ou non, par l'application destinataire.

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
Quand une application est en exécution elle peut rester _abonnée_ à des _fils d'information_.

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
  - c'est un appareil de confiance: il peut laisser quelques informations personnelles localement dessus (mais cryptées).
- **l'appareil qu'il s'apprête à utiliser ne lui est pas familier**, il est partagé par un grand nombre d'utilisateurs comme au cyber-café ou est celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelques informations que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement.

### Appareil _personnel_: ses profils
En pratique c'est en général un appareil qui peut être utilisé par d'autres que soi-même. Typiquement dans un cadre familial ou un couple, l'appareil n'est pas strictement _personnel_.

En général le _login_ à un tel appareil est _protégé_ par un mot de passe ou tout autre dispositif que seuls des proches connaissent (à moins d'ailleurs que l'appareil leur soit prêté avec une session ouverte). 

Il n'empêche que _plusieurs personnes peuvent plus ou moins occasionnellement s'en servir_: d'où le principe d'y avoir possiblement plusieurs _profils_.
- un browser comme Firefox a une notion d'utilisateur: on peut basculer d'un utilisateur à un autre (sans pour autant avoir changé de connexion au niveau de l'OS), ce qui va conférer à chacun ses sites favoris, son historique de navigation et ses mots de passe enregistrés.
- Thunderbird, le gestionnaire de mails locaux, supporte de gérer plusieurs _profils_, chacun avec ses comptes mails.

### Des _profils_ plus ou moins bien défendus
Ce n'est pas parce qu'on partage un appareil avec un proche qu'on a envie de partager avec lui ses informations confidentielles.

Or dans les cas cités ci-dessus, la confidentialité est plutôt _lâche_:
- Thunderbird ne demande rien: on choisit son profil, sans mot de passe ou quoi que ce soit. Les boîtes mail sont de toutes les façons en clair dans le file-system, confidentialité _intra-familiale_ zéro.
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

# Données d'un service
Un _service donné pour un prestataire donné_ dispose de deux entités de stockage dédiées:
- **UNE Base de données** pour toutes les données devant être gérées par des transactions.
- **UN Storage de fichiers**, non géré par des transactions mais dont la sécurité transactionnelle peut être assurée par la base de données avec un protocole léger pré-validation / validation. Il stocke des _fichiers_ identifiés par leur _path_: la présence de caractères `/` dans un path définit une sorte d'arborescence. Le _contenu_ de chaque fichier est une suite d'octets opaque.

Le Storage permet de disposer d'un volume pratiquement 10 fois plus importants à coût identique par rapport à la base de données: de nombreuses applications ont des données historiques / mortes ou d'évolutions sporadiques qui s’accommodent bien d'un support sur Storage.

> La Base et le Storage sont gérées et accédées exclusivement par le prestataire du service.

#### Le répertoire des applications terminales _abonnées_
Chaque application sur un appareil a un _jeton_ qui l'identifie de manière unique. Ce répertoire assure la gestion des applications _actives_ (ayant un abonnement en cours):
- maintien du répertoire à jour et détection des applications inactives: une application terminale _s'abonne_ auprès du prestataire pour toutes les organisations gérées par le prestataire et faisant partie du domaine d'intérêt de l'utilisateur de l'application.
- pour chaque application _active_, le répertoire détient la liste des identifiants des _dossiers à synchroniser_ et des _flux de news_ auxquels elle est _abonnée_.
- à l'occasion d'une évolution des données, une publication de notifications est déclenchée vers toutes les applications abonnées à un des flux / dossiers mis à jour.

> Chaque application terminale est en conséquence susceptible de s'abonner éventuellement auprès de plus d'un prestataire si toutes les organisations de son domaine d'intérêt ne sont pas toutes gérées par le même prestataire.

# Base de données partagée par tous les prestataires
Cette base unique n'est concerne toutes les applications et tous les prestataires.

Tous les services des prestataires peuvent y accéder pour les opérations qui leur sont ouvertes.

Toutes les applications terminales peuvent solliciter un quelconque des sprestataires afin d'accéder concernant un de leurs utilisateurs.

Les données de cette base sont:
- Le **répertoire des services**.
  - il liste pour toutes les applications le prestataire (son URL) gérant chaque organisation.
- Le **répertoire des utilisateurs**. Tout utilisateur qui s'y est inscrit (c'est une facilité pas une obligation) dispose d'une entrée personnelle confidentielle et sécurisée. Il y trouve:
  - un enregistrement de _préférences_ personnelles.
  - des _droits d'accès scellés_ déposés par un service d'une application. Chaque _droit_ est spécifiquement attribués à cet utilisateur et lui confère la possibilité d'accéder à des données et de s'abonner à des fils de news.

#### Glossaire technique
- **SH(s1, s2)** (Strong Hash): le SH s'applique à un couple de textes `s1 s2`, typiquement un login / mot de passe, mais aussi aux _passphrase_ en une ou deux parties. Il a une logueur de 32 bytes et est unique pour chaque couple de textes `s1 s2`. Il est _strong_ parce qu'incassable par force brute dès lors que le couple de textes ne fait pas partie des _dictionnaires_ des codes fréquement utilisés.
- **PP-x** : couples de clés publique / privée (Pub / Priv).
- **K-x** : clés AES de 32 bytes.

Soit `PP-S` le couple de clés généré par un serveur:
- `Pub-S` est une clé publique: toutes les applications l'obtiennent libremenet par un simple GET au serveur.
- `Priv-S` est la clé privée du serveur et fait partie des _secrets_ du logiciel serveur.

Soit `PP-ti` un couple de clés généré par une application terminale `ti` pour une conversation donnée avec le serveur `S`:
- `Pub-ti` est transmise dans les requêtes au serveur.
- Le serveur peut génèrer une clé `K-S-ti` à parir du couple `Pub-ti / Priv-S`: il s'en sert pour crypter ses réponses à l'application terminale ou toute donnée dont il veut que seule `ti` puisse les lire.
- L'application terminale peut génèrer une clé `K-ti-S` à parir du couple `Pub-S / Priv-ti`. Comme `K-ti-S` et `K-S-ti` sont égales, l'application terminale peut s'en serir pour décrypter les réponses / données cryptées pour elle par l'application serveur (et nul autre ne peut le faire).

Les applications terminales connaissent la clé publique du serveur Pub-S. Quand l'une d'elle veut échanger des données confidentielles avec le serveur:
- elle génère un couple PP-t,
- elle envoie ses requêtes au serveur en fournissant Pub-t.
- elle décryptera les réponses / données cryptées par le serveur par la clé Pub-S.

## Répertoire des services
Ce répertoire est en deux parties.

#### Liste des _application / prestataire_
Cette liste donne pour chaque _service_ identifié par le couple **application / prestataire**  
- son **URL** d'accès qui permet de lui adresser des requêtes.
- sa **clé publique** PUB-S de cryptage qui permet à qui le souhaite de crypter des données de manière à ce que seul les serveurs de ce service puissent les décrypter en sachant qui les a crypté.
- son **statut** à propos de l'état du service (ouvert / restreint / suspendu / fermé) et un _texte d'explication_ de cet état.
- un court texte d'information à propos du service.

#### Liste des _organisations par application_
Pour chaque couple **application / organisation**, cette liste donne:
- le code du **prestataire** qui la sert.
- un **statut** à propos de l'état du service vis à vis de l'organisation (ouvert / restreint / suspendu / fermé) et un texte éventuel d'explication de cet état.
- un court texte d'information à propos de l'organisation.

Quelques opérations de gestion sont proposées aux prestataires pour:
- s'enregistrer, donner son URL et sa clé publique, maintenir à jour son statut et son texte d'information.
- enregistrer une nouvelle organisation s'assurant de son unicité et lui attribuer / modifier son label, fixer son statut et son texte d'information.

Ce répertoire permet aux applications terminales:
- d'obtenir l'URL d'appel du service gestionnaire en fonction de l'organisation et sa clé publique de cryptage,
- de lire le statut du service et ses restrictions d'usage éventuelles fixées par l'adminisrateur du service à l'égard de leur organisation.

## Répertoire des utilisateurs
Chaque utilisateur **_PEUT_** et non _DOIT_ s'enregistrer dans ce répertoire dans le seul but de raccourcir les saisies d'information à fournir pour accéder aux applications de son choix. Il y est identifié par un USERID généré aléatoirement et sans signification.

Il y a deux natures d'informations qui peuvent être stockées dans l'entrée de l'utilisateur:
- des _droits d'accès_,
- des _préférences_.

#### Droit d'accès
Un droit d'accès est enregistré par un service pour l'utilisateur et est une donnée cryptée qui _autorise_ l'utilisateur à accéder,
- à des documents identifiés, voire à s'abonner aux évolutions de certains d'entre eux,
- à des flux de news identifiés.

Un droit d'accès contient deux parties:
- une partie **cible** interprétable par les applications terminales comme les serveurs:
  - à qui il est attribué (USERID),
  - pour quel _usage_ dont la codification est spécifique de chaque application.
- un **jeton opaque** pour les applications terminales, crypté par le serveur, donnant les authentifications nécessaires à accéder aux documents et fils de news.

Les services génèrent des _droits d'accès_: ceux-ci sont scellés. Les applications terminales peuvent:
- les obtenir des services et les lire mais sans pouvoir interpréter le jeton opaque.
- les enregistrer dans leur entrée de répertoire.
- ultérieurement les relire et les transmettre à un service pour bénéficier de l'accès correspondant.

#### Préférences
Une _préférence_ est une donnée nommée pour laquelle l'utilisateur a donné une ou des valeurs par défaut / préférées:
- langue préférée,
- mode sombre / clair,
- nom, e-mail, adresses, numéros de téléphone ...
- etc.

Quand une application demande l'une de ces informations, il est proposé à l'utilisateur en pré-saisie la valeur ou l'une valeurs inscrites en _préférences_ si elle y figure. L'utilisateur n'est pas obligé de s'y conformer et peut toujours fixer sa propre valeur à cet instant.

> Le défi est de garantir une stricte confidentialité de ces droits d'accès et préférences: a) seul l'utilisateur peut les obtenir et les changer, b) aucun serveur ne doit jamais être en mesure de les accéder, ni en lecture, ni en écriture.

#### Auto-enregistrement d'un utilisateur

Chaque _utilisateur_ peut s'enregistrer dans ce répertoire afin d'y disposer d'une entrée personnelle: il lui faut toutefois avoir été _parrainé_ dans le cadre d'une application qui lui a fourni un _jeton de parrainage_:
- **soit par un intermédiaire humain** qui lui a communiqué ce jeton sous forme d'une _phrase de parrainage_ par le canal humain de son choix.
- **soit par un code envoyé par le serveur** par exemple une moitié du code envoyée par mail à l'adresse donnée par l'utilisateur, la seconde passée en _notification_ sur l'appareil de l'utilisateur.

Une demande de parrainage est gérée par le service auquel l'utilisateur s'adresse. En général ceci va passer par un _formulaire_, minimal ou non, par lequel le service va juger de la pertinence de cette demande et si elle doit ou non corrélativement accorder un droit d'accès à l'application pour l'utilisateur demandeur.
- la _vraie_ demande de parrainage peut avoir été faite _humainement_ par la rencontre d'un parrain et d'un parrainé, le premier ayant à la suite de cette rencontre enregistré un _parrainage_ et communiqué le jeton au parrainé par le canal de son choix.
- une demande peut aussi être établie par un formulaire de demande, traitée sur l'instant ou non: le cas échéant ultérieurement un traitement différé peut générer les _parrainages_ par lot et en diffuser le résultat par mail.

> Ce protocole est fait pour éviter une inflation non contrôlée d'enregistrements sans objectifs d'accès aux applications.

Chaque application est responsable de sa logique de distribution de ses _jetons de parrainage_:
- le `USERID` de l'utilisateur est généré par l'application distributrice (long, aléatoire et sans signification).
- un `statut` _en création_ est fixé avec une date et heure limite de validité:
  - si l'utilisateur s'enregistre avant cette limite, le statut passe à _actif_, sinon l'enregistrement est supprimé à échéance.
- une `phrase ou jeton de parrainage` ouvrant un droit d'inscription au répertoire à une personne la connaissant. C'est le SHA du SH de cette phrase / jeton qui est enregistré.
- un `message d'information / bienvenue` éventuel: ce texte n'est pas forcément uniquement de _politesse_ et peut contenir des conditions techniques attachées à cette inscription.
- un `droit d'accès` éventuel permettant à l'utilisateur, en cas d'acceptation, de disposer d'un accès à l'application à laquelle il a demandé de parrainer sont enregistrement.
- deux URLs:
  - l'une à utiliser **après acceptation** et enregistrement: l'utilisateur pourra accompagner cet avis d'acceptation par un message de remerciement (ou non) et joindre le _droit d'accès_ attaché.
  - l'autre à utiliser **en cas de renoncement** d'enregistrement. Au vu du message d'information / bienvenue, ou pour toute raison, l'utilisateur peut renoncer et peut utiliser cette URL pour expliquer le motif de renoncement ou toute autre formule de politesse.

> Le cas échéant un _enregistrement_ dans le répertoire des utilisateurs peut également fournir conjointement un accès pour cet utilisateur au service.

Un utilisateur peut s'enregistrer:
- en fournissant le SH de la `phrase / jeton de parrainage` qui lui a été communiquée.
- en ayant généré une clé AES de cryptage `Kp`.
- en ayant généré un **couple de clés publique / privée** crypté par la clé Kp ci-dessus.
- en donnant **deux alias d'accès** qui lui permettront de s'identifier en tant qu'utilisateur:
  - un seul fait courir le risque de le perdre,
  - un second de _secours_ limite ce risque. Avec un des deux alias l'utilisateur pourra réinitialiser l'autre s'il est définitivement oublié ou s'il souhaite en changer.

Un alias d'accès est constitué de:
- `s1` : un texte identifiant (d'une longueur minimale) choisi par l'utilisateur.
- `s2` : un texte d'authentification (d'une longueur minimale) choisi aussi par l'utilisateur.
- la clé `Kp` cryptée par `s1 + s2`.

> L'utilisateur est le seul à connaître, dans sa tête, le couple `s1 s2`, jamais mémorisé nulle part et transmis seulement dans des SH.

#### Remarques techniques
- l'application terminale d'enregistrement allonge `s1` en `s1+` par un texte de remplissage quand sa longueur est inférieure au maximum mais supérieure au minimum (refus si inférieur à la longueur minimale). L'unicité du `SH(s1+, s1+)` est vérifiée.
- l'application terminale d'enregistrement allonge `s2` en `s2+` de même, mais le texte de remplissage est généré en fonction de `s1`.
- l'unicité de `s1` est vérifiée par l'enregistrement du `SHA(SH(s1+, s1+))`.
- l'unicité de `s2` est vérifiée par l'enregistrement du `SHA(SH(s1+, s2+))`.

#### Accès par l'utilisateur à son enregistrement
Désormais l'utilisateur est enregistré dans le répertoire des utilisateurs:
- en lui demandant l'un de ses couples d'accès `s1 s2`, l'application terminale peut fournir les couples `SH(s1+, s1+)` et `SH(s1+, s2+)`. 
- le serveur peut par `SHA(SH(s1+, s1+))` accéder à l'entrée `USERID` pour cet utilisateur et vérifier la validité de `SH(s1+, s2+)` pour cet `USERID`. Il peut retourner,
  - la clé `Kp` cryptée par `s1 + s2` (et qu'il est incapable de décrypter faute de connaître s1 et s2).
  - le couple de clés privée / publique que l'application terminale peut décrypter puisqu'ayant décrypter Kp.
  - le set des _préférences_ de l'utilisateur cryptées par cette clé `Kp`.
  - la liste des _droits d'accès_ enregistrés, dont il peut lire et interpréter une partie mais ne peut pas ni interpréter ni modifier la partie _opaque_ générée par les serveurs.

L'application dispose ainsi en mémoire d'une _fiche en clair_ représentant le décryptage de l'entrée cryptée de l'utilisateur dans le répertoire des utilisateurs.

L'application terminale peut ainsi désormais:
- mettre à jour ses _préférences_ et les enregistrer cryptées par la clé `Kp` qu'il vient de récupérer.
- pour chaque droit,
  - l'effacer le cas échéant,
  - le _commenter_ et le réenregistrer crypté par la clé `Kp` qu'il vient de récupérer. Un commentaire permet à l'utilisateur d'interpréter plus aisément l'usage du droit dont la codification peut être assez absconse.

#### Sauvegarde _locale_ de la fiche de l'utilisateur
L'utilisateur _peut_ sauvegarder localement cette fiche dans un stockage local avec plusieurs conditions:
- fournir un code local de _profil_, par exemple `bob`,
- fournir un couple `s1 s2` pour en crypter le contenu stocké dans le _storage_ local de son browser.

> L'utilisateur peut aussi demander _l'impression_ en HTML de sa fiche en clair ... qui dans cet état est utilisable par tout hacker un peu débrouillé, mais permet aussi à l'utilisateur de la stocker en lieu sûr hors de l'application.

## Profils locaux sur des appareils _personnels_
L'accès à sa _fiche personnelle_ cryptée dans le répertoire des utilisateurs est un service générique fourni par toute les applications et cette fiche va lui faciliter la vie au cours des accès aux applications.

**MAIS pour y accéder l'utilisateur aura dû fournir un couple `s1 s2` ce qui représente un texte long et le cas échéant fastidieux à saisir.**

D'où l'idée que sur un appareil _personnel_ il serait souhaitable de pouvoir être possible d'accéder à sa _fiche personnelle_ en s'identifiant de manière plus raccourcie, et ce sans (trop) compromettre la confidentialité qui était garantie par la longueur de `s1 s2`.

### Sur un appareil donné: nom local de _profil_ et son code PIN
Une application sur un appareil est identifiée par un _jeton_ technique qui permet aux serveurs de lui pousser des notifications.

Pour qu'une application s'exécute sur un appareil, il faut qu'une session de l'OS y ait été ouverte et avoir fourni a minima un mot de passe ou une empreinte digitale, bref avoir réussi une première authentification _personnelle_. 

> Si le login correspond à un compte _invité_ le dispositif ci-dessous **NE DOIT PAS** être employé: le _login_ de l'OS n'offre aucune garantie.

Un utilisateur peut sur un poste personnel déclarer l'ouverture d'un _profil_ qui lui est propre et lui donner un nom local, par exemple `bob`.
- ce nom N'EST PAS confidentiel.
- ultérieurement un utilisateur peut même choisir son profil dans la liste de ceux enregistrés sur le poste.

Lors de l'enregistrement de son profil `bob` sur un appareil l'utilisateur va déclarer:
- un `code PIN`, d'une taille minimale de 8 signes: voire plus avant l'importance de code.
- un des couples `s1 s2` d'accès à sa fiche dans le répertoire des utilisateurs.

Si le couple d'accès `s1 s2` est correct, la _fiche de l'utilisateur_ est en mémoire, sa clé Kp est connue. L'application peut alors faire enregistrer un nouveau _profil_ dans son entrée de répertoire:
- l'application sur l'appareil est identifiée par son _token_.
- cette entrée est identifiée par le SH(token, profil `bob`).
- un compteur d'échec de code PIN est initialisé à 0.
- la clé Kp cryptée le SH(code PIN _bruité_).

### Accès à sa _fiche_ par un utilisateur depuis son appareil
En fournissant son code de profil bob et son code PIN, l'application:
- retrouve l'entrée de l'utilisateur dans le le répertoire en fournissant le SH(_token_, profil).
- elle tente de décrypter la clé Kp depuis le code PIN: en cas de succès, la clé Kp est retourné ce qui permet à l'application de décrypter la fiche de l'utilisateur, le compteur d'échec est mis à 0 s'il ne l'était pas déjà.

En cas d'échec le nombre d'échec dans la fiche est incrémenté: **au second échec, l'entrée pour ce token et ce profil est détruite.**

### Principes de sécurité
Le code PIN **N'EST PAS** stocké localement: un voleur / hacker ne peut donc pas le retrouver. Le code PIN n'est présent que:
- en clair dans la tête de l'utilisateur (qui certes doit éviter de l'inscrire au feutre sur son appareil),
- sous forme Strong Hash dans le serveur.

Toutefois même si le hash est `fort`, le code PIN pouvant être court, le retrouver par _force brute_ (en essayant toutes les combinaisons possibles) reste potentiellement faisable.

> A noter que pour accéder à ce code PIN crypté dans le répertoire des utilisateurs il faut disposer du _token_ de l'application sur l'appareil ce qui a obligé à, a) dérober l'appareil, b) y lancer l'application, c) accéder à ce token en _debug_.

**Mais le serveur détruit l'entrée de `bob` au second essai infructueux** d'un code PIN.

La découverte par _force brute_ est donc impossible.

### Découverte par complicité
L'administrateur du serveur protège l'accès à la base de données et les données de celle-ci sont cryptées par une clé d'administration. Pour _décrypter les enregistrements de la base_ il faut donc,
- a) avoir accès à la base,
- b) avoir la clé de l'administrateur.

En supposant avoir les deux par complicité ou contrainte, un hacker peut obtenir le Strong Hash des codes PIN, en particulier celui de `bob`. Avec beaucoup de temps calcul, le code étant (relativement) court, sur un appareil quelconque, il arrivera à trouver ce code après un nombre d'essais faramineux mais que plus rien ne l'empêche de faire.

En conséquence pour casser le code PIN de `bob` sur son appareil, un hacker doit:
- dérober l'appareil pour en obtenir le _token_ de l'application,
- avoir la complicité de l'administration technique du serveur,
- avoir de gros moyens informatiques.

Cette triple condition est déjà un handicap sérieux ... et demande beaucoup d'argent et / ou l'usage de la force physique sur les humains. Si ce dernier cas est envisageable, le moins coûteux est de _persuader_ `bob` de donner son code PIN !

### _Dureté_ du code PIN
Avec un code PIN `1234` et autres vedettes des mots de passe friables, l'effort ne devrait pas durer longtemps.

Toutefois UN SEUL essai d'un code demande un temps calcul important, le Strong Hash n'est _strong_ que parce qu'il exige du temps calcul non parallélisable et inapte à bénéficier de processeurs dits _graphiques_.

Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il peut être incassable: pour être mnémotechnique certes il va s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. mais il y a N façons de saisir `allons enfants de la`, avec ou sans séparateurs, des chiffres au milieu, des alternances de minuscules / majuscules. Il est difficilement concevables de coder l'inventivité des variantes, sans compter le nombre énorme de variantes possibles à exécuter à partir d'une seule _idée_ de texte de longueur inconnue.

## Accès en mode _avion_ à un profil (A AFFINER)
En mode _avion_ il n'y a pas d'accès à Internet: la fiche d'un utilisateur ne peut pas être obtenue depuis un nom de profil et un code PIN.

Si la fiche de l'utilisateur a été enregistrée localement, elle est lisible en fournissant:
- le code du profil sous lequel elle a été enregistrée.
- le couple `s1 s2` de textes qui la crypte.

Les stockages locaux de document sont stockés où, cryptés par quoi ? La clé Kp ? Si oui l'enregistrement de l'utilisateur en répertoire est obligatoire par accéder au mode avion.

-----------------------------------------------------------------------

# Contributions antérieures


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

#### Cohérence _forte_ dans un arbre, _faible_ entre arbres
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

# Use Case _circuit court_

## Concepts
Un groupe de consommateurs est identifié par son code gc. Sa fiche de renseignement donne des informations de contact, son ou ses mots de passe et spécifie la liste des groupements de producteurs auxquels il peut commander. 

Un consommateur est identifié par son code dans son groupe: gc co. Sa fiche de renseignement donne des informations de contact et son mot de passe.

Un groupement de producteurs est identifié par son code gp. Sa fiche de renseignement donne des informations de contact, son ou ses mots de passe et spécifie la liste des groupes de consommateurs qui peuvent lui émettre des commandes.

Un producteur est identifié par son code dans son groupement: gp pr. Sa fiche de renseignement donne des informations de contact et son mot de passe.

Une référence de produit est identifié par son code dans son producteur: gp pr rp.

Une livraison d'un groupement est identifiée par le groupement livrant et sa date de livraison: gp livr. La date est immuable une fois déclarée, mais des dates effectives peuvent être déclarées et changées:
- date-heure d'ouverture: on ne peut pas commander avant cette date. Les conditions de vente (prix / poids des produits) sont fixées mais peuvent subir des variations après ouverture. Dans ce cas les conditions à l'ouverture et les conditions courantes existent toutes les deux.
- date-heure de clôture: on ne peut plus commander après cette date-heure.
- date-heure d'expédition. Les conditions prix / poids des produits ne peuvent plus changer après cet instant, des paiements effectifs et définitifs peuvent s'engager.
- date-heure de livraison: il y a une date-heure par point de livraison au groupe de consommateur.
- date-heure de distribution: il y a une date-heure par point de livraison au groupe de consommateur, les produits peuvent commencer à être récupérés.
- date-heure de clôture: plus rien ne peut changer, les rectifications ultérieures sont traitées manuellement, la livraison est archivée.

Les commandes par les consommateurs indiquent une quantité pour chaque produit. 
- La quantité peut être en unité ou en poids au Kg. 
- Les unités peuvent être des _demi_ (une demi caisse) selon les produits (pas de _demi_ paquet de café).

La _commande_ d'un groupe de consommateurs est la somme des commandes des consommateurs du groupe.

Quand une commande est réceptionnée, pour chaque produit (en général commandé mais pas forcément), figure une quantité livrée ou un poids livré.
- pour certains produits, des poulets par exemple, la quantité livrée est le nombre N de poulets et le poids livré est un tableau de N poids individuels des poulets. Quand il y a un poids total c'est, soit temporaire en estimation avant obtention des poids individuels, soit la somme des poids individuels.

Le calendrier des livraisons d'un groupement de producteurs donne pour chaque livraison:
- sa date de livraison (théorique et immuable).
- ses dates-heures d'ouverture, expédition ... Des règles fixent comment et quand ces dates peuvent être changées, en particulier les unes par rapports aux autres.
- la liste des groupes de consommateurs livrés (point de livraison).

La catalogue des produits d'un groupement
Il est constitué de 2 documents:
- un catalogue général. Il donne pour chaque produit:
  - un descriptif permanent: 
    - son code et sa référence (_code-barre_).
    - un descriptif (libellé) et des indicateurs de label / qualité, taux de TVA applicable, si c'est un produit _sec_, _frais_ ou _surgelé_.
    - si le produit est vendu à l'unité, au poids et / ou par _caisse_ ou _demi-caisse_.
    - ces caractéristiques ne peuvent pas changer, sauf rectifications orthographiques / documentaires (un produit _à l'unité_ ne peut pas devenir _au poids_, il faut définir un autre produit).
  - une liste chronologique de date-heure à laquelle les conditions de ventes du produit ont changé:
  - sa disponibilité,
  - son prix,
  - ses poids _net_ et _brut_: le poids _net_ est celui sans l'emballage (ce que mange le consommateur), son poids _brut_ inclut l'emballage (ce que ça pèse dans le camion).

- un catalogue par livraison. Il donne pour une livraison donnée, la liste des produits avec pour chacun sa conditions de vente.
  - la catalogue d'une livraison est _calculé_ avant ouverture de la livraison depuis le catalogue général.
  - il peut être amendé ponctuellement jusqu'à l'ouverture de la commande.
  - après ouverture de la commande, quand une condition de vente d'un produit change, les deux conditions existent: celle _actuelle_ et celle _à l'ouverture de la commande_.
  - les conditions de vente ne peuvent plus changer après la date-heure de livraison (des paiements définitifs ont pu avoir lieu).

News / chats par livraison
- une news / chat alimentée par chaque groupement de producteurs: gp livr. Tous les groupes de consommateurs sont concernés. Un groupe de consommateurs peuvent y déposer aussi des news (avec modération).
- une news alimentée par chaque groupe de consommateur: gc livr. Tous les consommateurs du groupe sont concernés. Un groupement de producteur est également habilité à y déposer des news.

News / chats par groupe de consommateurs: écrits par le groupe et les consommateurs du groupe indépendamment de toute livraison.

News / chats par groupements de producteurs: écrits par le groupement et les producteurs du groupement indépendamment de toute livraison.

> La partie de gestion des paiements est omise dans ce use-case.

> La partie de consultation de _l'historique_ figé des commandes clôturées est omise dans ce use-case.

## Objectifs
On s'intéresse ici aux deux scénarios simplifiés se rapportant à un groupement de consommateurs et à un consommateur. Des scénarios voisins

### Point de vue d'un consommateur.
Documents de son périmètre d'intérêt
- les calendriers généraux des livraisons: vision de _reporting_, il n'a pas à en suivre en temps-réel les variations avec zoom à la demande sur chaque calendrier.
- la liste des livraisons ouvertes avec leur statut et dates-heures: vison _synchronisée_ présentant pour chaque livraison la _synthèse de sa commande_ : nombre de références, somme des poids brut et somme des prix.
- il peut _zoomer sur une livraison et avoir une vision _synchronisée_ de la livraison:
  - le détail de sa commande,
  - la synthèse de la commande pour son groupe: en effet pour chaque produit un consommateur aime à savoir le montant global des autres membres du groupe.
- les catalogues généraux des produits de chaque groupement: vision _reporting_ avec zoom à la demande de chaque catalogue.
- les catalogues des produits spécifiques des livraisons ouvertes: vision _reporting_ avec zoom à la demande de chaque catalogue.
- chat de son groupe: avec vision _synchronisée_.

Fils de news
- chat de son groupe.
- chats des livraisons ouvertes. 

> Il ne suit pas par fils de news les évolutions tarifaires, les évolutions des dates, etc. C'est l'animateur du groupement qui en fera les informations de synthèses sur le chat du groupe.

### Point de vue d'un groupe (un de ses animateurs) (A SUIVRE).
Documents de son périmètre d'intérêt
- les calendriers généraux des livraisons: vision de _reporting_, il n'a pas à en suivre en temps-réel les variations avec zoom à la demande sur chaque calendrier.
- la liste des livraisons ouvertes avec leur statut et dates-heures: vison _synchronisée_ présentant pour chaque livraison la _synthèse de sa commande_ : nombre de références, somme des poids brut et somme des prix.
- il peut _zoomer sur une livraison et avoir une vision _synchronisée_ de la livraison:
  - le détail de sa commande,
  - la synthèse de la commande pour son groupe: en effet pour chaque produit un consommateur aime à savoir le montant global des autres membres du groupe.
- les catalogues généraux des produits de chaque groupement: vision _reporting_ avec zoom à la demande de chaque catalogue.
- les catalogues des produits spécifiques des livraisons ouvertes: vision _reporting_ avec zoom à la demande de chaque catalogue.
- chat de son groupe: avec vision _synchronisée_.

Fils de news
- chat de son groupe.
- chats des livraisons ouvertes. 

> Il ne suit pas par fils de news les évolutions tarifaires, les évolutions des dates, etc. C'est l'animateur du groupement qui en fera les informations de synthèses sur le chat du groupe.

