---
layout: page
title: Réflexions à propos d'un framework "Orienté document"
---

Le mot _application_ (Facebook, TikTok ...) désigne en réalité un système applicatif dont l'architecture schématique peut se résumer en deux niveaux:
- des _serveurs centraux_ détiennent les données et effectuent les calculs sollicités par des demandes émises par ...
- des _applications terminales_ s'exécutant sur les terminaux / appareils des utilisateurs et sollicitant les serveurs centraux.

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
- l'application _peut ou doit_ (selon le browser utilisé et l'OS de l'appareil) être _installée_ par le browser. Elle apparaît ensuite comme une application locale de l'appareil avec une icône de lancement, typiquement sur le bureau.

#### Application de type _mobile_
L'utilisateur l'installe depuis le ou un des magasins d'application supportés par l'OS du mobile.

> Il n'y a ensuite quasiment pas de différence perceptible par l'utilisateur à l'utilisation de l'application, il clique sur une icône pour l'ouvrir (la lancer).

> On peut installer une application Web-PWA sur un mobile: elle est dans la suite du document considérée comme application PWA (et non comme _mobile_).

### Serveurs
Les applications en exécution sur leur appareil envoient à des **serveurs centraux** des demandes de mise à jour et consultation des données de l'application.

Le terme générique de _serveurs_ recouvre des variantes techniques non perceptibles de l'extérieur:
- **Serveurs permanents**: plusieurs processus sont en exécution en permanence afin de traiter les requêtes qui leurs parviennent sur l'URL du pool de processus et ont été routées vers l'un ou l'autre.
- **Cloud Function**: un _serveur éphémère_ du _Cloud_ est lancé pour traiter une demande de service reçue sur son URL:
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
- enfin ils peuvent différer selon le prix de leurs prestations et leur qualité (temps de réponse, disponibilité, restrictions de volume, ...).

> Si le nom d'un service comme `randos` traduit sa fonctionnalité générale, un _service précis_ est désigné par le couple du **nom de service / nom du prestataire** le proposant : `randos/Rouge`. Ce couple se traduit par une URL qui permet de solliciter _ce_ service spécifiquement délivré par _ce_ prestataire.

# Les _applications_, leurs magasins, leurs variantes
Le logiciel d'une application terminale est disponible dans un magasin: dans le cas d'une application Web le magasin est un site Wen statique dont chaque URL correspond à une application.

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
Dans le cas de l'application `randos`, un utilisateur donné peut être membre de plus d'une association de randonneurs: une pour ses randonnées près de chez lui, une autre pour les randonnées de montagne et une troisième pour les treks lointains. Depuis la même application il peut basculer d'une organisation à une autre.

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

> L'application _peut_ en tenir compte et effectuer des traitements et des requêtes ultérieures aux serveurs.

**Quand l'application destinatrice d'une notification est au PREMIER PLAN:**
- elle peut afficher un court message dans une petite fenêtre _popup_ (voire émettre un son ...) pour alerter l'utilisateur,
- elle peut effectuer le traitement simple ou complexe adapté aux données portées par la notification.

**Quand l'application destinatrice est en ARRIÈRE PLAN:**
- elle peut (ou l'OS de l'appareil ou le browser dans lequel elle s'exécute) peut afficher en _popup_ la notification ce qui alerte l'utilisateur,
- si l'utilisateur clique sur cette _popup_, l'application correspondante repasse au premier plan.

**Quand l'application destinatrice N'EST PAS en exécution:**
- l'OS de l'appareil ou le browser dans lequel elle est enregistrée, peut afficher en _popup_ la notification ce qui alerte l'utilisateur,
- si l'utilisateur clique sur cette _popup_, l'application est lancée.

## Des applications _écoutantes_ réagissant au flux d'informations poussées
Les applications **_sourdes_** classiques ne peuvent afficher des écrans que sur sollicitation de l'utilisateur: l'écran ne se remet à jour que suite à une action de l'utilisateur:
- si ce dernier ne fait rien, l'écran ne change pas et affiche des données plus ou moins anciennes qui ont déjà été modifiées par l'action d'autres utilisateurs, du temps qui passe, etc.

Les applications **_écoutantes_** peuvent remettre à jour leurs écrans et données détenues localement même sans action d'un utilisateur simplement en fonction des _notifications_ poussées vers elles par les serveurs.

# Le paradigme _documents / fils de documents / news_
**Un _document_ contient un ensemble structuré de données:** la structure peut être complexe et un document peut être volumineux. Par exemple un _bon de commande_ d'un consommateur pour une livraison d'un jour donné.

**Chaque document _peut_ être attaché à un ou plusieurs _fils de document_**: en tirant sur le _fil_ de la livraison de samedi on tire tous les documents attachés: par exemple le détail de la livraison et tous les bons de commande attachés à cette livraison.

**Un _fil_ fait aussi office de _news_, d'alerte:** quand un document change (un bon de commande), le ou les fils auxquels il est attaché (la livraison de samedi)sont informés et le fil a noté qu'un bon de commande a changé. Si des applications s'étaient abonné à ce fil, leurs utilisateurs vont voir apparaître sur leurs écrans une _notification_ indiquant _qu'un bon de commande a changé pour la livraison de samedi_. Si l'application d'un utilisateur était lancée et que la page courante montrait cette livraison, l'application est allé chercher les bons de commande attachés au fil de la livraison de samedi ayant une version pplus récente que celle que l'application a en mémoire: la page est mise à jour à l'écran sans intervention de l'utilisateur.

Suivant ce paradigme, une application présente à son utilisateur trois concepts:
- des **_fils d'information_** annonçant des évolutions de documents ou de collections de documents qui l'intéresse: l'arrivée de nouveaux échanges sur un _chat_ (un document), uné évolution tarifaire (un tarif vu comme une collection de documents). Ces fils **annoncent** par des notifications courtes une évolution de certains documents, mais n'en donne q'un minimum d'information.
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
- un clic sur un ces _popups_ ouvre l'application ce qui lui permet de connaître en détail les documents ayant changé.

> Chaque application détermine pour chaque fil auquel elle est abonné, si l'abonnement s'interrompt ou non quand l'application s'arrête.

### Des _fils_ plus ou moins riches
Pour assurer la synchronisation d'une collection de documents, le _fil_ correspondant est riche: il peut y avoir beaucoup de documents modifiés. Les **_fils de synchronisation_** ne donnent lieu à des _popups_ que sur des critères très restrictifs gérés par l'application afin de ne pas submerger l'utilisateur.

Les **_fils de news_** sont a contrario beaucoup plus sobres: ils correspondent à quelques documents / collections bien ciblés et pas à tous ceux qui seraient nécessaires à une synchronisation complète de ces documents.

> Ce paradigme ne peut pas être mis en œuvre dans toute sa généralité: comment sont définis les _fils de news_ et les _documents synchronisés_ peut aboutir à une impossibilité technique de mise en œuvre, ou à un coût de développement prohibitif, ou à un coût calcul insupportable. La solution générique décrite ci-après correspond à une restriction de ces concepts permettant de faire fonctionner des applications avec un minimum d'effort, tant de développement que de calcul.

# Un utilisateur, ses appareils et ses applications

Un utilisateur qui veut utiliser une application (Web-PWA ou non) est placé devant deux cas de figure:
- **soit l'appareil qu'il s'apprête à utiliser est _le sien_**: un peu plus généralement c'est un appareil,
  - qu'il utilise régulièrement, que se soit le sien ou celui d'un proche,
  - c'est un appareil de confiance: il peut laisser quelques informations personnelles localement dessus (mais cryptées).
- **soit l'appareil qu'il s'apprête à utiliser ne lui est pas familier**, il est partagé par des utilisateurs inconnus comme au cyber-café ou est celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelques informations que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour raccourcir ses saisies.

### Appareil _personnel_: ses profils
C'est un appareil qui _peut_ être utilisé par d'autres que soi-même. Typiquement dans un cadre familial ou un couple, l'appareil n'est pas strictement _personnel_.

Le _login_ à un tel appareil est _protégé_ par un mot de passe ou tout autre dispositif que seuls des proches connaissent, à moins que l'appareil leur soit prêté déverrouillé. 

_Plusieurs personnes peuvent plus ou moins occasionnellement s'en servir_: d'où le principe d'y avoir possiblement plusieurs _profils_.
- un browser comme Firefox a une notion d'utilisateur: on peut basculer d'un utilisateur à un autre (sans pour autant avoir changé de connexion au niveau de l'OS). Chacun a ses sites favoris, son historique de navigation et ses mots de passe enregistrés.
- Thunderbird, le gestionnaire de mails locaux, supporte de gérer plusieurs _profils_, chacun avec ses comptes mails.

### Des _profils_ plus ou moins bien défendus
Ce n'est pas parce qu'on partage un appareil avec un proche qu'on a envie de partager avec lui ses informations confidentielles.

Or dans les cas cités ci-dessus, la confidentialité est plutôt _lâche_:
- Thunderbird ne demande rien: on choisit son profil, sans mot de passe ou quoi que ce soit. Les boîtes mail sont de toutes les façons en clair dans le file-system, confidentialité _intra-familiale_ zéro.
- Chrome s'ouvre sur le compte _courant_: si vous ne vous déconnectez pas **explicitement** avant de fermer le browser, il s'ouvre la fois suivante sur votre compte, ses mots de passe, ses historiques et ses favoris. Si vous vous déconnectez Chrome est strict sur la connexion et demandera même à votre téléphone si c'est vraiment vous qui essayez de vous connecter: il suffit d'y répondre OUI et c'est bon (même si vous vous êtes fait voler votre téléphone en état déverrouillé, Google est content).

### Profil personnel local à un appareil
Chaque profil dispose d'un stockage local de données destiné à:
- faciliter et accélérer le démarrage des applications qui vont y trouver des données d'autorisation, le cas échéant multiples et potentiellement fastidieuses à fournir.
- pouvoir utiliser, avec des restrictions, une application en _mode avion_, sans aucun accès à Internet, à partir des documents synchronisés lors d'une utilisation antérieure.

Ce stockage est **crypté**: même l'accès par le _file-system_ ne permet pas à un _pirate_ d'accéder à son contenu.

La clé de cryptage est dérivée d'une **phrase complexe** que seul le détenteur du profil connaît: après deux échecs sur la donnée de celle-ci, le _profil_ s'efface (pas les données sur les serveurs des applications).

### Lancement d'une application sur un appareil _personnel_
La liste des quelques profils enregistrés est présentée:
- l'utilisateur choisit le sien et donne sa phrase de protection.
- si c'est la bonne phrase, l'application s'ouvre en ayant une série d'autorisations préremplies, ses groupes de dossiers synchronisés, ses fils de news ... Il peut:
  - en désactiver certains, temporairement ou non,
  - en activer de nouveaux qui compléteront son profil.

En fermant l'application, l'utilisateur peut choisir:
- de laisser ses _fils de news_ activés: des notifications apparaîtront, même quand l'application sera fermée et même si ce n'est plus le même utilisateur qui dispose de l'appareil. Les _notifications_ ne font qu'annoncer des changements sans en donner les détails et jamais d'informations confidentielles. 
- de fermer ses _fils de news_: aucune notification ne parviendra plus sur l'appareil relativement à cette application. L'utilisateur peut le prêter à quelqu'un d'autre sans risque ... mais lui-même ne recevra plus de notifications (il faut choisir).

### Lancement d'une application EN MODE AVION sur un appareil _personnel_
La liste des quelques profils enregistrés est présentée:
- l'utilisateur choisit le sien et donne sa phrase de protection.
- si c'est la bonne phrase, l'application va s'ouvrir en disposant des _dossiers synchronisés_ qui l'ont été lors de la dernière utilisation connectée.
- en mode avion, il n'y a pas de réseau, pas de _fils de news_: les dossiers peuvent être consultés mais pas mis à jour. L'utilisateur peut saisir des textes ou formulaires purement locaux et stocker des fichiers (comme des photos prises en mode avion): toutes ces informations sont cryptées dans le stockage local et pourront être utilisées pour mettre à jour des documents quand le réseau sera à nouveau disponible en sécurité.

### Lancement d'une application sur un appareil _anonyme_
L'application se lance sans aucune autorisation activée: elle ne va guère afficher que des informations d'une grande banalité et proposer de saisir les autorisations nécessaires pour,
- accéder à des fils de documents synchronisés, les consulter et mettre à jour selon les droits ouverts par l'autorisation fournie.
- ouvrir des _fils de news_ également en citant les autorisations nécessaires.

> Pour l'application, les possibilités sur un appareil _anonyme_ sont les mêmes que sur un appareil _personnel_: l'utilisateur a seulement fourni plus d'effort pour faire valoir ses droits d'accès (alors que son _profil_ les a préremplis pour un appareil _personnel_).

La fermeture de l'application ne laisse pas le choix sur un appareil _anonyme_: les abonnements aux _fils de news_ sont tous supprimés, aucune notification ne parviendra plus sur cet appareil résultant de l'usage précédent de l'application.

### Mode _veille_
Les applications ouvertes ont un mode _veille_ optionnel: s'il est activé, l'application se met en veille en cas de non utilisation pendant quelques minutes (fixés selon le degré de paranoïa de l'utilisateur). Pour sortir de la veille un code PIN est nécessaire et au second échec l'application se ferme.

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

# Base de données partagée par tous les prestataires
Cette base unique n'est concerne toutes les applications et tous les prestataires.

Tous les services des prestataires peuvent y accéder pour les opérations qui leur sont ouvertes.

Toutes les applications terminales peuvent solliciter un quelconque des prestataires afin d'accéder concernant un de leurs utilisateurs.

Les données de cette base sont:
- Le **répertoire des services**.
  - il liste pour toutes les applications le prestataire (son URL) gérant chaque organisation.
- Le **répertoire des utilisateurs**. Tout utilisateur qui s'y est inscrit (c'est une facilité pas une obligation) dispose d'une entrée personnelle confidentielle et sécurisée. Il y trouve:
  - un enregistrement de _ses préférences_ personnelles.
  - une liste de _credentials_, de _droits d'accès_ déposés par une application terminale au nom de l'utilisateur.

> Un _credential_ est une petite structure qui renferme des données d'authentification comme un couple login / mot de passe (ou tout autre dispositif). Parfois en retour d'une demande d'authentification, un serveur peut retourner un _jeton crypté_ à joindre aux requêtes suivantes: Ce type de jeton peut être enregistré dans un _credential_.

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
- L'application terminale peut générer une clé `K-ti-S` à partir du couple `Pub-S / Priv-ti`. Comme `K-ti-S` et `K-S-ti` sont égales, l'application terminale peut s'en serir pour décrypter les réponses / données cryptées pour elle par l'application serveur (et nul autre ne peut le faire).

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

# Documents, fichiers et _fils_ traçant leurs évolutions
## Document
Un document est composé de:
- un agrégat de données structurées selon le concept JSON:
  - données primitives: _string, number, boolean_.
  - deux structures:
    - liste ordonnée,
    - map _clé (string) / valeur.
- un ensemble de _fichiers_.

> Un document peut en conséquence être volumineux.

Il y a plusieurs _types_ de document, chacun correspondant à une structure dont la racine est une map de _propriétés_, chacune pouvant aussi avoir elle-même une valeur primitive ou une liste ou une map.

Parmi ces propriétés une liste ordonnée de propriétés _string_ immuables constitue l'identifiant du document (clé primaire en SQL, path en NOSQL). 

Exemple du document `CART` du _use-case circuit court_:
- un _carton_ est un ensemble de produits emballés ensemble par un producteur `pr` d'un groupement `gp` gérant un camion à destination de points de livraison `gc` pour une livraison donnée `livr`.
- 4 propriétés sont identifiantes: `gp pr livr gc`.

On peut définir des **regroupements** d'identifiants:
- le regroupement #1 `gp.livr`: en fixant cette valeur un point de livraison peut obtenir la liste des cartons à décharger du camion expédié par le groupement pour cette livraison, tous producteurs confondus.
- le regroupement #2 `gp.pr`: en fixant cette valeur un producteur peut obtenir la liste de tous les cartons qu'il doit composer pour toutes les livraisons en cours et tous les points de livraison.

**Parmi les propriétés certaines (de type _string_ ou _number_) sont _indexables_**.
- soit pour être utilisées comme identifiants secondaires du document,
- soit pour filtrer la collection du document par des quantités de seuil.

La propriété `version` du document est un numéro d'ordre de mise à jour: la numérotation est _chronologique_ mais pas _continue_.

La propriété `del` contient le jour de suppression quand la document est supprimé logiquement mais pas purgé physiquement (il est _zombi_).

### Stockage d'un document d'un type donné
Le document stocké dans une table (SQL) ou une collection (NOSQL) spécifique du type de document.

**L'ensemble des propriétés** est sérialisé dans un champ dénommé `_data_`: ce contenu est désérialisable dans les applications terminales et les serveurs.

En base de données, les propriétés **visibles de la base de données** sont:
- `id` : clé primaire ou path.
- `version`.
- `del`.
- `_data_`.
- `_files_` : structure descriptive des fichiers attachés aux document.
- `z1 z2 ...` : les _regroupements_ de propriétés identifiantes (s'il y en a).
- `p1 p2 ...` : les _propriétés_ indexables.

Le contenu structuré complexe du document est en conséquence _opaque_ pour la base de données (et crypté pour la plupart des types de documents).

### Fichiers attachés à un document
Un fichier est stocké en deux parties:
- son **descriptif** figurant dans le document (dans _files_).
- son **contenu effectif** stocké dans un **Storage** (de type AWS/S3, Google Storage, etc.).

Un fichier est identifié par `fid` un id aléatoire universel:
- un fichier **ne change pas** de contenu, un autre est créé avec un nouveau contenu.

Le descriptif d'un fichier a les propriétés suivantes: 
- `nom` : c'est un texte dont la seule contrainte est d'être un nom acceptable dans un système de fichiers (ne pas contenir de `/` ...).
- `time` : la date-heure de la transaction qui l'a validé.
- `type` : type mime du fichier comme 'image/jpg'.
- `size` : sa taille en bytes (son _original non crypté_).
- `sha` : le digest SH256 de l'original non crypté.

La propriété `_files_` du document est une map avec une entrée fid par fichier et pour valeur le descriptif du fichier.

> Selon la logique de l'application, la propriété `nom` **est ou non unique dans son document**. Si elle est unique, le stockage d'un fichier d'un nom donné supprime d'office le fichier portant antérieurement ce nom. Si la propriété nom n'est contrainte à être unique, plusieurs fichiers porteront le même nom dans un document (avec des propriétés `time` différentes) vus comme autant de _révisions_ pour un nom donné.

Le contenu du fichier est stocké sous un _path_ dans l'espace de stockage `folderId/fid`:
- `fid` est suffisant pour garantir l'unicité du contenu.
- `folderId` définit un _folder_ de rangement et a une structure `a/b/c ...` dont le seul intérêt est de pouvoir purger en une seule commande tous les fichiers sous une partie de ce path, par exemple les fichiers dont le path commence par `a/b`.
- les termes qui définissent le folderId sont ceux apparaissant dans l'id du document:
  - `gp pr livr gc` dans l'exemple ci-avant, l'id du document,
  - `gp livr` le groupement d'id #2 défini pour le rattachement au fil CMDGP.

### Protocole de stockage / suppression d'un ou plusieurs fichiers
Une ou plusieurs opérations de **preload** charge le contenu du fichier dans le storage sous le path `folderId/fid`, `fid` étant généré à cet instant.

Avant le stockage physique la ou les opérations de _preload_ note dans la table `todelete` le couple (`folderId/id`, `date du jour`).

Une opération de validation enregistre ensuite dans le ou les documents concernés les nouveaux fichiers fid et leurs descriptifs. Cette opération s'accompagne éventuellement d'une liste de fid à détruire dans ces mêmes documents.
- pour chaque document sa propriété _files_ est mise à jour.
- s'il y a des fichiers à détruire,
  - leurs entrées sont enlevés des propriétés _files_ de leurs documents.
  - leur couple (`folderId/id`, `date du jour`) est inséré dans la table todelete.
- la transaction est _validée par un commit_ de la base de données, les fichiers nouveaux _existent_ les fichiers supprimés n'existent plus.

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
Le _type_ d'un fil définit son objectif: tracer les évolutions d'un certain nombre de documents et pour chaque selon quel filtrage sur son id. 

Dans le _use-case circuit court_ par exemple `CMDGP` sert à rattacher tous les documents utiles à une livraison `livr` gérée par un groupement `gp`.
- l'identifiant d'un fil de type `CMDGP` est `gp.livr`.
- les types de documents rattachés à ce fil sont listés `CHD, BCG, CART`.
  - `CHD` (le chat ouvert pour une livraison donnée) a pour identifiant `gp livr`: il n'y aura au plus qu'un seul document `CHD` rattaché au fil.
  - `BCG` (un bon de commande d'un point de livraison) a pour identifiant `gp livr gc`: il y aura donc une collection de documents `BCG` dans le fil, tous ceux ayant pour regroupement indexé de propriétés `gp.livr` (soit au plus un par point-de-livraison `gc`).
  - `CART` à pour identifiant `gp pr livr gc`: il y aura donc une collection de documents `CART` dans le fil, tous ceux ayant pour regroupement indexé de propriétés `gp.livr`. Un carton est créé par un producteur qui y met tous les produits d'une livraison destiné à un même point-de-livraison.

> Connaissant le type et l'identifiant d'un fil, par exemple `CMDGP/gp.livr` on peut _tirer_ toute une collection, le cas échéant nombreuse, de documents `CHD`,  `BCG`, `CART` rattachés au même fil, en l'occurrence celui concernant la livraison d'un camion organisé par un groupement de producteur pour une livraison donnée à plusieurs point-de-livraison (une _tournée_).

Un _type de fil_ définit de facto un critère de sélection s'appliquant à un ensemble de documents ayant pour identifiant ou regroupement d'identifiant une valeur donnée.

**Un fil donné, une instance de son type pour un identifiant donné, est _stocké_ en base de données** dans une table / collection portant le nom du type, par exemple `CMDGP`. Ces tables / collections ont toutes le même schéma.

**Propriétés indexées:**
- `id` : concaténation des ids de sa clé primaire / path: `gp.livr`. Les valeurs individuelles des éléments de la clé ne sont pas citées (mais extractibles depuis l'ID).
- `version` : numéro séquentiel d'ordre de mise à jour du document le plus récent attaché au fil (ou détruit).

**Propriété opaque _data_:**
- `versions` est une map avec une entrée pour chaque type de document donnant le dernier numéro de version, soit _du_ document si c'est un singleton, soit du document de la collection _le plus récemment mis à jour_.
- `cleandate` : c'est la date du dernier nettoyages des suppressions (voir plus avant).

### Utilisation des _fils_
Chaque fil est une **trace** de l'évolution la plus récente des documents qui lui sont attachés: 
- le fait que la version d'un fil s'incrémente à chaque mise à jour d'un de ses documents fait du fil un événement _notifiable_.
- dans cette _notification_, une application terminale peut retrouver pour chaque type de document (UN document si c'est un singleton dans le fil, sinon une collection des documents) si ce document ou cette sous-collection a évolué depuis la version qu'elle détenait.

Une application terminale qui a gardé en mémoire le dernier fil qui lui lui a été transmis, peut à réception du nouvel état du fil, demander à un serveur la liste des documents de version postérieure à celle qu'elle détenait et en effectuer la mise à jour dans sa mémoire. Cette mise à jour est :
- optimale: elle n'est demandée QUE si un des documents d'un type qui intéresse l'application a changé.
- incrémentale: seuls les documents ayant changé depuis la version connue de l'application terminale sont transmis.

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

Le document est devenu _zombi_.

> _Remarque_: rien ne l'empêche de renaître plus tard.

La possibilité d'obtention d'une mise à jour incrémentale depuis un état détenu à la date `d` est bornée par la possibilité de disposer des suppressions.
- si elles sont gardées, même _zombi_ avec une taille minimale, sans limite de temps, la base peut être encombrée de zombis.
- en fixant un délai d'un an par exemple, les mises à jour depuis un état de plus d'an sont traitées comme une demande _intégrale_ avec la fourniture de tous les documents existants. C'est à l'application terminale de déterminer les suppressions de documents depuis l'état à la date `d` qu'elle connaît et le nouvel état complet.

Le serveur va à l'occasion d'une suppression d'un document regarder les `cleandate` du ou des fils auxquels est rattaché le document: si ces dates ont plus de 18 mois, il va purger effectivement les documents zombis depuis plus d'an de ces fils (en testant leur propriété `del`). Il mettra à jour la ou les `clean dates` du ou de ces fils.

De cette façon les documents supprimés seront purgés au fil de l'eau mais avec des opérations distantes de six mois au moins.

# Annexe: le Use Case _circuit court_

## Vision générale: les _documents_

Un **groupe de consommateurs** est identifié par son code `gc`. 
- **Document FGC** : fiche de renseignement donnant des informations de contact, son ou ses mots de passe, liste des groupements de producteurs auxquels il peut commander. 

Un **consommateur** est identifié par son code dans son groupe: `gc co`. 
- **Document FCO** : fiche de renseignement donnant des informations de contact, son mot de passe, un statut de blocage.

Un **groupement de producteurs** est identifié par son code `gp`. 
- **Document FGP** : fiche de renseignement donnant des informations de contact, son ou ses mots de passe, liste des groupes de consommateurs qui peuvent lui émettre des commandes.

Un `producteur` est identifié par son code dans son groupement: `gp pr`. 
- **Document FPR** : fiche de renseignement donnant des informations de contact et son mot de passe.

Une **référence de produit** est identifiée par son code dans son producteur: `gp pr rp`.

Le **calendrier des livraisons d'un groupement** de producteurs est identifié par le code du groupement `gp`.
- **Document CALG** : il donne la liste des livraisons déclarées avec pour chacune:
  - son de code `livr`, date de livraison (théorique et immuable).
  - les dates-heures d'ouverture, de clôture des commandes, d'expédition et d'archivage. Des règles fixent comment et quand ces dates peuvent être changées, en particulier les unes par rapports aux autres.
  - la liste des groupes de consommateurs livrés (point de livraison).

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
  - généré / mis à jour à chaque mise à jour d'un BCC du groupe pour une ligne concernant un produit de ce producteur. 
  - il donne par produit du producteur la somme des quantités commandées dans le groupe. Ce document est une redondance générée / mise à jour à chaque mise à jour 

_Remarque 1_: quand une commande est réceptionnée, pour chaque produit (en général commandé mais pas forcément), figure une quantité livrée ou un poids livré.
- pour certains produits, des poulets par exemple, la quantité livrée est le nombre N de poulets et le poids livré est une liste de N poids individuels des poulets. Quand il y a un poids total c'est, soit temporaire en estimation avant obtention des poids individuels, soit la somme des poids individuels.

_Remarque 2_: le rapprochement entre les informations de commande et celles de livraison (réception) permet de détecter des problèmes à résoudre:
- des quantités excédentaires ou insuffisantes: il faudra pour chaque consommateur ajuster la quantité.
- des paquets individualisés (des poulets) au déchargement non attribués et des consommateurs n'ayant pas reçu leurs paquets.
- des produits livrés mais pas commandés:il faudra les répartir sur des consommateurs, le cas échéant le _groupement_ lui-même.

Le **catalogue général des produits** d'un groupement est identifié par `gp`.
- **Document CATG**. Il donne pour chaque produit:
  - un _descriptif permanent_ ne pouvant pas changer après déclaration, sauf le libellé. Un produit _à l'unité_ ne peut pas devenir _au poids_, il faut définir un autre produit.
    - son code et sa référence (_code-barre_).
    - un libellé descriptif,
    - si c'est un produit _sec_, _frais_ ou _surgelé_.
    - des indicateurs de label / qualité, taux de TVA applicable, 
    - si le produit est vendu à l'unité, au poids et / ou par _caisse_ / _demi-caisse_.
  - une _liste chronologique_ de dates à laquelle les conditions de ventes du produit ont changé:
    - sa disponibilité,
    - son prix unitaire,
    - ses poids _net_ et _brut_: le poids _net_ est celui sans l'emballage (ce que mange le consommateur), son poids _brut_ inclut l'emballage (ce que ça pèse dans le camion).

Le **catalogue d'une livraison** d'un groupement est identifié par `gp livr`.
- **Document CATL**. Il donne pour une livraison, la liste des produits avec pour chacun sa conditions de vente.
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

Le **répertoire des producteurs** d'un groupe est identifié par le code du groupe `gc`.
- **Document RPR** : pour le groupement lui-même et pour chaque producteur il donne une fiche de contact.

Le **chat d'une livraison** est identifié par le groupement livrant et la date de livraison `gp livr`.
- **Document CHL**: c'est une suite chronologique de news alimentée par le groupement. Tous les groupes de consommateurs sont concernés. Un groupe de consommateurs peuvent y déposer aussi des news (avec modération).

Le **chat d'une distribution** est identifié par le groupement livrant, la date de livraison et le groupe distributeur `gp livr gc`.
- **Document CHD**: c'est une suite chronologique de news alimentée par le groupe et les consommateurs.

Le **chat d'un groupe de consommateurs** est écrit par le groupe et les consommateurs du groupe indépendamment de toute livraison et est identifié par `gc`.
- **Document CHCO**: c'est une suite chronologique de news alimentée par le groupe et les consommateurs. Les groupements de producteurs peuvent émettre, avec modération, des news.

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
- chat de son groupe
- chats des livraisons ouvertes de son groupe
- ses commandes sur les livraisons ouvertes. Ce dernier fil peut être utile pour un _consommateur_ pour lequel il y a plusieurs utilisateurs susceptibles de commander: famille, proches, voisins... Il permet de voir apparaître des notifications quand un de ces utilisateurs a modifié une commande.

> Il ne suit pas par fils de news les évolutions tarifaires, les évolutions des dates, etc. C'est l'animateur du groupement qui en fera les informations de synthèses sur le chat du groupe.

### Point de vue d'un groupe (un de ses animateurs) (A SUIVRE).

> Il ne suit pas par fils de news les évolutions tarifaires, les évolutions des dates, etc. C'est l'animateur du groupement qui en fera les informations de synthèses sur le chat du groupe.

### _Fils de synchronisation_ des documents
Chaque document est accessible par son identifiant.

Certains documents peuvent être _synchronisés_: pour cela il faut définir dans quels _fils de synchronisation_ chaque document est rattaché:
- chaque application terminale déclare à quels _fils_ elle est abonnée de manière à recevoir une notification circonstanciée quand un document rattaché à ce fil a changé.
- chaque document peut être rattaché à au plus DEUX fils de synchronisation.

#### Liste des documents
- FGC: fiche d'un groupe. gc
- FCO: fiche d'un consommateur: gc co
- FGP: fiche d'un groupement: gp
- FPR: fiche d'un producteur: gp pr
- CALG: calendrier d'un groupement: gp
- LIVRG: livraison d'un groupement: gp livr
- BCC: bon de commande d'un consommateur: gc co gp livr
- BCG: bon de commande d'un groupement: gc gp livr
- CART: carton d'un producteur pour la livraison à un groupe: gp pr livr gc
- CATG: catalogue des produits d'un groupement: gp
- CATL: catalogue d'une livraison: gp livr
- RG: répertoire général des groupes et groupements
- RC: répertoire des consommateurs: gc
- RP: répertoire des producteurs: gp
- CHL: chat d'une livraison: gp livr
- CHD: chat d'une distribution: gp livr gc
- CHCO: chat d'un groupe de consommateurs: gc
- CHPR: chat d'un groupement de producteur: gp

Fil #CMDG : gc gp livr - commande d'un groupe à un groupement
- BCG : 
- CART : pr
- BCC : co

Fil #CMDC : gc gp livr - commandes des consommateurs
- BCC : co

Fil #CMDOV : gc gp - commandes ouvertes d'un groupe à un groupement
- BCG : livr

Fil #CALGP : gp - calendrier des livraisons d'un groupement
- CALG :
- LIVRG : livr

Fil #CMDGP : gp livr - commandes des groupes à un groupement
- CHD :
- BCG : gc
- CART : pr gc

Fil #CMDPR : gp pr : commandes ouvertes d'un producteur
- CART : livr gc

Fil #RGC
- RG :
- RC : gc

Fil #RGP
- RG :
- RP : gp

Fil #CHL : gp
- CHL : livr
- CHD : gc livr

Fil #CHCO : gc
- CHCQ :

Fil #CHPR : gp
- CHPR :

#### Abonnement
Pour s'abonner à un il faut fixer:
- son code et son path exact: #CMDGP/gc.gp.livr
- un filtre s'appliquant seulement quand le path a matché: ne générer une notification que si le ou les documents modifiés ont une id compatibe avec la condition de filtre pour son type,
  - aucun CHD ou tous CHD/* ou tous CHD  dont le gc de l'id est égal à la valeur indiquée: CHD/gc=gc

#### Abonnements pour un groupe: `gc`
Abonnement à `#RGP [RG, RP]` : donne une liste des groupements `gp` à qui il peut commander.

Abonnements aux fils `#CALGP/gp [CALG, LIVRG]` pour tous les `gp` récupérés.

Quand il fixe une livraison courante `gp.livr` il s'abonne au fil `#CMDG/gc.gp.livr [CMD, BCG, CART]`.

Abonnements à tous les fils `#CHL/gp [CHL, CHD/gc=gc]`

Abonnements à `#CHCO/gc`.

#### Abonnements pour un consommateur: `gc co`
Abonnement à `#RGP [RG, RP]` : donne une liste des groupements `gp` à qui il peut commander.

Abonnements aux fils `#CALGP/gp [CALG, LIVRG]` pour tous les `gp` récupérés.

Quand il fixe une livraison courante `gp.livr`, abonnement au fil `#CMDC/gc.gp.livr, [BCC/co=co]`.

Abonnements à tous les fils `#CHL/gp [CHL, CHD/gc=gc]`

Abonnements à `#CHCO/gc`.

#### Abonnements pour un groupement: `gp`
Abonnement à `#RGC [RG, RC]` : donne une liste des groupes `gc` qui peuvent commander.

Abonnement au fil `#CALGP/gp [CALG, LIVRG]`

Quand il a fixé une livraison `livr`, abonnement à `#CMDGP/gp.livr [CHD, BCG, CART]`.

Abonnement au fil `#CHL/gp [CHL, CHD]`

Abonnements à `#CHPR/gp`.

#### Abonnements pour un producteur: `gp pr`
(TODO)

### Index sur les documents
Exemple de `CART`
- ID: `gp pr livr gc`
- Fait partie des fils:
  - `#CMDGP gp.livr` - Index requis I1: `gp.livr`
  - `#CMDPR gp.pr` - Index requis I2: `gp.pr`

CART doit être stocké accessible par son ID:
- en SQL avec une PK `gp pr livr gc`
- NOSQL avec un path `gp.pr.livr.gc`

Récupérer tous les `CART` d'un fil `#CMDGP` d'une version `v > vx` est une condition d'égalité sur `gpx.livrx` (index I1) et `> à sur vx`.


-----------------------------------------------------------------------

# Contributions antérieures

# Mémoire _cache_ locale de données sur un appareil / mode "avion"
Au lancement d'une application sur un appareil, l'utilisateur se trouve devant plusieurs possibilités:
- il y a du réseau et l'utilisateur le considère comme fiable (non écouté malicieusement):
  - si une session est _ouverte_, il peut saisir son code PIN et la reprendre.
- sinon s'identifier 

Au lancement d'une application sur un appareil, l'utilisateur va déclarer si cet appareil est _personnel_, soit qu'il lui appartient, soit qu'il le partage avec une ou quelques personnes de confiance.


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

