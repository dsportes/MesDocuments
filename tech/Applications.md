---
layout: page
title: Architecture pour des applications réactives
---

Le fonctionnement d'une **application réactive** fait intervenir plusieurs éléments logiciels.

### Les "applications"
Les utilisateurs peuvent exécuter une **application** sur un de leurs _appareils / terminaux_ munis d'un moyen de communication avec un humain (écran, clavier, souris ...). Selon la variante technique choisie, un utilisateur lance une application portant un nom (comme `monAppli`):
- soit dans un browser _en ouvrant la page Web_ de l'application: l'URL désigne où le logiciel de l'application est stocké.
- soit _en l'installant_ sur un de ses appareils depuis un _magasin d'applications_ puis en la lançant.

> Sur un terminal donné, l'application `monAppli` est ou n'est pas en exécution, il n'est pas possible d'en lancer plusieurs simultanément.

### Les "services" de traitement des données
Un **service** porte un nom et est localisé par une URL:
- un service supporte plusieurs **opérations**. 
- les opérations sont demandées par une application. Une opération reçoit en entrée des paramètres, effectue le traitement demandé et retourne un résultat qui en général va influer sur l'affichage de l'application l'ayant sollicitée.

En première approche lorsqu'une opération d'un **service** est invoquée, un programme démarre _quelque part sur Internet_, exécute le traitement demandé puis s'arrête.

Si logiquement ceci est perçu comme tel vu de l'extérieur, le scénario _technique_ est un peu différent:
- au lieu de se terminer après la fin de l'opération, le programme reste _vivant_ en attendant qu'une autre opération soit demandée afin que l'énergie de calcul dépensée pour le chargement du programme soit _amortie_ sur un plus grand nombre d'opérations.
- le programme de plus est capable de traiter plusieurs opérations en parallèle.
- de facto le programme ne s'arrête que quand aucune demande d'opérations n'est parvenue _pendant un certain temps_.
- si le service est très sollicité, plusieurs programmes peuvent être lancés, sur des calculateurs différents le cas échéant, afin d'écouler le trafic des demandes d'opérations.

> Le temps au bout duquel un programme de traitement du service s'arrête en l'absence de trafic est un des paramètres de configuration de l'installation du service, de même que le nombre maximal de programmes s'exécutant en parallèle. 

> Certaines configurations peuvent fixer un nombre fixe de ces exécutions et spécifier que les programmes ne s'arrêtent pas même en l'absence de trafic: les choix résultent d'une valorisation économique dépendant des tarifs des fournisseurs de traitements à distance.

> UNE **application** donnée, par exemple `monAppli`, peut faire appel à plusieurs **services**, par exemple `compta` et `rando`. Une application qui ne fait appel à aucun service a un comportement de _calculette_ et n'utilise aucune donnée externe.

### La "base de donnée d'un service", sa partition par "organisation"
En première approche un **service** (`rando` par exemple) a **SA** base de données (`rando-1` par exemple): deux services différent ne partagent pas une même base.

La base de données est **partitionnée** par **organisation**:
- Les données relatives à une organisation `IDF` sont totalement disjointes de celles de l'organisation `PACA` mais la structure des données est unique.
- **une opération d'un service est strictement spécifique à UNE organisation** et n'accède dans la base de données qu'aux données de celle-ci.

> Ce dispositif _multi-tenant_ rend possible d'ajouter une nouvelle organisation sans interruption des services (même ceux en cours d'exécution).

Toutefois, ce mécanisme peut conduire à avoir une base de données trop volumineuse quand un grand nombre d'organisations sont supportées. Pour éviter ce problème, un **service** peut accéder à plusieurs bases de données, chaque organisation étant _hébergée_ dans une de ces bases (et une seule).

> Vu de l'extérieur c'est _comme si_ il n'y avait qu'une base unique et même c'est _comme si_ celle-ci était dédiée à l'organisation spécifiée en paramètre de chaque opération du service.

> **L'intégration des données provenant de plusieurs services** se fait au niveau des applications. Par clarté de conception mais surtout parce qu'invoquer une opération demandes _credentials_ cryptés qui ne sont détenus que par les applications mais jamais par les services.

> L'intégration des données provenant de plusieurs organisations pour un service donné se fait au niveau des applications: une _opération_ ne traite qu'une organisation.

### Le "storage de fichiers d'un service", sa partition par "organisation"
Un _storage_ a une structure qui s'apparente à un _file-system_.
- le stockage d'un _fichier_ (contenu binaire crypté) se fait en une opération atomique,
- le stockage de plusieurs fichiers ne fait pas l'objet d'un commit de type ACID (chacun peut ou non avoir été mis à jour).

Le _storage_ est également partitionné par _organisation_.

Comme pour une base de données, au cas où le volume l'exigerait, plusieurs _storage_ peuvent exister, chacun avec sa propre technologie le cas échéant.

> Un _storage_ n'est pas non plus partagés par plusieurs _services_.

> Un storage permet de mémoriser des volumes considérables de données,
- peu ou pas mises à jour après stockage,
- dont le contenu est en général _opaque_ pour les services (mais ce n'est pas obligatoire),
- adapté à l'archivage de données de _legacy_.

### Les "Utilisateurs" et leurs _droits d'accès / credential_ 
Les **utilisateurs** sont identifiés par un identifiant aléatoire et anonyme, sans référence avec des identifiants personnels dans la _vraie_ vie.

Depuis un appareil quelconque un utilisateur peut lancer une application dès lors qu'il en connaît l'URL. Celle-ci peut invoquer des **services** et leurs opérations MAIS toute opération exige en général que l'utilisateur exhibe un ou des _droits d'accès_ appropriés pour l'opération demandée et ses paramètres.

Par exemple une opération d'accès aux données d'un `adhérent` identifié `abcd` va exiger que l'application communique à l'opération un _jeton_ qui prouve que l'utilisateur dispose du droit d'accéder aux données de cet adhérent. Le _droit_ requis peut être différent selon que l'opération effectue une lecture ou une mise à jour de l'adhérent.

Un _droit d'accès_ comporte deux parties:
- **une partie conservée par l'utilisateur** dont le texte comporte les éléments cryptographiques lui permettant de _signer_ chaque jeton attaché une demande d'une opération.
- **une partie conservée dans la base de données** qui permet à l'opération de _vérifier_ que le _jeton_ reçu en paramètre de l'opération est effectivement valide et contient bien les données qu'il prétend détenir.

Ce mécanisme détaillé par ailleurs permet,
- de ne pas stocker dans la base de données les éléments de _signature_,
- de pouvoir refuser des _jetons usurpés_, c'est à dire ayant déjà été présentés une fois et représentés plus tard.

### _Coffre fort_ / _safe_ d'un utilisateur
Chaque _droit d'accès_ est un texte long, comportant des textes d'apparence aléatoire, bref impossibles à mémoriser (et à inventer par _force brute_). 

L'utilisateur pourrait certes disposer d'un fichier personnel où il les rangerait mais la sécurité et l'accès depuis plusieurs terminaux à ce fichier exposerait ces données de sécurité _critiques_ aux pertes et aux vols.

Chaque utilisateur dispose à cet effet d'un _coffre-fort_ personnel où ses droits d'accès seront rangés, cryptés et sécurisés. 

Le _coffre-fort_ d'un utilisateur a pour identifiant celui de l'utilisateur (ou l'inverse un utilisateur est identifié par le numéro de son coffre). Il comporte plusieurs _rubriques_:
- son **entête** qui détient les éléments cryptographiques techniques nécessaires à son fonctionnement.
- la **liste de ses droits d'accès**.
- une **liste de terminaux de confiance**, c'est à dire des terminaux d'où il pourra s'identifier par un code PIN plus simple que son identification _forte_ et sur lesquels chaque application pourra laisser des _documents en mémoire cache_ locale cryptée permettant un usage en _mode avion_.
- une **liste de préférences** de comportement et d'affichage de son choix afin de retrouver en lançant une nouvelle session, l'organisation de l'écran qu'il souhaite, les options de son choix, sa langue de travail, etc.

#### Dépôts des coffres-forts : _standard_  ou _personnels_
Un **dépôt _standard_** est géré: tout utilisateur peut y disposer de son _coffre-fort_.

Mais certains utilisateurs sont prudents / paranoïaques et peuvent ne pas vouloir que leurs droits d'accès soient conservés par une entité qu'ils ne maîtrisent pas ou qui pourrait être défaillante ... 

Chaque utilisateur (ou groupes d'utilisateurs) peut installer son propre dépôt de _coffres-forts_ dans une base de données MySQL d'un site Web de son choix (et sous son entière responsabilité d'administration) muni d'un script PHP standard mais dont il peut lire le texte et s'assurer de sa non nocivité. 

Des moyens sont données pour basculer du dépôt _standard_ vers un _dépôt spécifique_ (et réciproquement), ainsi que pour effectuer des _backup_: l'image d'un _coffre-fort_ peut être exportée crypté par une clé détenue par le seul utilisateur.

> Le _contenu_ d'un coffre-fort est lisible _en clair_ **pour son propriétaire et seulement lui**, sauf que étant plein de données cryptographiques le terme _en clair_ est une vue de l'esprit.

### Exécution d'une application en _mode AVION_
Quand un utilisateur a déclaré un ou des terminaux **de confiance** quand il y lance une session d'une application celle-ci peut utiliser une **mémoire cache de documents et fichiers**, cryptée et sécurisée sur le terminal.

Depuis ce même terminal, l'utilisateur peut rouvrir une session qui s'est antérieurement exécutée sur ce terminal:
- s'il a accès au réseau Internet, le lancement sera rapide du fait que beaucoup de documents n'auront pas à être redemandés aux services, étant déjà _en cache_.
- s'il n'a pas accès au réseau Internet il peut rouvrir son application en **mode AVION** et accéder (en lecture seulement) aux documents disponibles en cache du fait d'une exécution antérieure.

### Synthèse
Les **applications** s'exécutent sur le terminal de l'utilisateur où elles ont été chargées depuis leur URL (ou un magasin d'applications).

Elles font appel à des **services de traitement des données** distants, chacun ayant un jeu **d'opérations** pouvant lire / écrire SA **base de données** (éventuellement SES bases en cas de volume excessif) et SON **storage de fichiers** (éventuellement SES) partitionnés par **organisation**.

Chaque requête à une opération est dédiée à UNE organisation et n'accède qu'à la partition de la base de données dédiée à cette organisation et / ou du storage dédié à cette organisation.

Tout **utilisateur** dispose d'un **coffre-fort** détenant en particulier ses _droits d'accès_ requis à l'appel des opérations des services par une application. Tout utilisateur peut décider de confier la gestion de SON coffre-fort, soit au **dépôt standard**, soit à un **dépôt spécifique** géré par le site Web de son choix.

# Installation des applications sur un _terminal_

Un PC, une tablette, un mobile sont des _appareils / terminaux_ munis d'un moyen de communication avec un humain (écran, clavier, souris ...).

Selon la variante technique choisie, un utilisateur démarre une application:
- soit dans un browser _en ouvrant la page Web_ de l'application.
- soit _en l'installant_ sur un de ses appareils puis en la lançant.

### Application de type Web : PWA _Progressive Web Application_
Depuis un browser l'utilisateur appelle une URL d'un _magasin d'applications_ qui ouvre la page d'accueil de l'application:
- l'application peut être directement utilisable depuis cette page.
  - L'utilisateur peut déclarer un _raccourci sur son bureau_ ou dans son browser vers cette URL afin d'éviter la ressaisie de celle-ci. 
  - Certains OS (comme iOS) des appareils ne permettent pas une utilisation directe d'une telle page Web et oblige à une _installation_, au demeurant simple, de l'application depuis cette page.
- l'application _peut ou doit_ selon le browser utilisé et l'OS de l'appareil, être _installée_ par le browser. Elle apparaît ensuite comme une application locale de l'appareil avec une icône de lancement, typiquement sur le bureau.

Le changement de version d'une application PWA est automatique, la vérification d'existence et le téléchargement d'une nouvelle version intervenant au lancement: l'utilisateur est convié à appuyer sur un bouton pour redémarrer celle-ci après  installation de la nouvelle version.

> Un site comme `github.io` peut être utilisé comme _magasin d'applications_ Web-PWA: la mise en ligne d'un logiciel sur ce site est simple et gratuite, de même que la mise en ligne de sa documentation / aide en ligne.

### Application de type _mobile_
L'utilisateur l'installe depuis le ou un des magasins d'application supportés par l'OS du mobile.

Le changement de version est en général automatique mais peut être opéré manuellement.

> Il n'y a ensuite quasiment pas de différence perceptible par l'utilisateur à l'utilisation de l'application, il clique sur une icône pour l'ouvrir (la lancer).

> On peut installer une application Web-PWA sur un mobile: elle est dans la suite du document considérée comme application PWA (et non comme _mobile_).

# Services dans le _cloud_

Les applications en exécution sur leur appareil envoient des requêtes à des **services _cloud_**, chacune consistant à invoquer une opération de consultation et/ou de mise à jour des données de l'application dans la base de données ou le storage de fichiers du service.

Un _service_ est techniquement déployé selon des variantes techniques non perceptibles de l'extérieur:
- **Serveurs permanents**: plusieurs processus sont en exécution en permanence afin de traiter les requêtes qui leur parviennent sur l'URL du pool de processus et ont été routées vers l'un ou l'autre.
- **Cloud Functions**: un _serveur éphémère_ du _Cloud_ est lancé pour traiter une demande de service reçue sur son URL:
  - la demande est traitée et le serveur éphémère reste actif un certain temps pour traiter d'autres demandes. Un serveur éphémère peut traiter plusieurs dizaines de demandes en parallèle.
  - en l'absence de nouvelles demandes, un serveur éphémère reste en attente, entre 3 et 60 minutes pour fixer les idées, puis s'interrompt.
  - si le flux des demandes sature la capacité d'un serveur éphémère, un deuxième, voire un troisième etc. sont lancés.

> Ces choix de déploiement technique ne sont pas détectables par les applications terminales qui envoient des demandes aux serveurs.

## Développement du logiciel (éditeur) et ses déploiements (opérateurs)
Un _service_ correspond à un logiciel qui a été développé par un **éditeur** en vue d'assurer une **finalité applicative** bien délimitée comme par exemple:
- le service `circuitscourts` : gestion de prises de commandes entre des producteurs et des consommateurs.
- le service `discussions` : gestion de groupes de partage de documents et d'échanges interactifs.
- le service `randos` : proposition de randonnées, inscription, échanges, etc.
- le service `boutiques` : gestion du catalogue d'une boutique, de son stock, etc.

Un ou des opérateurs de _déploiement_ peuvent installer / _déployer_ ce logiciel sur le _cloud_ et le rendre accessible pour les sollicitations des applications. Deux opérateurs, par exemple **Rouge** et **Bleu**, peuvent déployer un même service logiciel:
- **Rouge** peut proposer `randos` et `discussions`,
- **Bleu** peut proposer `circuitscourts` et `randos`.

Les déploiements du logiciel `randos` par **Rouge** et **Bleu** ont chacun leur URL d'accès et peuvent différer en prix et qualité d'usage: temps de réponse, disponibilité, restrictions de volume...

## Organisations: services _multi-tenant_
Un service comme `randos`, peut à la manière de Discord, héberger les applications d'associations de randonneurs distinctes: chaque organisation / _tenant_ dispose de _son_ espace de données propre complètement étanche à celui des autres.

Un service `boutiques` propose de gérer plusieurs boutiques, pas une seule, mais de manière à ce que les données de chacune soient totalement isolées de celle des autres.

Les données d'un service d'un opérateur sont stockées dans deux _mémoires persistantes_:
- **UNE base de données** logiquement **strictement partitionnée par organisation**, sans aucun lien ou référence à des données / documents d'une organisation par une autre.
- **UN _storage_ de fichiers**, comme un directory de fichiers classiques, avec une **racine** par organisation.

> Une organisation peut _migrer_ d'un opérateur à un autre: ce transfert technique des données est génériquement possible, contractuellement c'est une autre affaire.

## Applications / services

Une **application déployée** dispose dans sa configuration des URLs d'accès au(x) service(s) avec lesquels elle travaille:
- le choix de l'opérateur **Rouge** ou **Bleu** pour le service `randos` par exemple est donc inscrit dans l'application déployée.
- une même application logicielle peut en conséquence être déployée plus d'une fois, autant que d'ensemble d'opérateurs des services utilisés.

Cette configuration des accès n'est pas noyée au milieu du logiciel: un utilisateur _terminal_ a les moyens techniques de vérifier que l'application déployée qu'il entend utiliser,
- correspond bien au logiciel _officiel_ (et non pirate) mis en ligne en _source_ par son _éditeur_: sa version a pu être certifiée par une autorité de sécurité indépendante,
- accède bien aux _services officiels_ prévus et non à des sites pirates.

> Il est possible d'accorder sa confiance à une application déployée d'un éditeur la rendant accessible en _open source_ parce qu'il est possible à une entité de certification externe à l'éditeur de vérifier la conformité de ses déploiements.

## Une application terminale peut accéder à plus d'une organisation
Dans le cas de l'application `randos`, un utilisateur peut être membre de plus d'une association de randonneurs: une pour ses randonnées près de chez lui, une autre pour les randonnées de montagne et une troisième pour les treks lointains. Depuis la même application il peut basculer d'une organisation à une autre.

Un gestionnaire de boutiques peut par exemple gérer trois boutiques différentes (trois organisations) avec des rôles différents pour chacune.

Les utilisateurs de Discord accèdent souvent à plusieurs _serveurs_ qui s'ignorent entre eux, ayant des sujets d'intérêt totalement différents.

L'utilisateur qui ouvre sur son terminal son application `randos` peut disposer de pages de synthèse lui montrant ce qui est important pour chacune des associations auxquelles il participe. Pour agir effectivement sur l'une d'entre elles, il basculera sur une page d'accueil spécifique de l'association sélectionnée et ses actions de mises à jour ne porteront que sur celle-là.

> L'application déployée utilisée désignant exactement pour chaque service l'URL de l'opérateur choisi, elle ne peut accéder à plusieurs organisations que pour autant qu'elles soient _hébergées_ chez le même opérateur du service. 

### $$DISCUSSION$$
Cette dernière contrainte peut être libérée à condition que _pour chaque organisation_ l'application terminale sache quel est l'opérateur l'hébergeant:
- par une liste exhaustive embarquée dans la configuration: mais ceci impose un redéploiement d'application à l'apparition d'une nouvelle organisation.
- par _préfixe_ du code de l'organisation (`r` pour `Rouge`, `b` pour `Bleu`) indiquant quel est son opérateur de service: le changement d'opérateur hébergeant pour une organisation implique dans ce cas le changement de son code:
  - le transfert technique des données ne contenant pas le code de l'organisation, à l'occasion d'un changement d'opérateur une organisation peut donc changer de code.
  - rendre pour les utilisateurs ce changement de code _transparent_ (plus ou moins) reste un défi ouvert. Mais faut-il que ce soit _transparent_ ? 
  - le changement d'opérateur pour une _organisation_ est une opération qui va être rare et pas anodine. Quand un utilisateur se _trompe_ de code d'organisation, le service sollicité détecte qu'il ne l'a pas / plus chez lui et incite l'utilisateur à ressaisir son code, c'est un moyen pour lui rappeler que son code autrefois `rorg1` est désormais `borg1`.

On peut aussi envisager un _DNS_ central indiquant pour chaque couple organisation / service, l'URL (ou le code) de l'opérateur correspondant: 
- la gestion de ce _DNS_ implique un point de centralisation, des administrateurs, etc.
- chaque service sollicité par une requête mentionnant une organisation qu'il ne gère pas / plus peut retourner à l'application appelante un _code de redirection_ donnant l'URL du service en charge obtenu du _DNS_: l'application peut gérer un cache de ces redirections afin d'éviter de sur-solliciter les appels au _DNS_.

# Exécution d'une application sur un terminal
Sur un terminal donné, on ne peut pas lancer plus d'une exécution d'une application donnée, par exemple une seule application `randos`.

> Dans le cas d'une application Web-PWA, chaque browser (Firefox, Chrome ...) est vu comme un **terminal différent**: on peut avoir s'exécutant au même instant sur son PC, une même application sous Firefox ET sous Chrome (comme si on avait deux mobiles).

**Une application sur UN terminal** peut avoir trois états:
- être en exécution au **premier plan**. Sa fenêtre est affichée et a le _focus_, elle capte les actions de la souris ou du clavier. Pour un mobile c'est celle (ou l'une des deux ?) visible.
- être en exécution en **arrière plan** : elle a été lancée mais est recouverte par d'autres.
  - sur un browser, c'est un autre onglet qui a le focus ou la fenêtre du browser est en icône: l'utilisateur peut cliquer sur son onglet pour l'amener au premier plan ou sur l'icône du browser dans la barre d'icônes pour l'afficher.
  - sur un mobile elle est cachée mais peut être ramenée au premier plan quand l'utilisateur la choisit dans sa liste des applications _ouvertes mais cachées_.
- être **non lancée**: son exécution n'a pas encore été demandée, ou a été active puis fermée.

### Une application peut _envoyer_ des requêtes aux services
C'est l'application qui appelle par son URL un service qui **traite la requête et retourne un résultat**.
- requêtes et réponses peuvent être volumineuses.

### Une application peut _écouter_ des notifications émises par les services
Une application donnée sur un appareil donné est identifiée par un _jeton_ qui une sorte de numéro de téléphone universel: tout service ayant connaissance de ce jeton peut envoyer des _notifications_ à l'application correspondante sur le poste correspondant.

Une notification ressemble à un SMS:
- son texte est _court_ (certes plus long que celui d'un SMS).
- on ne répond pas à une notification: le service émetteur ne sait rien de la suite donnée, ou non, par l'application destinataire.

> L'application _peut_ en tenir compte et effectuer des traitements et des requêtes ultérieures aux services.

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

Pour un _service donné_ assuré par un opérateur donné il existe deux stockages dédiés:
- une base de données,
- un _storage_ de fichiers.

Les stockages sont _partitionnés_ par _organisation_, une partition pour chaque organisation hébergée par ce service.

### La base de données
Elle gère les documents selon un mode _transactionnel_ (ACID).

Elle gère aussi les _abonnements_ des applications terminales aux _documents (synchronisables)_ qui les intéressent: chaque application sur un appareil a un _token_ qui l'identifie de manière unique. 

> Une _micro base de données locale_ pour chaque application / appareil peut détenir en _cache_ les _documents_ récemment demandés et les _abonnements_ en cours de l'application. 

Quand un ou des documents évoluent par exécution d'une opération, elle retrouve toutes les applications terminales abonnées et effectue une publication de notifications vers elles.

> Chaque application terminale est en conséquence susceptible de s'abonner éventuellement auprès de plusieurs services, y compris si toutes les organisations de son domaine d'intérêt sont gérées par des opérateurs différents.

### Le _Storage_ de fichiers 
Il stocke des _fichiers_ identifiés par leur _path_: la présence de caractères `/` dans un path définit une sorte d'arborescence. Le _contenu_ de chaque fichier est une suite d'octets opaque.

En lui-même il n'est pas soumis à un protocole transactionnel (ACID): sa sécurité transactionnelle est déportée sur la base de données avec un protocole simple à deux phases. Le couple base de données / storage permet de garantir qu'un fichier existe ou non, de manière atomique.

Le Storage permet de disposer d'un volume pratiquement 10 fois plus important à coût identique par rapport à la base de données: de nombreuses applications ont des données historiques / mortes ou d'évolutions sporadiques qui s’accommodent bien d'un support sur Storage.

# Autres notes associées

### _[Documents et fichiers, souscriptions et synchronisations"](tech/Documents.html)_

### _[Utilisateurs et 'coffres forts'"](tech/Safe.html)_

# Services, opérateurs, organisations, opérations, credentials
### Service
Définit une liste d'opérations qui peuvent être invoquées avec leurs signatures.
- code `SVC` : majuscule + 2 à 7 majuscules / chiffres : `AS2`

### Opérateur
Un opérateur fournit des prestations de calcul / stockage de données pour plusieurs services.
- code `$OP`: $ + 2 à 7 majuscules / chiffres : `$RED1`
- **chaque service supporté à son URL**.
- l'URL d'un `service opérateur` peut changer.

### Organisation
Une organisation dispose de ses propres données regroupées par **service**.
- son code `org`: minuscule + 2 à 15 minuscules / chiffres: `test demo dodacoltes`
- pour chaque **service** elle a choisi UN **opérateur**. 
- l'opérateur d'un `organisation service` peut changer.

Un `service opérateur`(une URL) dispose d'un document `singletons` de clé primaire `orgs` donnant pour chaque organisation le couple des codes de la base de données et du storage hébergeant ses données.

    { "demo": ["sqlite_A", "storage_a"], "doda": [...] }

### Opérations standard
L'identifiant complet d'une opération est le couple _service opération_.
- le code d'une opération est un nom de classe.
- elle est invoquée par l'URL du `service opérateur` avec:
  - son **code d'opération** (relatif à son service),
  - un **code organisation** `org` : une opération est strictement dédiée à une seule organisation.

### Opérations d'administration d'un service d'un opérateur
Son URL est celle de son `service opérateur` avec:
- son **code d'opération** (relatif à son service) qui commence par `$`,
- le code de l'opérateur `$OP`.

## Le directory central MASTERDIR
Il est hébergé dans la base de données gérant le _safe générique_, dont l'URL est donnée dans la configuration statique de chaque application.

Comme pour le _Safe générique_, il n'y a qu'un seul MASTERDIR de production.

> Il peut y avoir autant de MASTERDIR de test que souhaité par les développeurs pouvant ainsi disposer chacun d'environnements totalement privatifs.

### Tables: `SAFEURLS SAFEORGS SAFEPEMS SAFE`

#### Table `SAFEURLS`
- `key` : clé primaire, le code d'un service.
- `v` : _epoch_ en secondes de mise à jour.
- `value`: un texte JSON donnant pour chaque opérateur son URL:

    { 
      "$RED1": { "url": "https://..."}, 
      "$BLUE": { "url": "https:// ..."}
    }

#### Table `SAFEORGS`
- `key` : clé primaire, le code d'une organisation.
- `v` : _epoch_ en secondes de mise à jour.
- `value`: un texte JSON donnant pour chaque service le code de l'opérateur qui l'assure:

    { "AS2": "$BLUE", "CG1": "$RED" }

#### Table `SAFEPEMS`
Pour chaque utilisateur, enregistré ou non dans le _safe générique_, un row par utilisateur:
- `key`: clé primaire, userId de l'utilisateur.
- `v` : _epoch_ en secondes de mise à jour.
- `value`: JSON des deux clés publiques de cryptage et vérification (sans les bannières ---BEGIN ...).

    [ "abcd ...", "drfgh ..." ]

#### Table `SAFE`
Pour les utilisateurs dont le safe est hébergé dans le MASTERDIR (générique).

C'est un document avec les colonnes: `id hp0 hr0 hct lam data`

## Status

### Status d'un `service opérateur`
Un Administrateur d'un opérateur peut fermer / ouvrir séparément chacun des services déployés.

Dans la base de données déclaré _de référence_ pour son URL, la table `singletons` à une entrée `status` qui donne en JSON:

    { "at":1771588453502,"st":1,"txt":"hello world!" }

- `at` : date-heure (epoch) de dernière mise à jour du status.
- `st` : état du service. 9: DOWN, 1: UP
- `txt` : texte non crypté destiné à l'affichage informatif dans les applications.

### Status d'une organisation pour un `service opérateur`
Un Administrateur d'un opérateur peut fermer / ouvrir séparément chaque **organisation** qu'il héberge pour chaque service déployé.

Le status d'une organisation est enregistré dans un document:
- `org`: code de l'organisation (et clé primaire).
- ...
- `data`: sérialisation cryptée des propriétés:
  - `at` : date-heure (_epoch_) de dernière mise à jour du status.
  - `st` : état du service. 9: DOWN, 1: UP, 2: READ-ONLY
  - `txt` : texte non crypté destiné à l'affichage informatif dans les applications.
  - _autres propriétés dépendante du service._

#### Discussion
Idéalement on pourrait souhaiter que les _managers_ d'une organisation (pour un service / opérateur) puissent disposer d'une clé par exemple pour que leur _chat_ soit confidentiel, c'est à dire NON lisibles par l'opérateur.
- ceci imposerait de stocker cette clé quelque part, typiquement une sorte d'entrée de _Safe_ pour l'organisation.
- mais ceci impose qu'une ou des personnes en maîtrisent les pseudo / phrase correspondantes: d'où un risque de perte irrémédiable. Ce n'est pas un _simple utilisateur isolé_ qui peut disparaître mais une organisation alors que tout a été fait pour permettre à un _opérateur_ de supporter une organisation simplement par déclaration de l'ID d'un _administrateur_ dans sa configuration de déploiement.
- il faut se résigner à ce que _l'opérateur_ ait accès aux _chats_ des managers d'une organisation.
- ceux-ci peuvent ouvrir des _chats_ privés sur invitation pour échanger en toute confidentialité mais avec la fragilité,
  - que tout nouveau _manager_ n'en fasse pas partie automatiquement,
  - que le _chat privé_ s'auto dissolve quand son dernier membre a disparu (avec perte des échanges).

Un **groupe privé** fonctionne par _cooptation_ qui est le mécanisme permettant de transmettre une clé à un nouvel entrant de par l'action d'un membre actuel.
- une action externe au groupe ne peut jamais inscrire de force un nouvel entrant faute de disposer de la clé.
- c'est une assurance de confidentialité intégrale dans le groupe mais une vulnérabilité pour la vie du groupe lui-même qui peut disparaître par suite de la disparition ou inactivité longue (d'où disparition) de ses membres sans jamais qu'un nouvel entrant puisse y être intégrer de l'extérieur.
- pour rendre ça possible il faut disposer d'une _autorité supérieure_ ayant toutes les clés de tous les groupes privés entraînant le risque d'y faire entrer des indésirables non cooptés.

Certes cette autorité supérieure pourrait n'agir que sur donnée d'un mot de passe par exemple, communiqué hors du système par le _propriétaire contractuel_ du groupe à cette autorité. **Mais** qui enregistrerait le mot de passe de _panique_, qui pourrait le changer, sous quelle authentification ? Double clé avec l'Administrateur ?

## Opérations d'Administration Technique
Tout utilisateur peut être reconnu _Administrateur Technique_ à l'hébergement d'un service par un opérateur: son ID est ajouté aux listes statiques de configuration:
- `MASTERDIRADMINUSERS` : pour le MASTERDIR,
- `ADMINUSERS` : pour les autres services déployés.

Lors du contrôle d'authentification à l'entrée d'une opération requérant un droit d'Administrateur, le userId du requérant est,
- certifié par vérification de la signature du _challenge_ par usage de la clé publique de vérification de cet utilisateur obtenu de la table `SAFEPEMS`,
- par présence du `userId` dans,
  - la liste `ADMINUSERS` du déploiement d'UN service SVC / $OP.
  - la liste `MASTERDIRADMINUSERS` pour l'Administrateur du MASTERDIR.

### Liste des rôles ADMIN s'un utilisateur
Pour pouvoir afficher la page _Administration Technique_, un utilisateur doit auto-déclarer dans son _safe_ la liste des couples `SVC $OP` pour lesquels il a ce pouvoir:
- quand il en ajoute un, le fait qu'il le soit réellement est vérifié.
- si son `userId` a été retiré de la configuration, il doit remettre à jour cette liste.
- sil ne s'inscrit pas de lui-même de facto il en perd le pouvoir simplement par impossibilité d'atteindre la page d'administration.

> La révocation d'un Administrateur se fait en enlevant son ID de la liste `ADMINUSERS / MASTERDIRADMINUSERS` correspondante et en redéployant le logiciel.

## Depuis les _Outils Techniques >> Hot_
Après authentification ce dialogue propose plusieurs actions qui requièrent d'être reconnu comme Administrateur du _MASTERDIR_.

#### Déclaration de l'URL d'un service SVC hébergé par un opérateur $OP
Cette opération créé / met à jour l'URL correspondante pour $OP dans la ligne SVC de la table `SAFEURLS`.

Ceci vaut _déclaration d'existence_ au couple `SVC / $OP`.

> Le service correspondant n'est pas pour autant _ouvert au trafic ou non_ ce qui est une décision de l'Administrateur du service / opérateur (et non de celui du _MASTERDIR_).

#### Activation / révocation d'une organisation `org` pour un `SVC / $OP`
Pour un service donné, une organisation est hébergée par un seul opérateur: c'est en conséquence une tâche d'Administration générale que d'assigner l'organisation pour chaque service à l'opérateur l'hébergeant. 

Le row `org` de la table `SAFEORGS` est créé / mis à jour.

> L'accès à l'organisation correspondante n'est pas pour autant _ouvert au trafic ou non_ ce qui est une décision de l'Administrateur du service / opérateur (et non de celui du _MASTERDIR_).

## Depuis les _Outils Techniques >> Status des Services_
Après authentification ce dialogue propose plusieurs actions qui requièrent d'être reconnu comme Administrateur _DU service_ SVC cité hébergé par _L'opérateur_ $OP cité.

#### Le status de SVC / $OP 
Il peut être mis à _UP ou DOWN_ et être accompagné d'un court texte informatif donné par l'Administrateur.
- les opérations sont bloquées quand le status est DOWN, SAUF celle qui modifie ce status et peut en conséquence le remettre UP et adapter l'information.

#### Le status d'une organisation org hébergée par SVC / $OP
Il peut être mis à _UP LECTURE-SEULE ou DOWN_ et être accompagné d'un court texte informatif donné par l'Administrateur.
- les opérations sont bloquées quand le status est DOWN, SAUF celle qui modifie ce status et peut en conséquence le remettre UP et adapter l'information.

#### La configuration d'une organisation org hébergée par SVC / $OP
Elle consiste à attacher l'organisation à,
- UNE des bases de données gérées par SVC / $OP,
- UN des _storage_ gérées par SVC / $OP,

> Cette configuration est déclarative seulement et ne permet en aucun cas un _transfert_ de base ou de storage, opérations lourdes gérées en ligne de commande par un administrateur système de l'opérateur.

## Credentials
UN _credential_ gère UN droit pour,
- UN utilisateur `userId`,
- accéder aux opérations d'UN service `SVC`
- accédant à UNE organisation `org`.

La cible du credential est identifiée par le couple `role/docId`:
- `role` : UN rôle est matérialisé par le couple `classe.role`,
  - d'UNE classe de document (par exemple `Redacteur`),
  - et éventuellement d'un rôle complémentaire `chef` (quand l'entité correspondante peut être abordée selon plusieurs rôles).
- `docId` est l'identifiant du document correspondant.
  - quand `docId` est vide, le credential a pour cible TOUS les documents de la classe. C'est typiquement le cas pour `Org` n'ayant qu'un document _son ID 1_ n'est pas donnée.

**Plusieurs _versions_** d'un _credential_ `userId SVC org role docId` peuvent exister au cours du temps: chaque version est distingué par `time` la date-heure en secondes de sa déclaration.

> Une **ID aléatoire** est générée à la création d'une version d'un credential.

### Les _faces_ "safe" et "document" d'un credential
**La face _safe_ est enregistrée dans le _Safe_ de l'utilisateur**, avec:
- la **clé privée de signature** de cette version du credential,
- quelques propriétés détaillées ci-après.
- au détail près d'un _commentaire_ facultatif donné par l'utilisateur lui-même, cet objet est immuable.

**La face _document_ est un document enregistré dans la base de données du service**, avec:
- la **clé publique de vérification** de cette version du credential.
- quelques propriétés détaillées ci-après qui peuvent évoluer au cours du temps.

#### Dans une session de l'application
- toutes les faces _safe_ sont lues depuis le _safe_ de l'utilisateur pour les credentials relatifs à un des services accédés par l'application.
- les _documents_ credential de tous les services accédés par l'application relatifs au `userId` de l'utilisateur sont lisibles / synchronisables.
- une application dispose des deux faces de chaque version de credential, mais:
  - ne peut mettre à jour QUE la propriété `comment` de sa face application (celle stockée dans son _Safe_).
  - ne peut QUE lire les _documents_ credential correspondant.

> Un credential qui n'est pas repris par un _document_ (qui n'a que la face _safe_) est ineffectif (et supprimable).

#### Dans une opération d'un service
- tous les _documents_ des versions de credentials citées par l'objet `AuthRecord` attaché à la requête sont lisibles.
- les faces _safe_ ne sont pas accessibles.

### Création d'une version d'un credential
Elle peut être effectuée selon deux modes:
- **attribution directe** à un utilisateur U par un utilisateur A _attributaire_.
- **auto-attribution après invitation** par l'utilisateur U suite à une **invitation** générée par un utilisateur I _invitant_.

#### Attribution directe par A à U
L'attributaire A _connaît_ l'utilisateur U, 
- soit par son `userId` obtenue par l'application ou tout autre moyen,
- soit par son `contact` un pseudo ou phrase unique temporaire que U a inscrit à titre de _pseudo externe_.

L'attributaire A a lui-même un ou des credentials lui donnant accès à une opération d'enregistrement du futur _document_ credential de U:
- il génère un couple de clés de _signature / vérification_.
- construit le document à enregistrer en y incluant la clé publique générée.
- soumet ce document à une opération d'enregistrement qui s'assurera du droit à cet enregistrement et complétera éventuellement celui-ci en retournant le cas échéant quelques données requises par A pour la suite de la procédure.
- construit la face _Safe_ du credential et l'enregistre dans le _safe_ de U.

Cet enregistrement fait apparaître le credential dans un état _en attente_ dans le _safe_ de U:
- il est crypté par le couple clé publique C de U / clé privée de A,
- il est accompagné de la clé publique C de A.
. lors du prochain accès de U à son _safe_ pour chaque credential en attente, U:
  - décrypte l'objet _credential_ par le couple _clé privée de U / clé publique de A_,
  - le ré-encrypte par sa clé `keyK` et le stocke dans son _safe_ (il est devenu _permanent_), le credential _temporaire_ étant détruit.

> Dans ce mode d'attribution directe, **à la limite**, A peut attribuer des droits à U sans son accord, rien que par le fait qu'il connaît le `userId` ou le pseudo / phrase de `contact` de U.

#### Auto-attribution par U suite à invitation de I
Dans ce mode un utilisateur I a enregistré pour U un _document d'invitation_ qui va contenir toutes les données nécessaires à U pour s'auto-enregistrer. 
- une partie de ces données peut être _cryptées_ par I pour n'être lisible que par U.
- les autres propriétés serviront à l'opération d'enregistrement pour juger de son acceptabilité.

Le processus par U est le suivant:
- il accède au document _invitation_.
- il génère un couple de clés signature / vérification pour la version de credential en création.
- il fabrique les deux faces de sa version de credential.
- il stocke sa face _safe_ dans son son safe.
- il soumet le _document credential_ à une opération d'enregistrement en y joignant les références de son invitation. Cette opération vérifie la validité de la demande puis complète éventuellement le _document credential_ et l'enregistre.
- en cas d'échec de cette opération il supprime sa version de credential de son _safe_.

> Dans ce mode par invitation U n'a inscrit qu'un droit qu'il a sollicité et approuvé. En contrepartie, ça oblige U à a) solliciter une invitation, b) en attendre le retour. En mode attribution directe l'attributaire pouvait inscrire directement le droit de U.

### Propriétés de l'objet face _safe_
Parmi les propriétés _communes_ `ID userId SVC org role docId time`,
la propriété `userId` n'est pas stockée, puisque le _safe_ est dédié à ce `userId`.

Les autres propriétés sont:
- `pems`: la clé privée de signature générée pour cette version du credential.
- `comment`: un commentaire libre et facultatif de l'utilisateur qui peut l'aider quand il constitue un profil de session ne reprenant que _certains_ des credentials.
- `name`: le `docId` cible est un code ininterprétable humainement. Toutefois au moment de la création, le créateur peut connaître un _nom / libelle /etc._ explicitement lisible par un humain. Ceci peut aussi aider U quand il constitue un profil de session ne reprenant que _certains_ des credentials.
- `skey` : une clé symétrique AES (facultative). Lire ci-après.

Dans les documents certaines propriétés sont _lisibles_ par le service, d'autre sont _opaques_ pour le service.
- **propriétés lisibles**: c'est l'état normal et le logiciel du service peut les utiliser dans ses traitements.
- **propriétés opaques**: elles sont _cryptées_ par une clé qui n'est disponible QUE dans les applications terminales et sont en conséquences inutilisables par une opération du service.

Dans certains cas des données peuvent être considérées comme _confidentielles_, de lisibilité restreinte, par exemple à un _comité directeur de .._, à un _agent_ pour ses données personnelles, etc. Le principe est que la _clé AES_ qui a rendu ces données _opaques_ aux opérations du service, n'est PAS stockée dans la base de données du service: même en cas de piratage de celle-ci elles restent inviolées, illisibles.

Chaque utilisateur ayant un droit d'accès à ce _comité directeur_ disposera de cette clé dans la propriété `skey` du credential correspondant et pourra ainsi décrypter ces données _opaques_ pour les opérations du service.

`skey` a été transmise:
- dans le cas d'une attribution directe par A: A en est détenteur lui-même.
- dans le cas d'une auto-attribution suite à invitation, l'invitant I, lui-même détenteur de cette clé, l'a inscrite dans l'invitation (cryptée par la clé I / U) lisible que par U.

> Il est toutefois possible que certaines clés d'opacité soient présentes dans un _document_ et non pas uniquement dans les _safes_: elles se trouvent alors elles-mêmes cryptées dans des propriétés _opaques_. Une clé _maîtresse_ `skey` peut servir à _opacifier_ un jeu peut-être important de clés _secondaires_ accessibles de facto dès qu'un utilisateur détient la `skey` _maîtresse_.

### Propriétés du _document_ stocké en base du service
Parmi les propriétés _communes_ `ID userId SVC org role docId time`,
les propriétés `SVC org` ne sont stockées explicitement _dans_ le document (`org` est stockée à part et tous les documents de la base de données sont spécifiques du service `SVC`).

Les autres propriétés sont:
- `pemv`: la clé publique de vérification générée pour cette version du credential.
- `limit`: date-heure en secondes de la limite de validité de cette version. Si absente la validité est éternelle. Elle peut être mise à jour par une opération de prolongation / révocation.
- `cond`: c'est un objet dépendant du rôle du credential contenant les données explicitant les conditions d'exercice du credential: _seuils divers, liste de permissions, etc._

### L'objet `AuthRecord`
Toute opération requérant la présence d'au moins un credential est sollicitée en passant en arguments un objet de classe `AuthRecord`, construit par l'application et ayant les propriétés suivantes:
- `userId`: de l'utilisateur.
- `sessionId`: identifiant de session.
- `time`: date-heure en seconde de création du record.
- _challenge_: propriété virtuelle _userId + '/' + time_
- `userSign`: signature par la clé privée de signature de l'utilisateur, du _challenge_.
- `signatures`: objet ayant une propriété par ID de credential inscrit dans le record donnant la signature du challenge par la clé privée de signature du credential.

Au démarrage d'une opération, le `AuthRecord` joint est scanné et un compte rendu est généré sous forme d'une map `roles`:
- _clé_: `docClass.role/docId`.
- _valeur_: un objet ayant les propriétés suivantes:
  - `cred`: la version du credential la plus récente dont la signature est bonne OU la plus récente (bonne signature ou non)
  - `status`: _true_ si la signature est bonne.

Le compte-rendu est _en échec_ s'il existe une entrée du compte-rendu en status _false_. Quand il existe plusieurs versions pour une même rôle `role.docId`, la plus récente à _true_ est gardée, il suffit donc _qu'une_ version soit acceptable pour valider le credential.
- si la signature du `userId` n'est pas validé, c'est un échec.

Dans le cours du traitement de l'opération, cette map est consulté pour déterminer si l'opération peut ou non être acceptable en fonction de ses propres paramètres, de l'état des documents et des données issues du cond attaché au credential dont la version a été retenue.
