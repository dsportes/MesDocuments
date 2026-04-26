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
- les opérations sont demandées par une application. Une opération,
  - reçoit en entrée des paramètres, 
  - effectue le traitement demandé, 
  - retourne un résultat qui en général va influer sur l'affichage de l'application l'ayant sollicitée.

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

> **L'intégration des données provenant de plusieurs services** se fait au niveau des applications. Par clarté de conception mais surtout parce qu'invoquer une opération demande des _credentials_ cryptés qui ne sont détenus que par les applications mais jamais par les services.

> L'intégration des données provenant de plusieurs organisations pour un service donné se fait au niveau des applications: une _opération_ ne traite qu'une organisation.

### Le "storage de fichiers d'un service", sa partition par "organisation"
Un _storage_ a une structure qui s'apparente à un _file-system_.
- le stockage d'un _fichier_ (contenu binaire crypté) se fait en une opération atomique,
- le stockage de plusieurs fichiers ne fait pas l'objet d'un commit de type ACID (chacun peut ou non avoir été mis à jour).

Le _storage_ est également partitionné par _organisation_.

Comme pour une base de données, au cas où le volume l'exigerait, plusieurs _storage_ peuvent exister, chacun avec sa propre technologie le cas échéant.

> Un _storage_ n'est pas non plus partagé par plusieurs _services_.

> Un storage permet de mémoriser des volumes considérables de données,
>- peu ou pas mises à jour après stockage,
>- dont le contenu est en général _opaque_ pour les services (mais ce n'est pas obligatoire),
>- adapté à l'archivage de données de _legacy_.

### Les "Utilisateurs" et leurs _droits d'accès / credential_ 
Les **utilisateurs** sont identifiés par un identifiant aléatoire et anonyme, sans référence avec des identifiants personnels dans la _vraie_ vie.

Depuis un appareil quelconque un utilisateur peut lancer une application dès lors qu'il en connaît l'URL. Celle-ci peut invoquer des **services** et leurs opérations MAIS toute opération exige en général que l'utilisateur exhibe un ou des _droits d'accès_ appropriés pour l'opération demandée et ses paramètres.

Par exemple une opération d'accès aux données d'un `adhérent` identifié `abcd` va exiger que l'application communique à l'opération un _jeton_ qui prouve que l'utilisateur dispose du droit d'accéder aux données de cet adhérent. Le _droit_ requis peut être différent selon que l'opération effectue une lecture ou une mise à jour de l'adhérent.

Un _droit d'accès_ comporte deux parties:
- **une partie conservée par l'utilisateur** dont le texte comporte les éléments cryptographiques lui permettant de _signer_ chaque jeton attaché à une demande d'une opération.
- **une partie conservée dans la base de données du service pour l'organisation souhaitée** qui permet à l'opération de _vérifier_ que le jeton reçu en paramètre de l'opération est effectivement valide et contient bien les données qu'il prétend détenir.

Ce mécanisme détaillé par ailleurs permet,
- de ne pas stocker dans la base de données les éléments de _signature_,
- de pouvoir refuser des _jetons usurpés_, c'est à dire ayant déjà été présentés une fois et représentés plus tard.

### _Safe Box_ d'un utilisateur
Chaque _droit d'accès_ est un texte long, comportant des textes d'apparence aléatoire, bref impossibles à mémoriser (et à inventer par _force brute_). 

L'utilisateur pourrait certes disposer d'un fichier personnel où il les rangerait mais la sécurité et l'accès depuis plusieurs terminaux à ce fichier exposerait ces données de sécurité _critiques_ aux pertes et aux vols.

Chaque utilisateur dispose à cet effet d'une _Safe Box_ personnelle où ses droits d'accès sont rangés, cryptés et sécurisés. 

La _Safe Box_ d'un utilisateur a pour identifiant celui de l'utilisateur (ou à  l'inverse un utilisateur est identifié par le numéro de sa Safe Box). Il comporte plusieurs _rubriques_:
- son **entête** qui détient les éléments cryptographiques techniques nécessaires à son fonctionnement.
- la **liste de ses droits d'accès**.
- une **liste de terminaux certifiés de confiance**, c'est à dire des terminaux d'où il pourra s'identifier par un code PIN plus simple que son identification _forte_ et sur lesquels chaque application pourra laisser des _documents en mémoire cache_ locale cryptée permettant un usage en _mode avion_.
- une **liste de préférences** de comportement et d'affichage de son choix afin de retrouver en lançant une nouvelle session, l'organisation de l'écran qu'il souhaite, les options de son choix, sa langue de travail, etc.
- une **liste des profils de sessions favorites**, un profil ouvrant une session avec une liste de droits d'accès réduite à ceux requis pour ouvrir une session avec un but spécifique. 

#### Dépôts des Safe Box: _standard_ ou _opérateur spécifique_
Un **dépôt _standard_** est géré: tout utilisateur peut en disposer pour y déposer sa Safe Box.

Mais les utilisateurs peuvent préférer confier leurs données de sécurité à un _opérateur_ en qui ils ont confiance (voire être eux-mêmes ou pour un groupe d'entre eux leur propre opérateur). 

Chaque utilisateur (ou groupes d'utilisateurs) peut déployer son propre dépôt de _Safe Box_ par exemple dans une base de données MySQL d'un site Web de son choix (et sous son entière responsabilité d'administration). Un exemple d'un script PHP est fourni: n'importe qui peut lire le texte et s'assurer de sa non nocivité. Mais l'opérateur choisi peut avoir le sien

Des moyens sont données pour basculer du dépôt _standard_ vers un _dépôt spécifique_ (et réciproquement), ainsi que pour effectuer des _backup_: l'image d'une _Safe Box_ peut être exportée cryptée par une clé détenue par le seul utilisateur.

> Le _contenu_ d'un coffre-fort est lisible _en clair_ **pour son propriétaire et seulement lui**, ... mais étant plein de données cryptographiques le terme _en clair_ est un peu une vue de l'esprit.

### Exécution d'une application en _mode AVION_
Quand un utilisateur a déclaré un ou des terminaux qu'il a certifié **de confiance** et qu'il y ouvre une session d'une application, celle-ci peut utiliser une **mémoire cache de documents et fichiers**, cryptée et sécurisée sur le terminal.

Depuis ce même terminal, l'utilisateur peut rouvrir une session qui s'est antérieurement exécutée sur ce terminal, elle y a été _épinglée_:
- s'il a accès au réseau Internet, le lancement sera rapide du fait que beaucoup de documents n'auront pas à être redemandés aux services, étant déjà _en cache_ (cryptés) dans le terminal.
- s'il n'a pas accès au réseau Internet il peut rouvrir son application en **mode AVION** et accéder (en lecture seulement) aux documents disponibles en cache du terminal du fait d'une exécution antérieure.

### Synthèse
Les **applications** s'exécutent sur le terminal de l'utilisateur où elles ont été chargées depuis leur URL (ou un magasin d'applications).

Elles font appel à des **services de traitement des données** distants, chacun ayant un jeu **d'opérations** pouvant lire / écrire SA **base de données** (éventuellement SES bases en cas de volume excessif) et SON **storage de fichiers** (éventuellement SES) partitionnés par **organisation**.

Chaque requête à une opération est dédiée à UNE organisation et n'accède qu'à la partition de la base de données dédiée à cette organisation et / ou du storage dédié à cette organisation.

Tout **utilisateur** dispose d'une **Safe Box** détenant en particulier ses _droits d'accès_ requis à l'appel des opérations des services par une application. Un utilisateur peut décider de confier la gestion de SA Safe Box, soit au **dépôt standard**, soit à un **dépôt spécifique** géré par l'opérateur de son choix.

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

Le changement de version d'une application PWA est automatique, la vérification d'existence et le téléchargement d'une nouvelle version intervenant au lancement: l'utilisateur est convié à appuyer sur un bouton pour redémarrer celle-ci après installation de la nouvelle version.

> Un site comme `github.io` peut être utilisé comme _magasin d'applications_ Web-PWA: la mise en ligne d'un logiciel sur ce site est simple et gratuite, de même que la mise en ligne de sa documentation / aide en ligne.

### Application de type _mobile_
L'utilisateur l'installe depuis le ou un des magasins d'application supportés par l'OS du mobile.

Le changement de version est en général automatique mais peut être opéré manuellement.

> Il n'y a ensuite quasiment pas de différence perceptible par l'utilisateur à l'utilisation de l'application, il clique sur une icône pour l'ouvrir (la lancer).

> On peut installer une application Web-PWA sur un mobile: elle est ci-après considérée comme application PWA (et non comme _mobile_).

# Services dans le _cloud_

Les applications en exécution sur leur appareil envoient des requêtes à des **services _cloud_**, chacune consistant à invoquer une opération de consultation et/ou de mise à jour des données de l'application dans la base de données ou le storage de fichiers du service.

Un _service_ est techniquement déployé selon des variantes techniques non perceptibles de l'extérieur:
- **Serveurs permanents**: plusieurs processus sont en exécution en permanence afin de traiter les requêtes qui leur parviennent sur l'URL du pool de processus et ont été routées vers l'un ou l'autre.
- **Cloud Functions**: un _serveur éphémère_ du _Cloud_ est lancé pour traiter une demande de service reçue sur son URL:
  - la demande est traitée et le serveur éphémère reste actif un certain temps pour traiter d'autres demandes. Un serveur éphémère peut traiter plusieurs dizaines de demandes en parallèle.
  - en l'absence de nouvelles demandes, un serveur éphémère reste en attente, entre 3 et 60 minutes pour fixer les idées, puis s'interrompt.
  - si le flux des demandes sature la capacité d'un serveur éphémère, une deuxième instance, voire une troisième etc. sont lancées.

> Ces choix de déploiement technique ne sont pas détectables par les applications terminales qui sollicitent des _services_.

## Développement du logiciel (éditeur) et ses déploiements (opérateurs)
Un _service_ correspond à un logiciel qui a été développé par un **éditeur** en vue d'assurer une **finalité applicative** bien délimitée comme par exemple:
- le service `circuitscourts` : gestion de prises de commandes entre des producteurs et des consommateurs.
- le service `discussions` : gestion de groupes de partage de documents et d'échanges interactifs.
- le service `randos` : proposition de randonnées, inscription, échanges, etc.
- le service `boutiques` : gestion du catalogue d'une boutique, de son stock, etc.

Un ou des opérateurs de _déploiement_ peuvent installer / _déployer_ ce logiciel sur le _cloud_ et le rendre accessible pour les sollicitations des applications. Deux opérateurs, par exemple **Rouge** et **Bleu**, peuvent déployer un même service logiciel:
- **Rouge** peut proposer `randos` et `discussions`,
- **Bleu** peut proposer `circuitscourts` et `randos`.

Les déploiements du logiciel `randos` par **Rouge** et **Bleu** ont chacun leur URL d'accès et peuvent différer en _prix_ et _qualité_ d'usage: temps de réponse, disponibilité, restrictions de volume...

## Organisations: services _multi-tenant_
Un service comme `randos`, peut à la manière de Discord, héberger les applications d'associations de randonneurs distinctes: chaque organisation / _tenant_ dispose de _son_ espace de données propre complètement étanche à celui des autres.

Un service `boutiques` propose de gérer plusieurs boutiques, mais de manière à ce que les données de chacune soient totalement isolées de celle des autres.

Les données d'un service d'un opérateur sont stockées dans deux _mémoires persistantes_:
- **UNE base de données** logiquement **strictement partitionnée par organisation**, sans aucun lien ou référence à des données / documents d'une organisation par une autre.
- **UN _storage_ de fichiers**, comme un directory de fichiers classiques, avec une **racine** par organisation.

> Une organisation peut _migrer_ d'un opérateur à un autre: ce transfert technique des données est génériquement possible, contractuellement c'est une autre affaire.

## Applications / services

> Un service MASTER d'accès au **Master Directory** a dans sa base de données une petite table `ZZSVCOPS` ayant une ligne par _service_ qui donne la liste des opérateurs proposant ce service avec l'URL d'accès correspondante.

Une **application déployée** dispose dans sa configuration de l'URL d'accès à **Master Directory** ce qui lui permet d'obtenir pour le service `randos` par exemple les URLS pour les opérateurs **Rouge** ou **Bleu**.

Un utilisateur _terminal_ a les moyens techniques de vérifier que l'application déployée qu'il entend utiliser,
- correspond bien au logiciel _officiel_ (et non pirate) mis en ligne en _source_ par son _éditeur_: sa version a pu être certifiée par une autorité de sécurité indépendante.
- accède bien aux _opérateurs officiels_ prévus et non à des sites pirates.

> Il est possible d'accorder sa confiance à une application déployée d'un éditeur la rendant accessible en _open source_ parce qu'il est possible à une entité de certification externe à l'éditeur de vérifier la conformité de ses déploiements.

## Une application terminale peut accéder à plus d'une organisation
> La table `ZZORGS` du Master Directory dispose d'une ligne par _organisation_ donnant _pour chaque service_ le code de l'opérateur choisi par l'organisation.

Dans le cas de l'application `randos`, un utilisateur peut être membre de plus d'une association de randonneurs: une pour ses randonnées près de chez lui, une autre pour les randonnées de montagne et une troisième pour les treks lointains. Depuis la même application il peut basculer d'une organisation à une autre (le cas échéant avoir des vues les globalisant).

Un gestionnaire de boutiques peut par exemple gérer trois boutiques différentes (trois organisations) avec des rôles différents pour chacune.

Les utilisateurs de Discord accèdent souvent à plusieurs _serveurs_ qui s'ignorent entre eux, ayant des sujets d'intérêt totalement différents.

L'utilisateur qui ouvre sur son terminal son application `randos` peut disposer de pages de synthèse lui montrant ce qui est important pour chacune des associations auxquelles il participe. Pour agir effectivement sur l'une d'entre elles, il sélectionnera celle souhaitée et ses actions de mises à jour ne porteront que sur celle-là.

# Exécution d'une application sur un terminal
Sur un terminal donné, pour une une application donnée, une seule exécution peut être active à un instant donné, par exemple une seule application `randos`.

> Dans le cas d'une application Web-PWA, chaque browser (Firefox, Chrome ...) est vu comme un **terminal différent**: on peut avoir s'exécutant au même instant sur son PC, une même application sous Firefox ET sous Chrome (comme si on avait deux mobiles).

**Une application sur UN terminal** peut avoir trois états:
- être en exécution au **premier plan**. Sa fenêtre est affichée et a le _focus_, elle capte les actions de la souris ou du clavier. Pour un mobile c'est celle (ou l'une des deux ?) visible.
- être en exécution en **arrière plan** : elle a été lancée mais est recouverte par d'autres.
  - sur un browser, c'est un autre onglet qui a le focus ou la fenêtre du browser est en icône: l'utilisateur peut cliquer sur son onglet pour l'amener au premier plan ou sur l'icône du browser dans la barre d'icônes pour l'afficher.
  - sur un mobile elle est cachée mais peut être ramenée au premier plan quand l'utilisateur la choisit dans sa liste des applications _ouvertes mais cachées_.
- être **non lancée**: son exécution n'a pas encore été demandée (ou a été active puis fermée).

### Une application peut _envoyer_ des requêtes aux services
C'est l'application qui appelle par son URL un service qui **traite la requête et retourne un résultat**.
- requêtes et réponses peuvent être volumineuses.

### Une application peut _écouter_ des notifications émises par les services
Une application donnée sur un appareil donné est identifiée par un _jeton_ qui est une sorte de numéro de téléphone universel: tout service ayant connaissance de ce jeton peut envoyer des _notifications_ à l'application correspondante sur le terminal correspondant.

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
Les applications **sourdes** classiques ne peuvent afficher des écrans que sur sollicitation de l'utilisateur. 

L'écran ne se remet à jour que suite à une action de l'utilisateur: si ce dernier ne fait rien, l'écran ne change pas et affiche des données plus ou moins anciennes qui ont pu être déjà modifiées par l'action d'autres utilisateurs, du temps qui passe, etc.

Les applications **écoutantes** peuvent remettre à jour leurs écrans et données détenues localement même sans action d'un utilisateur simplement en fonction des _notifications_ poussées vers elles par les serveurs. Elles _peuvent_ rester à l'écoute (même non lancée) et l'utilisateur peut, à réception d'une notification, rouvrir l'application d'un clic.

# Les stockages des données

Pour un _service donné_ assuré par un opérateur donné il existe deux stockages dédiés:
- une base de données,
- un _storage_ de fichiers.

Les stockages sont _partitionnés_ par _organisation_, une partition pour chaque organisation hébergée par ce service.

### La base de données
Elle gère les documents selon un mode _transactionnel_ (ACID).

Elle gère aussi les _abonnements_ des applications terminales aux _documents (synchronisables)_ qui les intéressent, chaque application sur un appareil ayant un _token_ qui l'identifie de manière unique. 

> Sur un terminal _certifié de confiance_ par un utilisateur, une _micro base de données locale_ pour chaque session d'application _épinglée_ détient en _cache_ les _documents_ récemment demandés et les _abonnements_ en cours de l'application. 

Quand un ou des documents évoluent par exécution d'une opération, celle-ci retrouve toutes les applications terminales abonnées et effectue une publication de notifications vers elles.

> Chaque application terminale est en conséquence susceptible de s'abonner éventuellement auprès de plusieurs services, y compris si toutes les organisations de son domaine d'intérêt sont gérées par des opérateurs différents.

### Le _Storage_ de fichiers 
Il stocke des _fichiers_ identifiés par leur _path_: la présence de caractères `/` dans un path définit une sorte d'arborescence. Le _contenu_ de chaque fichier est une suite d'octets opaque.

En lui-même il n'est pas soumis à un protocole transactionnel (ACID): sa sécurité transactionnelle est déportée sur la base de données avec un protocole simple à deux phases. Le couple base de données / storage permet de garantir qu'un fichier existe ou non, de manière atomique.

Le Storage permet de disposer d'un volume pratiquement 10 fois plus important à coût identique par rapport à la base de données: de nombreuses applications ont des données historiques / mortes ou d'évolutions sporadiques qui s’accommodent bien d'un support sur Storage.

# Autres notes associées

### _[Documents et fichiers, souscriptions et synchronisations"](tech/Documents.html)_

### _[Utilisateurs et leur 'Safe Box'"](tech/Safe.html)_

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

Un `service opérateur`(une URL) dispose d'un document `singletons` de clé primaire `orgs` donnant _pour chaque organisation_ le couple des codes de la base de données et du storage hébergeant ses données.

    { "demo": ["sqlite_A", "storage_a"], "doda": [...] }

### Opérations standard
L'identifiant complet d'une opération est le couple _service opération_.
- le code d'une opération est un nom de classe.
- elle est invoquée par l'URL du `service opérateur` avec:
  - son **code d'opération** (relatif à son service),
  - un **code organisation** `org` : une opération est strictement dédiée à une seule organisation.

### Opérations d'administration d'un service d'un opérateur
Son URL est celle de son `service opérateur` avec:
- son **code d'opération** (relatif à son service) se termine par $ ce qui permet d'ouvrir la base de données par défaut du service (et non celle associée à l'organisation),
- le code de l'opérateur `$OP`.

## Le directory central MASTERDIR
Il est hébergé dans la base de données d'un opérateur dont l'URL est donnée dans la configuration statique de chaque application.

Comme pour le store _générique_ des Safe Box, il n'y a qu'un seul MASTERDIR de production.

> Il peut y avoir autant de MASTERDIR de test que souhaité par les développeurs pouvant ainsi disposer chacun d'environnements totalement privatifs.

### Tables: `ZZSVCOPS ZZORGS ZZUSERS ZZINVITS`

#### Table `ZZSVCOPS`
- `key` : clé primaire, le code d'un service.
- `v` : _epoch_ en secondes de mise à jour.
- `value`: un texte JSON donnant pour chaque opérateur son URL:

    { 
      "$RED1": { "url": "https://..."}, 
      "$BLUE": { "url": "https:// ..."}
    }

#### Table `ZZORGS`
- `key` : clé primaire, le code d'une organisation.
- `v` : _epoch_ en secondes de mise à jour.
- `value`: un texte JSON donnant pour chaque service le code de l'opérateur qui l'assure:

    { "AS2": "$BLUE", "CG1": "$RED" }

#### Table `ZZUSERS`
Voir le document 
Une ligne est déclaré pour chaque utilisateur à l'occasion de la création de sa Safe Box:
- `userId`: clé primaire.
- `hshk`: hash du Strong Hash de sa clé K, servant à vérifier sur certaines opérations que demandeur est bien propriétaire de la Safe Box d'ID userId.
- `hsha1`: hash du Strong Hash de l'alias 1 de l'utilisateur.
- `hsha2`: hash du Strong Hash de l'alias 2 de l'utilisateur.
- `C` : clé publique de cryptage.
- `V` : clé publique de vérification de signature.
- `llq`: dernier trimestre d'accès à la Safe Box.
- `store`: code de l'opérateur gérant la Safe Box si ce n'est pas l'opérateur générique.

Les objectifs de cette table sont les suivants:
- fournir le userId et le store d'un utilisateur depuis un des deux alias qu'il a déclaré.
  - lors du login de l'utilisateur pour lui permettre _d'ouvrir_ sa Safe Box (après avoir fourni sa phrase secrète d'ouverture).
  - pour un utilisateur _sponsor_ d'obtenir le userId d'un utilisateur dont il connaît un alias.
- fournir aux services les clés publiques de cryptage et de vérification de signature d'un utilisateur.
- accessoirement, déterminer les utilisateurs inactifs depuis un certain temps et lancer des _garbage collectors_.

#### Table `ZZINVITS`
Elle est détaillée plus avant dans ce document.

Pour permettre à un utilisateur d'avoir une vue d'ensemble sur ses invitations en cours, cette table énumère les références des invitations ouvertes. Elle comporte les propriétés suivantes:
- `invitId`: c'est sa clé primaire.
- `userId`: utilisateur de l'invitation.
- `v`: version de l'invitation dans la DB.
- `lastView` : date-heure à laquelle l'utilisateur a consulté pour la dernière fois l'invitation.
- `data`: une sérialisation cryptée des propriétés `svc org major minor` immutables de l'invitation.

#### Table `ZZSAFE`
Pour les utilisateurs dont la Safe Box est hébergée dans store _générique_ des Safe Box.
- `userId`: ID du propriétaire de la Safe Box.
- `llq` : numéro du dernier trimestre d'accès.
- `data` : contenu crypté de la Safe Box.

## Status

### Status d'un `service opérateur`
Un Administrateur d'un opérateur peut fermer / ouvrir séparément chacun des services déployés.

Dans la base de données déclarée _de référence_ pour son URL, la table `singletons` à une entrée `status` qui donne en JSON:

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

Lors du contrôle d'authentification à l'entrée d'une opération requérant un droit d'Administrateur, le `userId` du requérant est,
- certifié par vérification de la signature du _challenge_ par usage de la clé publique de vérification de cet utilisateur obtenu de la table `ZZUSERS`,
- par présence du `userId` dans,
  - la liste `ADMINUSERS` du déploiement d'UN service SVC / $OP.
  - la liste `MASTERDIRADMINUSERS` pour l'Administrateur du MASTERDIR.

### Liste des rôles ADMIN s'un utilisateur
Pour pouvoir afficher la page _Administration Technique_, un utilisateur doit auto-déclarer dans sa _Safe Box_ la liste des couples `SVC $OP` pour lesquels il a ce pouvoir:
- quand il en ajoute un, le fait qu'il le soit réellement est vérifié.
- si son `userId` est ensuite retiré de la configuration, il doit remettre à jour cette liste.
- sil ne s'inscrit pas de lui-même de facto il ne peut pas atteindre la page d'administration.

> La révocation d'un Administrateur se fait en enlevant son ID de la liste `ADMINUSERS / MASTERDIRADMINUSERS` correspondante et en redéployant le logiciel.

## Depuis les _Outils Techniques >> Hot_
Après authentification ce dialogue propose plusieurs actions qui requièrent d'être reconnu comme Administrateur du _MASTERDIR_.

#### Déclaration de l'URL d'un service SVC hébergé par un opérateur $OP
Cette opération créé / met à jour l'URL correspondante pour $OP dans la ligne SVC de la table `ZZSVCOPS`.

Ceci vaut _déclaration d'existence_ au couple `SVC / $OP`.

> Le service correspondant n'est pas pour autant _ouvert au trafic ou non_ ce qui est une décision de l'Administrateur du service / opérateur (et non de celui du _MASTERDIR_).

#### Activation / révocation d'une organisation `org` pour un `SVC / $OP`
Pour un service donné, une organisation est hébergée par un seul opérateur: c'est en conséquence une tâche d'Administration générale que d'assigner l'organisation pour chaque service à l'opérateur l'hébergeant. 

Le row `org` de la table `ZZORGS` est créé / mis à jour.

> L'accès à l'organisation correspondante n'est pas pour autant _ouvert au trafic ou non_ ce qui est une décision de l'Administrateur du service / opérateur (et non de celui du _MASTERDIR_).

## Depuis les _Outils Techniques >> Status des Services_
Après authentification ce dialogue propose plusieurs actions qui requièrent d'être reconnu comme Administrateur _DU service_ `SVC` cité hébergé par _L'opérateur_ `$OP` cité.

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

> Cette configuration est déclarative seulement et ne correspond pas à un _transfert technique_ de base ou de storage, opérations lourdes gérées en ligne de commande par un administrateur système de l'opérateur.

# Credentials
Un credential est un document dont l'ID `credId` est aléatoirement généré à sa création:
- de facto il _n'appartient_ qu'au seul utilisateur qui le détient dans sa Safe Box.
- il définit le droit de cet utilisateur à accéder aux opérations d'UN service `svc` spécifiquement pour une organisation `org`.

Sa **cible spécifique** est identifiée par le couple `role/docId`:
- `role` : UN rôle est matérialisé par le couple `classe.role`,
  - d'UNE classe de document (par exemple `Redacteur.`),
  - et éventuellement d'un rôle complémentaire `Redacteur.chef` (quand l'entité correspondante peut être abordée selon plusieurs rôles).
- `docId` est l'identifiant du document correspondant.
  - quand `docId` est vide, le credential a pour cible TOUS les documents de la classe. C'est typiquement le cas pour `Org.` n'ayant qu'un document _son ID 1_ n'est pas donnée.

### `CredSafe` et `Credential`
`CredSafe` est l'objet représentant du credential enregistré dans la _Safe Box_ de l'utilisateur, avec:
- sa **clé privée de signature** du credential,
- quelques propriétés détaillées ci-après.
- au détail près du _commentaire_ facultatif donné par l'utilisateur lui-même, cet objet est immuable.

`Credential` est un document enregistré dans la base de données du service, avec:
- la **clé publique de vérification** de cette version du credential.
- quelques propriétés détaillées ci-après qui peuvent évoluer au cours du temps.

#### Dans une session de l'application
- tous les `CredSafe` sont lus depuis la _Safe Box_ de l'utilisateur pour les credentials relatifs à un des services accédés par l'application.
- les _documents_ correspondants `Credential` dans la DB du service `SVC` et l'organisation `org` sont lisibles par accès par la `credId`.

Une application dispose des deux objets représentant le credential, mais:
- ne peut mettre à jour QUE la propriété `comment` de `CredSafe`.
- ne peut QUE lire le _document_ `Credential` DB correspondant.

#### Dans une opération d'un service
- tous les _documents_ `Credential` citées par l'objet `AuthRecord` attaché à la requête sont lisibles simplement du fait que leur `credId` y figure.
- les objets correspondants `CredSafe` ne sont pas accessibles. L'ID de la Safe Box propriétaire (`userId`) n'est PAS enregistrée dans le document `Credential` afin d'éviter de pouvoir faire des rapprochements basés sur les _credential_ d'un utilisateur.

### Cycle de vie d'un credential
Pour simplifier on se fixe sur UN credential (pour un service, une organisation et un utilisateur) identifié par `role / docId`.
- lors de l'opération **Validation d'une invitation** par l'utilisateur U (voir plus avant) sa création est faite par **inscription conjointe** du credential,
  - dans la Safe Box de l'utilisateur,
  - du document correspondant dans la DB du service.
  - c'est une opération du service SVC qui inscrit ce document et c'est lui qui en gère le contenu.
- des opérations du service ayant connaissance de son `credId` peuvent faire évoluer le document en DB:
  - modification des conditions d'application de la propriété `cond`, offrant plus ou moins de _pouvoirs_ au détenteur du credential.
  - inscription / effacement de la date-heure `limit` invalidant or revalidant l'usage du credential **dans le futur**.
- une **suppression**,
 - soit **conjointe** dans la Safe Box et la DB du service demandée par l'utilisateur (qui efface les deux),
 - soit par fixation par une opération du service de `limit` au jour J (ou dans le passé) correspondant à une suppression logique (puis à terme physique) du _document_.

> Au début d'une opération, un jeton émis par la session est vérifié, la signature du challenge par la clé de signature extraite de la _Safe Box_ est vérifiée par la clé de vérification détenue la DB du service. Mais le credential peut être marqué hors limite: la session n'en n'a pas été informée. 

#### Credentials _brisés_
Un credential est _brisé_ quand,
- il est connu par un `CredSafe` dans la _Safe Box_ de l'utilisateur et inconnu en tant que document `Credential` dans la DB du service: typiquement par une action en deux phases Safe Box / opération du service, la seconde ayant techniquement échoué.
- une limite inférieure au jour J a été inscrite dans la DB par une opération du service, mais l'objet correspondant `CredSafe` n'a pas été détruit et ne peut pas l'être par une opération: l'id de la Safe Box (de l'utilisateur) lui est inconnue mais surtout un service ne peut pas écrire dans une Safe Box.

### Synchronisation des credentials _service_
Pour éviter ces discordances, les documents _Credential_ sont synchronisés en début (et en cours) de session par abonnement à chacun des credential de la session identifiés par leur `credId`:
- la mise à jour du `CredSafe` dans la _Safe Box_ se limite à sa suppression suite à la détection de la suppression du _document_ (ayant un `limit` en deçà du jour courant).
- de facto le `CredSafe` d'un credential ne sert qu'à,
  - porter la clé de signature,
  - porter un commentaire fixé par l'utilisateur pour lui permettre, a) de supprimer les credentials considérés comme désormais sans intérêt, b) de gérer ses _profils de session_ par inclusion des credentials jugés pertinents pour chaque profil.

### Propriétés de `CredSafe`
Les propriétés _communes_ : `credId svc org role docId time`.

Les autres propriétés sont:
- `privs`: la clé privée de signature générée pour cette version du credential.
- `comment`: un commentaire libre et facultatif de l'utilisateur qui peut l'aider quand il constitue un profil de session ne reprenant que _certains_ des credentials.
- `recK` : un record facultatif sérialisé crypté par la clé K de l'utilisateur U. Lire plus avant (validation d'une invitation).

#### Opérations
- `$CreateCred`: création initiale d'un `CredSafe`.
- `$UpdateCredComment`: mise à jour de la seule propriété non immuable d'un `CredSafe`.
- `$AutoRevokeCreds`: suppression d'une liste de `CredSafe` par l'utilisateur.

### Propriétés du _document_ `Credential` stocké en DB du service
Parmi les propriétés _communes_ `credId svc org role docId time`,
les propriétés `svc org` ne sont pas stockées explicitement _dans_ le document (`org` est stockée à part et tous les documents de la base de données sont spécifiques du service `svc`).

Les autres propriétés sont:
- `pubv`: la clé publique de vérification générée pour ce credential.
- `limit`: date-heure en secondes de la limite de validité. Si absente la validité est éternelle. Elle peut être mise à jour par une opération de prolongation / révocation.
- `cond`: c'est un objet dépendant du _rôle_ du credential contenant les données explicitant les conditions d'exercice du credential: _seuils divers, liste de permissions, etc._

#### Le record `recK` d'un objet `CredSafe`
Pour certaines classes de documents, certaines propriétés sont _lisibles_ par le service, d'autre peuvent être _opaques_ pour le service.
- **propriétés lisibles**: c'est l'état normal et le logiciel du service peut les utiliser dans ses traitements.
- **propriétés opaques**: elles sont _cryptées_ par une clé qui n'est disponible QUE dans les applications terminales et sont en conséquences inutilisables par une opération du service.

Dans certains cas des données peuvent être considérées comme _confidentielles_, de lisibilité restreinte, par exemple à un _comité directeur de .._, à un _agent_ pour ses données personnelles, etc. Le principe est que les _clé AES_ qui ont rendu ces données _opaques_ aux opérations du service, ne sont PAS stockées dans la DB du service: même en cas de piratage de celle-ci elles restent inviolées, illisibles.

Chaque utilisateur ayant un droit d'accès à ce _comité directeur_ par exemple disposera de cette clé et la stockera dans la propriété `recK` de l'objet `CredSafe` du credential correspondant: il pourra ainsi décrypter ces données _opaques_ pour les opérations du service.

> Il est toutefois possible que certaines _clés d'opacité_ soient présentes dans un _document_ et non pas uniquement dans les _Safe Box_: elles se trouvent alors elles-mêmes cryptées dans des propriétés _opaques_. Une clé _maîtresse_ dans un `recK` peut servir à _opacifier_ un jeu peut-être important de clés _secondaires_ accessibles de facto dès qu'un utilisateur détient la clé _maîtresse_.

## L'objet `AuthRecord`
Toute opération requérant la présence d'au moins un credential est sollicitée en passant en arguments un objet de classe `AuthRecord`, construit par l'application et ayant les propriétés suivantes:
- `sessionId`: identifiant de session.
- `time`: date-heure en seconde de création du record.
- _challenge_: propriété virtuelle _sessionId '/' time_
- Si l'utilisateur doit être authentifié en tant que tel:
  - `userId`: de l'utilisateur.
  - `userSign`: signature par la clé privée de signature de l'utilisateur, du _challenge_.
- `signatures`: objet ayant une propriété par `credId` de credential inscrit dans le record donnant la signature du challenge par la clé privée de signature du credential.

Au démarrage d'une opération, le `AuthRecord` joint est scanné:
- si la signature du `userId` est présente mais pas validée, c'est un échec.
- une map des  `docClass.role/docId` dont la signature a été vérifiée et validée est établie et pointe sur le document _credential_ correspondant (obtenu depuis son `credId`).
- la liste de ceux en échec de vérification est également générée afin de documenter l'exception de rejet de l'opération associée.

Dans le cours du traitement de l'opération, cette map est consultable par la logique de l'application pour déterminer si l'opération peut ou non être acceptable en fonction de ses propres paramètres, de l'état des documents et des données issues du `cond` attaché au credential validé.

> Une opération _peut_ requérir que l'utilisateur soit _authentifié_, voire qu'il soit inscrit comme _administrateur_ du service.

# Invitations

Une **invitation** est un processus qui part d'une demande explicite d'un utilisateur U **ou** d'une proposition directe d'un sponsor S faite à U pour aboutir en cas de succès du processus à sa **validation** déclenchée par U:
- création de 0 à N1 documents (par exemple _Auteur, Relecteur ..._),
- création de 0 à N2 credentials,
  - d'accès aux documents créés ou existants,
  - d'autres natures (credential _Sponsor_ par exemple).

#### Cycle de vie
Une **invitation** est un dialogue entre,
- un utilisateur U ayant un `userId` bénéficiaire,
- **un ou des sponsors**, utilisateurs ayant un credential de _sponsor_ (ou administrateur). Le dernier _sponsor_ étant intervenu sur l'invitation y a laissé une signature (voir plus loin).

L'objectif recherché est la **validation** de l'invitation par U qui concrétise ce qu'il demandait (et a obtenu).

Au cours de sa vie une invitation subit des interventions de U et de _sponsors_.

In fine une **invitation** disparaît:
- soit par _validation_ par U.
- soit par _destruction_ par U, il y renonce.
- soit par _obsolescence_: une invitation s'auto-détruit au bout de quelques jours sans intervention de U ou d'un sponsor.

Une **invitation** a un _statut d'attente_:
- `waiting` _false_. Le dernier sponsor intervenant a considéré que _la balle est dans le camp de l'utilisateur_ et que c'est à lui d'effectuer une action. Toutefois, même dans cet état un sponsor _peut_ intervenir pour la faire évoluer s'il le souhaite.
- `waiting` _true_. Soit ...
  - l'invitation n'est **pas** en état d'être _validée_ faute de contenir les éléments suffisants que seul un sponsor peut fournir (dans la propriété `etc`).
  - soit elle _pourrait_ être validée en l'état mais U n'est pas satisfait de la proposition et a considéré préférable d'en attendre une autre / meilleure.

> _A priori_ les sponsors ne cherchent à intervenir QUE sur des invitations _en attente_, mais peuvent toutefois à la limite changer leur offre de leur propre initiative.

Une **invitation** a une **ardoise** (`tab`) d'échange textuel:
- elle permet à U et au(x) sponsor(s) de s'expliquer en termes humains.
- elle n'est pas cryptée et ne peut pas l'être les sponsors potentiellement intervenant étant inconnus d'avance.
- chacun peut à son gré en compléter ou remplacer le contenu.

#### Origine d'une invitation
Une invitation peut être _sollicitée_ par U:
- il a besoin _d'une autorisation, d'un compte, d'appartenir à un groupe,_ ... 
- il créé une invitation (avec un `major / minor`) et inscrit sur l'ardoise ses motivations, qui il est, pourquoi il effectue cette demande et toutes précisions souhaitables pour obtenir une proposition d'un sponsor.

Une invitation peut être _NON sollicitée_ par U:
- un sponsor fait une proposition à un utilisateur U dont il a eu par exemple un _alias_ ou qui lui a été recommandé, ou qu'il a rencontré dans la vraie vie. Il a déclaré un `major / minor` et inscrit dans etc les données qui seront nécessaires à la validation. Il a aussi écrit un mot sur l'ardoise pour U.
- U en sera informé et pourra, soit la valider en l'état, soit demander des précisions / améliorations.

> Hormis U, qui peut intervenir sur une _invitation_ ? Voir la section **Sponsors** qui explicite ce sujet en fonction de _l'objet_ d'une invitation codé par le couple `major / minor`.

### Propriétés d'une invitation
Une **invitation** a quelques propriétés immuables dans le temps:
- `invitId` : une ID générée aléatoirement à sa création.
- `svc org` : le service concerné, l'organisation concernée.
- `userId` : l'ID de U.
- `major minor`: _l'objet de l'invitation_ distingué entre un code de catégorie majeur et un code complémentaire précisant sa nature fine.

**Autres informations dynamiques:**
- `v` : c'est la date-heure (_epoch_) de sa dernière évolution, que soit par U ou par un des sponsors.
- `byU` : voir ci-dessus.
- `tab` : voir ci-dessus.
- `etc` : c'est un objet écrit exclusivement par les sponsors intervenant et contenant toutes les données nécessaires à la _validation_ de l'invitation.

A la création / mise à jour d'une invitation par un sponsor, il est vérifié que le credential qu'il a fourni lui permet de traiter l'invitation selon son `major minor`.

L'opération de _validation_ vérifie que le `etc` fourni par le sponsor est bien formé.

#### Document `Invitation`
Il contient les propriétés précédentes et est stocké dans la DB de l'organisation pour le service (`svc org` n'y figurent donc pas explicitement).
- la clé primaire est `invitId`.
- `major` et `[major minor]` sont des index immutables.
- `v` joue aussi le rôle de limite de validité (`maxLife` étant calculé depuis `v`).

#### Opérations de _validation_
Le traitement de validation lancé par U comporte plusieurs phases:
- (1) génération de clés requises pour les credentials à créer et celles à intégrer dans les documents créés ou mis à jour.
- (2) lancement de l'opération _validation_: les arguments comportent les clés _publiques_ ci-dessus.
  - l'opération créé les documents et credentials dont les paramètres sont dans `etc`, complétés des clés publiques des credentials qui viennent d'être reçues en argument.
  - l'opération se termine en supprimant le document `Invitation`. 
- (3) la session de U:
  - supprime la référence de l'invitation dans le _Master Directory_ (table `ZZINVITS`),
  - enregistre dans sa _Safe Box_ les credentials éventuellement créés et dont les clés _privées_ viennent d'être générées en phase (1).

### Remarques
Pour créer un _credential_ il faut disposer d'un couple de clés signature / vérification:
- celle de vérification est transmise en phase (2) par l'opération d'enregistrement (_validation_ par le service).
- celle de signature est enregistrée en phase (3) dans la _Safe Box_.*
- le service NE VOIT JAMAIS PASSER la clé de signature.

## Trace des invitations en _Master _Directory_ en `ZZINVITS`
Un utilisateur U peut être concerné plusieurs couples `service org` et une application donnée peut être utilisatrice de plusieurs services.

Or une invitation n'est enregistrée que dans une seule DB correspondant à son couple `svc org`.

Pour permettre à un utilisateur d'avoir une vue d'ensemble sur ses invitations en cours, une table du _Master Directory_ `ZZINVITS`, énumère les références des invitations ouvertes. Elle comporte les propriétés suivantes:
- `invitId`: c'est sa clé primaire.
- `userId`: utilisateur de l'invitation: cette propriété est indexée afin que l'utilisateur U puisse récupérer toutes ses invitations.
- `v`: version, date-heure de la dernière mise à jour de l'invitation dans la DB. Cette propriété a un double rôle:
  - indexée avec `userId` pour ne récupérer que les invitations ayant changé depuis la dernière demande,
  - destruction automatique au delà de N jours après `v`.
- `lv` : date-heure à laquelle l'utilisateur a consulté pour la dernière fois l'invitation (lui permettant de voir lesquelles ont changé).
- `data`: sérialisation cryptée des propriétés `svc org major minor` immutables de l'invitation. Une session de U sera en mesure d'obtenir auprès du service servant `svc org` le contenu complet de l'invitation.

### Opérations sur `ZZINVITS`
#### `$mdInvitNew` : création (ajout) d'une nouvelle invitation
- **soit par U**: il fournit la signature d'un _challenge_ pour justifier de son droit à cette opération.
- **soir par un sponsor** qui fournit:
  - la référence `spCredId` de son credential de sponsoring justifiant cette action ainsi que `sign` la signature du challenge par la clé privée de signature que le sponsor était censé posséder. 
  - L'opération peut accéder à sa DB (de `svc org`) et lui demander de vérifier,
    - que ce sponsor est bien détenteur de ce credential,
    - que le `role docId` de ce credential lui ouvre bien le droit à traiter le `major minor` de l'invitation à ajouter,
    - et de lui retourner la clé de la vérification de ce credential afin de vérifier que `sign` est la signature du challenge.

**Cas particulier d'un sponsoring par un _administrateur_:**
- le _major_ est `Org.manager` (pas de _minor_).
- les credential de _role_ `Sponsor.` et _docId_ `Org.manager` ne peuvent attribués que par un _administrateur_ du service.
- au lieu de fournir `spCredId`, il est fourni:
  - le `userId` du sponsor censé être _administrateur_,
  - `sign` est la signature du challenge par la clé de signature du sponsor (administrateur).
- l'opération demande au service,
  - de vérifier que `userId` est bien un de ses _administrateur_,
  - si oui de retourner sa clé publique de vérification.

**Après création toutes les propriétés sont constantes sauf deux:**
- `v` : change à chaque mise à jour du document `Invitation` correspondant (c'est la copie de son `v`).
- `lastView` : change a chaque fois que U a signalé avoir _vu_ l'invitation. Si la `lastView` est déjà postérieure à `v`, la mise à jour n'a pas d'intérêt et n'est pas faite.

#### `$mdInvitUpdV` : mise à jour du document de l'invitation
- soit par U soit par un sponsor.
- pas de contrôle en soi, l'opération va récupérer `v` dans la DB de `svc org`.

#### `$mdInvitUpdLV` : mise à jour de `lastView`
- par U : vérification du challenge signé par U.

#### `$mdInvitGetU` : obtention des invitations de U
- par U : vérification du challenge signé par U.
- ne retournent que les seules invitations modifiées après la version passée en argument.

# Sponsors
Un _sponsor_ est un utilisateur qui a un (des) credential de _sponsoring_:
- soit le credential de _role_ `Sponsor.` et _docId_ `Org.manager` (qui donne droit aux opérations qualifiées de management général): il est _sponsor universel_. Un _manager_ peut faire des propositions pour tous les couples `major/minor`.
- soit le credential `Sponsor.` avec un `docId` de la forme `major` ou `major/minor`. Il peut faire des propositions (directe ou en réponse à une demande) restreintes aux `major/minor`.

### `Major`
Une invitation a une cible fonctionnelle bien délimitée dont la liste, fermée, dépend de l'application, représentant en quelque sorte une _classe_ d'invitations. Par exemple:
- `Auteur` : invitation à pouvoir se comporter comme _Auteur_, avoir un document `Auteur` et un credential d'accès (le cas échéant avec des variantes de pouvoir différent).
- `Codir` : invitation à agir en tant que membre du _Comité directeur_ et de prendre les décisions afférentes.
- `Forum` : invitation à faire partie / gérer un _forum de discussion_.

La liste des codes **major** est définie et fermée pour chaque _service_.

### Minor
Une invitation a un code `major` et **peut spécifier un code `minor`** selon son `major`.

Certains `major` n'ont pas de `minor`: `Codir` par exemple, on est membre ou non du _comité directeur_ et il n'y en a qu'un dans l'organisation.

**Certains _majors_ ont une liste fermée de _minors_ possibles**: par exemple un _Auteur_ peut avoir une (ou des) prérogatives de sélection d'auteurs selon un _thème_: _science, politique, sociologie ..._ Cette liste,
- peut évoluer au cours du temps: des _thèmes_ nouveaux peuvent apparaître, des thèmes obsolètes disparaître, etc. mais pas à une fréquence frénétique.
- pour un _major_ donné la liste de ses _minors_ déclarés à un instant donné est assez courte pour permettre d'un désigner un dans une liste à l'écran.

**Enfin certains _majors_ ont une liste ouverte de _minors_ possibles**, par exemple:
- des centaines de forums sont possibles et un utilisateur peut désigner celui de son choix par un code qu'il a obtenu quelque part.
- un _minor_ peut aussi être un code _promotion / campagne_, ayant une durée de vie limitée. Les utilisateurs _sponsor_ ont alors un code _major/minor_ comme `Vente/PROMO5J`.
- soit ces codes ont été publiés quelque part, ou diffusés par un media externe, soit ils se sont transmis de bouche à oreille par un mécanisme de cooptation personnelle.

### Règles
- un **manager** est _sponsor_ universel, peut traiter toutes les demandes de tous _major_.
- un **sponsor** n'ayant qu'un _major_ peut traiter toutes les demandes d'invitation spécifiant ce major quelque soit le _minor_ spécifié dans la demande (ou l'absence de _minor_).
  - un _sponsor_ _Auteur_ peut traiter toutes demandes spécifiant _Auteur, Auteur/science Auteur/politique_.
- un **sponsor** ciblé `major/minor` ne peut traiter que les invitations spécifiant exactement ce code. Par exemple:
  - un sponsor _Auteur/science_ ne peut pas traiter les demandes _Auteur_ ni _Auteur/politique_ mais uniquement celles spécifiant _Auteur/science_.
  - un sponsor _Forum/randojuin26_ ne peut traiter que les invitations spécifiant une volonté de participation au _Forum/randojuin26_.

#### Usage d'un texte de _motivation_ sur l'ardoise
Quand un utilisateur U fait sollicite une invitation en spécifiant un major ou même un major/minor, le texte de motivation sur l'ardoise va aider un _sponsor_ la traitant à personnaliser son acceptation. Par exemple:
- _Forum/randojuin26_ : mais le cas échéant il peut y avoir plusieurs forums pertinents, lequel choisir ?
- pour assumer quelle fonction: _simple participant, organisateur, etc._
- ceci va influer sur,
  - le choix du ou des documents à créer : _inscription, etc._
  - les pouvoirs à conférer dans le ou les credentials accordés: par exemple droit à être sponsor soi-même, à avoir des droits d'animation ou non, etc.

### Retrait des droits de sponsoring
Un _sponsoring_ correspondant à un credential, la logique applicative peut avoir des opérations invalidant un credential de `Sponsor.` (comme de tout autre rôle).
