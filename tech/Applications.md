---
layout: page
title: Architecture pour des applications réactives
---

Le fonctionnement d'une **application réactive** fait intervenir plusieurs éléments logiciels décrits dans le chapitre ci-après.

# Vue générale

### Les "applications"
Les utilisateurs peuvent exécuter une **application** sur un de leurs _appareils / terminaux_ munis d'un moyen de communication avec un humain (écran, clavier, souris ...). Selon la variante technique choisie, un utilisateur lance une application portant un nom (comme `monAppli`):
- soit dans un browser _en ouvrant la page Web_ de l'application: l'URL désigne où le logiciel de l'application est stocké.
- soit _en l'installant_ sur un de ses appareils depuis un _magasin d'applications_ puis en la lançant.

> Sur un terminal donné, l'application `monAppli` est ou n'est pas en exécution, il n'est pas possible d'en lancer plusieurs simultanément.

### Les "services" de traitement des données
Un **service**:
- porte un nom qui qualifie synthétiquement son _objet / fonctionnalité_.
- il supporte plusieurs **opérations**: il correspond à un logiciel qui peut être déployé par un ou plusieurs _opérateurs_.
- les opérations sont demandées par une application en citant l'URL qu'un opérateur a utilisée pour déployer le service. Une opération,
  - reçoit en entrée des paramètres, 
  - effectue le traitement demandé, 
  - retourne un résultat qui en général va influer sur l'affichage de l'application l'ayant sollicitée.

En première approche lorsqu'une opération d'un **service** (déployé par un _opérateur_) est invoquée, un **programme** démarre _quelque part sur Internet_, exécute le traitement demandé puis s'arrête.

Si logiquement ceci est perçu comme tel vu de l'extérieur, le scénario _technique_ est un peu différent:
- au lieu de se terminer après la fin de l'opération, le programme reste _vivant_ en attendant qu'une autre opération soit demandée afin que l'énergie de calcul dépensée pour le chargement du programme soit _amortie_ sur un plus grand nombre d'opérations.
- le programme de plus est capable de traiter plusieurs opérations en parallèle.
- de facto le programme ne s'arrête que quand aucune demande d'opérations n'est parvenue _pendant un certain temps_.
- si le service est très sollicité, plusieurs programmes peuvent être lancés, sur des calculateurs différents le cas échéant, afin d'écouler le trafic des demandes d'opérations.

> Le temps au bout duquel un programme de traitement du service s'arrête en l'absence de trafic est un des paramètres de configuration de l'installation du service, de même que le nombre maximal de programmes s'exécutant en parallèle. 

> Certaines configurations peuvent fixer un nombre fixe de ces exécutions et spécifier que les programmes ne s'arrêtent pas même en l'absence de trafic: les choix résultent d'une valorisation économique dépendant des tarifs des fournisseurs de traitements à distance.

> UNE **application** donnée, par exemple `monAppli`, peut faire appel à plusieurs **services**, par exemple `assocs` et `rando`. Une application qui ne fait appel à aucun service a un comportement de _calculette_ et n'utilise aucune donnée externe.

### La "base de donnée d'un service", sa partition par "organisation"
En première approche un **service** déployé (`rando` par exemple) a **SA** base de données (`DB-A` par exemple): deux services différents ne partagent pas une même base.

La base de données est **partitionnée** par **organisation**:
- Les données relatives à une organisation `amis94` sont totalement disjointes de celles de l'organisation `balad59` mais la structure des données est unique.
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

### Les "Utilisateurs" et leurs _pouvoirs / credentials_ 
Les **utilisateurs** sont identifiés par un identifiant aléatoire et anonyme, sans référence avec des identifiants personnels dans la _vraie_ vie.

Depuis un appareil quelconque un utilisateur peut lancer une application dès lors qu'il en connaît l'URL. Celle-ci peut invoquer des **services** et leurs opérations MAIS toute opération exige en général que l'utilisateur exhibe un ou des _pouvoirs_ appropriés pour l'opération demandée et ses paramètres.

Par exemple une opération d'accès aux données d'un `adhérent` identifié `abcd` va exiger que l'application communique à l'opération un _jeton_ qui prouve que l'utilisateur dispose du pouvoir d'accéder aux données de cet adhérent. Le _pouvoir_ requis peut être différent selon que l'opération effectue une lecture ou une mise à jour de l'adhérent.

Un _pouvoir_ comporte deux parties:
- **une partie conservée par l'utilisateur** dont le texte comporte les éléments cryptographiques lui permettant de _signer_ chaque jeton attaché à une demande d'une opération.
- **une partie conservée dans la base de données du service pour l'organisation souhaitée** qui permet à l'opération de _vérifier_ que le jeton reçu en paramètre de l'opération est effectivement valide et contient bien les données qu'il prétend détenir, bref que l'utilisateur détient bien le _pouvoir_ qu'il prétend avoir.

Ce mécanisme détaillé par ailleurs permet,
- de ne pas stocker dans la base de données les éléments de _signature_,
- de pouvoir refuser des _jetons usurpés_, c'est à dire ayant déjà été présentés une fois et représentés plus tard.

### _Safe Box_ d'un utilisateur
Chaque _pouvoir_ est un texte long, comportant des données d'apparence aléatoire, bref impossibles à mémoriser (et à inventer par _force brute_). 

L'utilisateur pourrait certes disposer d'un fichier personnel où il les rangerait mais la sécurité et l'accès depuis plusieurs terminaux à ce fichier exposerait ces données de sécurité _critiques_ aux pertes et aux vols.

Chaque utilisateur dispose à cet effet d'une _Safe Box_ personnelle où ses pouvoirs sont rangés, cryptés et sécurisés. 

La _Safe Box_ d'un utilisateur a pour identifiant celui de l'utilisateur (ou à  l'inverse un utilisateur est identifié par le numéro de sa Safe Box). Il comporte plusieurs _rubriques_:
- son **entête** qui détient les éléments cryptographiques techniques nécessaires à son fonctionnement.
- la **liste de ses pouvoirs**.
- une **liste de terminaux certifiés de confiance**, c'est à dire des terminaux d'où il pourra s'identifier par un code PIN plus simple que son identification _forte_ et sur lesquels chaque application pourra laisser des _documents en mémoire cache_ locale cryptée permettant un usage en _mode avion_.
- une **liste de préférences** de comportement et d'affichage de son choix afin de retrouver en lançant une nouvelle session, l'organisation de l'écran qu'il souhaite, les options de son choix, sa langue de travail, etc.
- une **liste des profils de sessions favorites**, un profil ouvrant une session avec une liste de pouvoirs réduite à ceux requis pour couvrir un but spécifique. 

#### Dépôts des Safe Box: _standard_ ou _opérateur spécifique_
Un **dépôt _standard_** est géré: tout utilisateur peut en disposer pour y déposer sa Safe Box.

Mais les utilisateurs peuvent préférer confier leurs données de sécurité à un _opérateur_ en qui ils ont confiance (voire être eux-mêmes ou pour un groupe d'entre eux leur propre opérateur). 

Chaque utilisateur (ou groupes d'utilisateurs) peut déployer son propre dépôt de _Safe Box_ par exemple dans une base de données MySQL d'un site Web de son choix (et sous son entière responsabilité d'administration). Un exemple d'un script PHP est fourni: n'importe qui peut lire le texte et s'assurer de sa non nocivité. Mais l'opérateur choisi peut avoir le sien

Des moyens sont données pour basculer du dépôt _standard_ vers un _dépôt spécifique_ (et réciproquement), ainsi que pour effectuer des _backup_: l'image d'une _Safe Box_ peut être exportée cryptée par une clé détenue par le seul utilisateur.

> Le _contenu_ d'une Safe Box est lisible _en clair_ **pour son propriétaire et seulement lui**, ... mais étant plein de données cryptographiques le terme _en clair_ est un peu une vue de l'esprit.

### Exécution d'une application en _mode AVION_
Quand un utilisateur a déclaré un ou des terminaux qu'il a certifié **de confiance** et qu'il y ouvre une session d'une application, celle-ci peut utiliser une **mémoire cache de documents et fichiers**, cryptée et sécurisée sur le terminal.

Depuis ce même terminal, l'utilisateur peut rouvrir une session qui s'est antérieurement exécutée sur ce terminal, elle y a été _épinglée_:
- s'il a accès au réseau Internet, le lancement sera rapide du fait que beaucoup de documents n'auront pas à être redemandés aux services, étant déjà _en cache_ (cryptés) dans le terminal.
- s'il n'a pas accès au réseau Internet il peut rouvrir son application en **mode AVION** et accéder (en lecture seulement) aux documents disponibles en cache du terminal du fait d'une exécution antérieure.

### Synthèse
Les **applications** s'exécutent sur le terminal de l'utilisateur où elles ont été chargées depuis leur URL (ou un magasin d'applications).

Elles font appel à des **services de traitement des données** distants, chacun ayant un jeu **d'opérations** pouvant lire / écrire SA **base de données** (éventuellement SES bases en cas de volume excessif) et SON **storage de fichiers** (éventuellement SES) partitionnés par **organisation**.

Chaque requête à une opération est dédiée à UNE organisation et n'accède qu'à la partition de la base de données dédiée à cette organisation et / ou du storage dédié à cette organisation.

Tout **utilisateur** dispose d'une **Safe Box** détenant en particulier ses _pouvoirs_ requis à l'appel des opérations des services par une application. Un utilisateur peut décider de confier la gestion de SA Safe Box, soit au **dépôt standard**, soit à un **dépôt spécifique** géré par l'opérateur de son choix.

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

> Un service **Master Directory** a dans sa base de données une petite table `ZZSVCOPS` ayant une ligne par _service_ qui donne la liste des opérateurs proposant ce service avec l'URL d'accès correspondante.

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
- **chaque service supporté a son URL**.
- l'URL d'un `service opérateur` peut changer.

### Organisation
Une organisation dispose de ses propres données regroupées par **service**.
- son code `org`: minuscule + 2 à 15 minuscules / chiffres: `test demo amis94`
- pour chaque **service** elle a choisi UN **opérateur**. 
- l'opérateur d'un `organisation service` peut changer.

Un `service opérateur`(une URL) dispose d'un document `singletons` de clé primaire `orgs` donnant _pour chaque organisation_ le couple des codes de la base de données et du storage hébergeant ses données.

    { "demo": ["sqlite_A", "storage_a"], "amis94": [...] }

### Opérations standard
L'identifiant complet d'une opération est le couple _service opération_.
- le code d'une opération est un nom de classe.
- elle est invoquée par l'URL du `service opérateur` avec en arguments:
  - son **code d'opération** `opName` (relatif à son service),
  - un **code organisation** `org` : une opération est strictement dédiée à une seule organisation.

### Opérations d'administration d'un service d'un opérateur
Son URL est celle de son `service opérateur` avec:
- son **code d'opération** (relatif à son service) se termine par `$` ce qui permet d'ouvrir la base de données par défaut du service (et non celle associée à l'organisation),
- le code de l'opérateur `$OP`.

## Le directory central MASTER DIRECTORY
Il est hébergé dans la base de données d'un opérateur dont l'URL est donnée dans la configuration statique de chaque application.

Comme pour le store _générique_ des Safe Box, il n'y a qu'un seul MASTER DIRECTORY de production.

> Il peut y avoir autant de MASTER DIRECTORY de test que souhaité par les développeurs pouvant ainsi disposer chacun d'environnements totalement privatifs.

### Tables: `ZZSVCOPS ZZORGS ZZUSERS ZZCASES`

#### Table `ZZSVCOPS`
- `key` : clé primaire, le code d'un service.
- `v` : _epoch_ en secondes de mise à jour.
- `value`: un texte JSON donnant pour chaque opérateur son URL:

    {
    "RANDO" : { 
      "$RED": { "url": "https://..."}, 
      "$BLUE": { "url": "https:// ..."}
    },
    "ASSOCS": {}
    }

#### Table `ZZORGS`
- `key` : clé primaire, le code d'une organisation.
- `v` : _epoch_ en secondes de mise à jour.
- `value`: un texte JSON donnant pour chaque service le code de l'opérateur qui l'assure:

    {
      "amis94": { "RANDO": "$BLUE", "ASSOCS": "$RED" },
      "balad59": { "RANDO": "$BLUE", "ASSOCS": "$RED" },
    }

#### Table `ZZUSERS` 
Une ligne est déclarée pour chaque utilisateur à l'occasion de la création de sa Safe Box:
- `userId`: clé primaire.
- `hshk`: hash du Strong Hash de sa clé K, servant à vérifier sur certaines opérations que le demandeur est bien propriétaire de la Safe Box d'ID userId.
- `hsha1`: hash du Strong Hash de l'alias 1 de l'utilisateur.
- `hsha2`: hash du Strong Hash de l'alias 2 de l'utilisateur.
- `C` : clé publique de cryptage.
- `V` : clé publique de vérification de signature.
- `llq`: dernier trimestre d'accès à la Safe Box.
- `store`: code de l'opérateur gérant la Safe Box si ce n'est pas l'opérateur générique.

    {
      "qzFuser1...": { "alias": ["Leon27...", ""], "store": ""},
      "9Kvuser2...": { "alias": ["Paulo...", "BigMoi"], "store": "$RED"},
    }

Les objectifs de cette table sont les suivants:
- fournir le `userId` et le `store` d'un utilisateur depuis un des deux alias qu'il a déclaré.
  - lors du login de l'utilisateur pour lui permettre _d'ouvrir_ sa Safe Box (après avoir fourni sa phrase secrète d'ouverture).
  - pour un utilisateur _sponsor_ d'obtenir le userId d'un utilisateur dont il connaît un alias.
- fournir aux services les clés publiques de cryptage et de vérification de signature d'un utilisateur.
- accessoirement, déterminer les utilisateurs inactifs depuis un certain temps et lancer des _garbage collectors_.

#### Table `ZZCASES`
Elle est détaillée plus avant dans ce document.

Pour permettre à un utilisateur d'avoir une vue d'ensemble sur ses demandes / propositions en cours, cette table énumère les références de ses _cases_ ouverts. Elle comporte les propriétés suivantes:
- `caseId`: c'est sa clé primaire.
- `userId`: utilisateur de l'invitation.
- `v`: version de l'invitation dans la DB.
- `data`: une sérialisation cryptée des autres propriétés.

#### Table `ZZSAFE`
Pour les utilisateurs dont la Safe Box est hébergée dans le store _générique_ des Safe Box.
- `userId`: ID du propriétaire de la Safe Box.
- `llq` : numéro du dernier trimestre d'accès.
- `data` : contenu crypté de la Safe Box. Sections: `auth devices, creds profiles prefs`

    {
      "qzFuser1...": { auth:{}, devices:{}, creds:{}, profiles:{}, prefs:{} },
    }

## Schéma général : exemple

<img src="../tech/archi-1.svg" style="background-color:white">

#### Scenario
##### Obtention de la Safe Box
- L'utilisateur d'alias `Leon27` ouvre l'application **MesRandos** qui se trouve hébergée sous _github.io_ à l'URL `https://jollyapps.github.io/mesrandos`
- Il saisit son alias `Leon27` :
  - l'application consulte le _Master Directory_ dont l'URL figure dans sa configuration: la réponse est tirée de `USERS` qui indique que l'alias `Leon27` est bien enregistré et correspond au userId `qsduUs1` dont la Safe Box est hébergée par le _store_ `standard`.
- L'utilisateur saisit sa phrase secrète: le service _Safe Box standard_ vérifie que cette phrase secrète est bien celle enregistrée pour ce userId et le contenu de la Safe Box est copié dans la session de l'application.
- Dès lors `Leon27` peut utiliser l'application qui dispose de ses droits d'accès dans sa Safe Box.

##### Lancement d'une opération
L'application _MesRandos_ a besoin de solliciter une opération `op1` du service `RANDO` sachant que l'utilisateur a désigné son organisation `amis94` dans laquelle il il a un droit d'accès `Animateur/wxfr`.
- l'application demande au Master Directory l'URL du service correspondant:
  - Dans `ORGS` il obtient que l'organisation `amis94` a son service `RANDO` hébergé par l'opérateur `$BLUE`.
  - Dans `SVCOPS` il obtient que le service `RANDO` est assuré par l'opérateur `$BLUE` à l'URL `rndx.blue.org`.
- l'application envoie donc sa requête à cette URL.
- une table locale au service dans base de données _maître_ indique que l'organisation `amis94` est gérée par la base de données `DB-A`.
- l'opération accède aux données / traitement demandé:
  - elle a vérifié que la session disposait bien du droit d'accès correspondant identifié `RANDO/amis94/Animateur/wxfr`.
  - la session a signé un jeton par sa clé de signature et le service a vérifié par la clé de vérification détenue dans DB-A que ce jeton était bien signé.

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

## Opérations d'Administration Technique
Tout utilisateur peut être reconnu _Administrateur Technique_ à l'hébergement d'un service par un opérateur: son ID est ajouté aux listes statiques de configuration:
- `MASTERDIRADMINUSERS` : pour le MASTER DIRECTORY,
- `ADMINUSERS` : pour les autres services déployés.

Lors du contrôle d'authentification à l'entrée d'une opération requérant un droit d'Administrateur, le `userId` du requérant est,
- certifié par vérification de la signature du _challenge_ par usage de la clé publique de vérification de cet utilisateur obtenu de la table `ZZUSERS`,
- par présence du `userId` dans,
  - la liste `ADMINUSERS` du déploiement d'UN service SVC / $OP.
  - la liste `MASTERDIRADMINUSERS` pour l'Administrateur du MASTER DIRECTORY.

### Liste des rôles ADMIN s'un utilisateur
Pour pouvoir afficher la page _Administration Technique_, un utilisateur doit auto-déclarer dans sa _Safe Box_ la liste des couples `SVC $OP` pour lesquels il a ce pouvoir:
- quand il en ajoute un, le fait qu'il le soit réellement est vérifié.
- si son `userId` est ensuite retiré de la configuration, il doit remettre à jour cette liste.
- s'il ne s'inscrit pas de lui-même, de facto il ne peut pas atteindre la page d'administration.

> La révocation d'un Administrateur se fait en enlevant son ID de la liste `ADMINUSERS / MASTERDIRADMINUSERS` correspondante et en redéployant le logiciel.

## Depuis les _Outils Techniques >> Hot_
Après authentification ce dialogue propose plusieurs actions qui requièrent d'être reconnu comme Administrateur du _MASTER DIRECTORY_.

#### Déclaration de l'URL d'un service SVC hébergé par un opérateur $OP
Cette opération créé / met à jour l'URL correspondante pour $OP dans la ligne SVC de la table `ZZSVCOPS`.

Ceci vaut _déclaration d'existence_ au couple `SVC / $OP`.

> Le service correspondant n'est pas pour autant _ouvert au trafic ou non_ ce qui est une décision de l'Administrateur du service / opérateur (et non de celui du _MASTER DIRECTORY_).

#### Activation / révocation d'une organisation `org` pour un `SVC / $OP`
Pour un service donné, une organisation est hébergée par un seul opérateur: c'est en conséquence une tâche d'Administration générale que d'assigner l'organisation pour chaque service à l'opérateur l'hébergeant. 

Le row `org` de la table `ZZORGS` est créé / mis à jour.

> L'accès à l'organisation correspondante n'est pas pour autant _ouvert au trafic ou non_ ce qui est une décision de l'Administrateur du service / opérateur (et non de celui du _MASTER DIRECTORY_).

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

> Au début d'une opération, un jeton émis par la session est vérifié, la signature du challenge par la clé de signature extraite de la _Safe Box_ est vérifiée par la clé de vérification détenue la DB du service. Mais le credential peut être marqué hors limite: la session n'en n'a pas été informée. 

# Credentials (pouvoirs)
Un _credential_ est matérialisé **en deux parties en _Safe Box_ et dans un _document_** avec le but d'autoriser pour son service `svc` et son organisation `org`, 
- la lecture / synchronisation de documents,
- plus généralement l'exécution d'opérations.

> Un credential est attaché à un **document maître** de classe et ID `docCl docId`.

> Un credential en _safe box_ est une copie _retardée_ du document correspondant, mais une copie synchronisée en cours de session.

> Un credential reçoit à sa création un couple de clés de _signature / vérification_ `privs pubv` et un couple de clés de _décryptage / cryptage_ `privd pubc`.

### Propriétés d'un credential
#### Propriétés _immuables_
- `svc org` : le service et l'organisation d'exercice du credential. Ces deux propriétés sont _implicites_ dans le document qui comme tout document est relatif à un couple `svc org`.
- `docCl docId` : identifiant du _document_ auquel le credential est attaché et dont il contrôle les opérations.
- `credId` : identifiant absolu attribué à la création.
- `privs privd`: UNIQUEMENT en _Safe Box_.
- `pubv pubc`: UNIQUEMENT dans le document.

> Pour un utilisateur donné, il ne peut exister qu'un seul credential pour le quadruplet `svc org docCl docId`. `credId` _aurait pu_ être le hash de `[userId svc org docCl docId]`: toutefois, ce n'est pas le cas car il deviendrait possible par une opération de savoir quel est l'utilisateur détenteur d'un credential en scannant les ID de tous les utilisateurs.

#### Propriétés présentes en _Safe box_ MAIS PAS dans le document _credential_
Ces propriétés accessibles par une session d'application ne quittent pas la _Safe Box_ et ne sont ni lisibles ni modifiables par une opération du service.
- `nameK` : propriété facultative de texte libre crypté par la clé `keyK` de l'utilisateur utilisée à l'affichage pour expliciter le `docId` souvent constitué d'un code long et non significatif. Donne par exemple un _nom d'auteur_ parlant pour l'utilisateur à la place de l'id aléatoire `auteurId`.
- `dockeyK`: `dockey` est une clé de cryptage immuable associée au document _maître_. Elle est stockée en `dockeyK` cryptée par la clé `keyK` du détenteur du credential.
  - à la création du document _maître_ c'est l'utilisateur créateur qui l'a générée et inscrite dans le credential.
  - en cours de vie du document _maître_, lorsque B créera son propre credential sur ce document, il recevra cette clé de la part d'un utilisateur A qui en détenait une (transmise cryptée par la clé de cryptage de B / A).

#### Autres propriétés: spécifiques de la classe `docCl` du document _maître_ 
- `opaque`: cet objet, quand il existe, est crypté par `dockey` ce qui en rend le contenu _opaque_ aux opérations.
  - il permet à chaque utilisateur d'exposer des informations sur lui-même visibles de tous les autres ayant un credential **sur le même document maître**.
  - par convention `toString(opaque)` retourne un surnom / nom / pseudo ... à propos de l'utilisateur du credential.
- `more`: cet objet N'EST MODIFIABLE QUE par une opération mais est _lisible_ par les applications.
  - `limit` : la date-heure (_epoch_ en secondes) limite de validité du credential qui est considéré comme inexistant au-delà de cette limite quand elle est est non nulle. C'est la seule propriété toujours présente dans `more`.
  - Autres à titre _d'exemple_:
    - `mandats` : début et fin de _mandats_ attribués à l'utilisateur l'autorisant à agir selon telle ou telle responsabilité / pouvoir.
    - `lectureSeule` : les données du _document_ ne peuvent qu'être lues par les opérations sollicitées par les opérations invoquées par l'utilisateur détenteur du credential.

##### `docKey` du _document maître_
Certaines classes de documents ont pour chaque document une clé AES symétrique de cryptage utilisée pour rendre _opaque_ certaines propriétés du document aux opérations du service qui ne peuvent pas en voir le contenu. Par exemple un texte d'information confidentiel, d'autres clés diverses, etc. qui ne pourront jamais être lisibles même en dérobant frauduleusement le contenu de la DB.

### Credential _embarqué_ DANS son document _maître_
Dans cette première approche un credential est un _objet_ attaché à SON _document maître_ par sa propriété `creds`: c'est une **map** dont la clé est `credId` et la valeur un _objet_ `cred` qui contrôle l'authentification d'un accès au _document maître_ et les conditions dans lesquelles les opérations peuvent agir dessus.

### Credential implémenté par un _document séparé_
L'approche _embarquée_ pose un problème de _volume_ quand un grand nombre de credentials peuvent être attachés à un même document maître. Par exemple pour un _groupe_ de quelques centaines de membres (donc d'autant de _credentials_), le volume du _document_ représentant le groupe pourrait devenir considérable.

Dans ce cas un _document_ `Credential` séparé est créé:
- classe: `Credential`
- clé primaire: `credId`:
- l'indexation `docCl docPk` permet d'acquérir la collection des _credentials_ d'un même document maître.
- `cred` : le même objet `cred` que quand le credential est _embarqué_ dans son document maître.

> La configuration _statique_ des classes de documents indique pour chacune si les credentials sont _embarqués_ ou non.

### Classes _virtuelles_ de documents maîtres
Usuellement le `docCl` d'un credential désigne une classe de documents dont il existe de _vraies_ instances et c'est obligatoirement le cas pour les credentials _embarqués_.

Il est aussi possible de désigner des classes _virtuelles_, n'ayant aucune instance de documents MAIS ayant des credentials rattachées. Par exemple:
- on définit une classe `Section` d'auteurs. Les auteurs sont rattachés à quelques _sections_ (_Roman Nouvelle Science ..._ ) énumérées par une simple liste _configurable_ sans qu'aucun document ne matérialise par exemple `Section Science`.
- `Section` est un nom de classe de document _virtuelle_, `Roman Nouvelle Science` étant des **identifiants** pré-déclarés.
- on peut déclarer des credentials attachés par exemple à un _document maître virtuel_ `Section Science`. Un utilisateur détenant un tel credential a le _pouvoir_ d'enregistrer des auteurs dans cette section (voire de les changer de section, etc. ceci dépendant du détail du credential).

### Lecture d'un `cred` dans une session et une opération
Dans une session il est possible de lire les objets `creds` associés aux credentials détenus dans sa _Safe Box_ et d'en avoir un affichage détaillé:
- les propriétés `name opaque` sont les seules qu'une session d'application peut mettre à jour après création, `docKey` étant disponible mais immuable.

Dans une opération pour contrôler ce qu'elles peuvent faire ou non depuis un document _maître_ (virtuel ou réel), il lui faut accéder au credential correspondant:
- les propriétés `name dockey` n'y figurent pas (étant indéchiffrables pour l'opération comme pour tout autre utilisateur).
- `opaque` (cryptée par la `dockey` du document maître) est certes indéchiffrable pour l'opération, MAIS est compréhensible pour les autres sessions ayant un credential sur le même document maître: elle peut être _transmise_ par les opérations.
- `more` est la seule propriété que les opérations peuvent mettre à jour. C'est dans `more` que se trouvent toutes les informations permettant de régler finement ce que peuvent faire ou non les opérations (et que l'utilisateur ne peut pas fixer de son propre chef).

#### Credentials _désynchronisés_
A la création les copies _Safe Box_ et _document_ sont synchrones. Toutefois,
- la copie _Safe Box_ est écrite avant la copie _document_: si un incident intervient entre ces deux étapes, il existe en _Safe Box_ une copie _fantôme_ et inutilisable.

Depuis une session d'application: 
- la propriété `name` peut être mise à jour dans la _Safe Box_, la copie _document_ n'en n'a cure.
- la propriété `opaque` est mise à jour d'abord dans le _document_ puis une demande de synchronisation du document vers _Safe box_ est exécutée.

Depuis une opération seule la propriété `more` peut être mise à jour et devra, un jour, être synchronisée, avec la _Safe Box_:
- au moment de la mise à jour il se peut qu'aucune session d'application ne soit en exécution. La synchronisation est par principe différée jusqu'à ce qu'une session s'ouvre et le demande.
- dans `more` la propriété `limit` peut aussi être changée: ceci équivaut à une suppression du credential quand la limite est dans le passé.

En conséquence dans une session d'application, un credential en _Safe Box_,
- peut être en retard par rapport à la copie _document_.
- peut exister alors que la copie _document_ a disparu.

> L'utilisateur peut révoquer n'importe lequel de ses credentials, en étant conscients des risques que cela entraîne en termes de pouvoirs de lecture et d'action.

### Les _managers_
Certaines classes de document _maître_ sont indiquées _manager_:
- les utilisateurs ayant un credential pour ces documents sont des _managers_.
- la classe `Org` est une classe prédéfinie singleton _manager_: tout utilisateur ayant un credential sur `Org 1` est un _manager_ (principal).
- d'autres classes **virtuelles singletons**, par exemple `Redaction` peuvent être déclarées comme _manager_. Un utilisateur ayant un credential `Redaction 1` fait partie du _conseil de rédaction_ et a par exemple le pouvoir de déclarer de nouveaux auteurs (voire de les _fermer_).

> Les credentials _manager_ sont exclusivement attribuables par un administrateur: une organisation a demandé à un administrateur technique du service la supportant d'attribuer ces credentials à des utilisateurs ayant des pouvoirs étendus pour configurer et contrôler l'organisation pour un service donné.

Il n'est pas possible pour un utilisateur manager de gérer les autres, d'en inhiber certains et d'en nommer d'autres ce qui permettrait de véritables prises de contrôles. Cette légitimité ne peut se résoudre qu'extérieurement aux services.

> Les managers d'un type donné, par exemple `Redaction`, peuvent consulter la liste des autres managers et engager avec chacun un chat éventuel.

## L'objet `AuthRecord` attaché à toute demande d'opération
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
- une map des  `[docCl docPk]` dont la signature a été vérifiée et validée est établie donne le couple [`credId`, `signature`]. `credId` permet à l'opération,
  - soit d'accéder à l'objet `cred` pointée par `credId` dans la map `creds` du document identifié par `docCl docPk`
  - soit, si la classe de documents n'embarque pas les credentials, d'accéder au document _credential_ correspondant et d'en lire l'objet `cred`,
  - depuis cet objet la clé de vérification permet de vérifier que le _challenge_ a été correctement signé.
  - l'objet `cred` est conservé dans le `AuthRecord` par l'opération et pourra être consulté à volonté par le traitement de l'opération pour décider ce qu'il peut / doit faire en fonction des paramètres inscrits dans `cred`.

> La liste des credentials en échec de vérification est également générée afin de documenter l'exception de rejet de l'opération associée.

> Une opération _peut_ requérir que l'utilisateur soit _authentifié_, voire qu'il soit inscrit comme _administrateur_ du service.

# _Topics_ et _Cases_

Un utilisateur peut avoir besoin,
- soit de signaler à un _utilisateur ayant le pouvoir d'agir_ une situation problématique pour lui dont il souhaite la résolution: par exemple _blocage par insuffisance de quotas d'articles_.
- soit de solliciter un service pour lequel il n'a pas de _credential_ lui permettant d'invoquer directement l'opération correspondante. Par exemple  _disposer d'un compte d'auteur_.

Après avoir ouvert _cas_ pour résoudre son besoin, la résolution / satisfaction de celui-ci aboutit à exécuter une _opération terminale_:
- création de _documents_ comme des _comptes_ soumis à contrôle / autorisation préalable d'une _autorité_.
- obtention de _credentials_,
- upgrade de _credentials_ détenus,
- etc.

Dans le processus de résolution de la situation il intervient,
- un _**utilisateur U**_ demandeur / destinataire de l'action.
- un ou des utilisateurs _**sponsors**_ anonymes pour U ayant le(s) pouvoir(s) de traiter le cas de U.

> **Un _sponsor_ peut aussi prendre l'initiative de créer un _cas_** à destination d'un utilisateur dont il a eu connaissance du _userId_ (typiquement en connaissant un de ses _alias_). Il fait _une proposition_ qu'il pense pouvoir intéresser l'utilisateur, libre à celui-ci de l'accepter ou non.

Les types d'interventions possibles sont identifiés et classés par le service en définissant un `Topic` pour traiter chacun de ces types de problèmes et pour permettre aux _sponsors_ adéquat de se pencher dessus et d'agir.

> Chaque demande correspond à _l'ouverture d'un cas_ d'un _Topic_ donné.

**La liste des Topics est consignée dans une configuration _statique_ du service**: cette liste à 2 niveaux propose pour aider à l'affichage un regroupement des topics par catégories.

Après avoir identifié le `Topic` approprié à son besoin, un utilisateur peut ouvrir un **cas** et y exposer son besoin / problème par un texte écrit sur une _ardoise_ de communication. Selon le `Topic` il peut lui être demandé de fixer _formellement_ par un code le `sujet` précis de sa demande. Par exemple:
- pour se faire créer un compte _Auteur_ le code la catégorie d'auteurs (`Roman Nouvelle Science` etc.),
- pour rejoindre un groupe organisé par _commune_, par exemple un _code postal_,
- pour un groupe de discussion un code _alias_ exact du groupe qu'il souhaite rejoindre et qui lui a été transmis par ailleurs ou a pu rechercher.
- pour un magasin un code PROMO.

La configuration d'un `Topic` indique la composition de sa liste de **sujets**:
- **aucun _sujet_**.
- **liste statiquement prédéfinie** dans le code de l'application / service: ils changent peu et sont peu nombreux.
- **liste définie par un _singleton de configuration du service_** applicable à toutes les organisations: ils peuvent changer assez souvent mais la liste est assez courte pour pouvoir être transférée en session sur demande (le choix s'opérant par filtrage local).
- **liste définie par les valeurs d'une _Property_ de l'organisation**. Comme la précédente mais variant d'une organisation à une autre.
- **liste constituée des valeurs d'une propriété _alias_ d'une _classe_ de documents**, donc **par organisation**: l'existence d'un un code saisi en session est confirmé / infirmé par appel d'une opération recherchant le document de la classe citée dont la propriété alias (qui doit être indexée) correspond au code.

La configuration d'un `Topic` spécifie aussi **quels sont les credentials requis** d'un sponsor pour qu'il puisse traiter les cas qui en relèvent:
- pour un sponsor il suffit qu'il ait un des credentials requis pour pouvoir traiter un cas de ce topic.
- les credentials sont listés par une expression de la forme `"c1 c2 c3 ..."` où les `ci` peuvent être:
- `docCl/1` : les credentials ayant le couple `docCl 1` comme `docCl docId` sont candidats.
- `docCl/S` : les credentials ayant un couple `docCl docId` où `docId` est égal au `subject` du case sont candidats.

> Quand `docCl` correspond à une classe déclarée _manager_ (virtuelle ou `Org`), seul un _administrateur_ est habilité en tant que _sponsor_.

Un utilisateur **sponsor** peut lister les **cas** ouverts sur les topics pour lesquels il a un credential et ne voir que ceux ayant un _sujet_ qui l'intéresse. Il peut choisir un _cas_ l'ouvrir et le traiter.

### Proposition d'un sponsor sur un cas
Si un sponsor dispose d'un au moins des credential requis pour traiter un cas, il va définir les paramètres de sa proposition dans la propriété `etc` du cas qui sera interprétée par le traitement final:
- c'est donc un sponsor qui fixe comment le cas sera traité effectivement et sous quelles restrictions / possibilités réelles.
- en fonction des paramètres contenus dans la propriété `more` du credential présenté par le sponsor, ce qui est fixé dans `etc` peut varier, et à la limite ne permet pas au sponsor de satisfaire l'objet de la demande si elle excède ses propres pouvoirs.

### Traitement final d'un _cas_
L'objectif de l'ouverture d'un cas n'est en général pas cantonné à avoir des échanges textuels par l'ardoise entre un utilisateur et un ou des _sponsors_, mais a souvent pour but **d'aboutir à un traitement final**:
- la phase d'échange a permis à un _sponsor_ de définir les paramètres d'une _solution_ dans la propriété `etc` du cas.
- in fine, c'est l'utilisateur qui **valide** (ou non) le déclenchement du traitement final qui va s'exécuter selon les conditions fixées par le _sponsor_: _un ou des comptes seront créés, des credentials aussi,_ etc.

**Un _cas_ vit peu de temps:** quand il est _annulé_ ou _finalisé_ par son utilisateur, il devient passif puis s'auto-détruit quelque jours plus tard.

## Classe `Topic`
La classe de documents `Topic` est _virtuelle_, aucun document n'est stocké en DB pour représenter un topic.
- un singleton `topics` énumère en JSON les topics déclarés.
- il peut être mis à jour par un administrateur technique.
- il est rechargé dans le service quand la version détenue en cache est trop ancienne.

    [
      { id: topic1, categ: c1, keys: k12, subjects: "...", creds: "c1 c2 c3 ..." },
      ...
    ]

- `topic1` : ID du topic.
- `categ`: code catégorie.
- `keys`: des couples de clés _Décryptage / Cryptage_ sont enregistrés dans la configuration du service sous un _code_ à donner dans `keys`.
- `subjects`:
  - absent: le topic n'a pas de sujets.
  - `"a b c "`. Valeurs des codes séparées par un espace. Un libellé traduit chacun dans la langue choisie en session.
  - `"@sujet35"` : ID du _singleton_ (du service) portant la liste des codes.
  - `"$sujet35"` : ID du _Property_ (de l'organisation) portant la liste des codes.
  - `"DocCl/alias"` : nom de classe `DocCl` des documents dont `alias` est un propriété dont les valeurs constituent la liste des codes valides.
- `creds`: credentials requis listés par une expression de la forme `"c1 c2 c3 ..."` où les `ci` peuvent être:
- `docCl/1` : les credentials ayant le couple `docCl 1` comme `docCl docId` sont candidats.
- `docCl/S` : les credentials ayant un couple `docCl docId` où `docId` est égal au `subject` du case sont candidats.

En cours de session, les applications demandent aux services qu'elles gèrent la configuration des topics:
- la propriété `pubc` la clé de cryptage publique correspondant au code dans la configuration du service est transmise (mais pas `privd`). 
- le cas échéant:
  - le singleton définissant la liste de valeurs,
  - la vérification d'existence d'un _alias_.

Dans une session pour récupérer tous les _cases_ a priori traitables le process suivant est engagé:
- récupération de tous les credentials de l'utilisateur pour obtenir l'ensemble de leurs couples `[ docCl/docId, ...` où `docId` peut être `1` ou une autre valeur `SSS...` qui sera comparée avec la valeur du _subject_ de chaque case.
- la requête de l'opération de collecte récupère tous les cases dont le `docCl/docId` matche avec au moins un des termes de la liste élaborée en session.

### Credential d'un topic - ??? TODO à vérifier
Topic étant une classe _virtuelle_, les credentials associés sont des documents de class `Credential`:
- `docCl`: `Topic`
- `docId`: le `topicId` du topic.
- `cred`: { ..., more: { subjects: [] } }
  - `subjects`: si présente cette liste contient un ou plusieurs `subject` restreignant la portée du credential.

## Les _cases_
Un _case_ est un document de classe `Case` identifié par `caseId`:
- `caseId` : ID universel généré aléatoirement à la création.
- `v` : version du document. Elle détermine aussi la limite de validité du document.
- `userId`: ID de l'utilisateur détenteur du cas. Depuis une opération du service la clé publique de cryptage `CU` est donc accessible.
- `topicId` : ID du topic auquel le cas se rapporte. Sa clé de cryptage `CT` est publique et sa clé privée de décryptage `DT` n'est disponible que dans une opération.
- `subject` : code (facultatif) désignant une cible plus précise permettant à un utilisateur _sponsor_ de se concentrer sur un sujet précis. 
- `status`: 0-annulé 1-actif-U 2-actif-H 3-finalisé.
- `tabT`: texte de l'ardoise crypté par la clé T/U.
- `etc`: objet qui ne peut être écrit configuré que par une opération d'un _sponsor_ autorisé.

`caseId topicId userId subject` : ces propriétés sont immuables.

**Clé primaire et index:**
- `pk`: `caseId`
- `topic`: `topicId`
- `topicsub`: `topicId subject`

### Clé T/U pour crypter l'ardoise
Le cryptage par un _sponsor_ est effectué par:
- la clé _privée D_ du topic,
- la clé _publique C_ de U.
- quand un sponsor est le premier à écrire (proposition), il connaît néanmoins U puisqu'il s'adresse nommément à lui.

Le cryptage par U est effectué par:
- la clé _privée D_ de U,
- la clé _publique C_ de T disponible publiquement.

**Une session de U** est en conséquence capable de crypter / décrypter `tabT`, aussi bien à la création qu'à la mise à jour et en lecture.

**Une session d'un _sponsor_** en revanche ne dispose pas _naturellement_ de la clé **privée D** du topic du case:
- les opérations savent déterminer si un _sponsor_ est habilité à être _sponsor_.
- les opérations de création / mise à jour pour un _sponsor_ reçoivent en conséquence `tab` en clair et, disposant de la clé _privée D_ du topic et de la clé _publique C_ de U, crypter `tab` en `tabT`.

En conséquence:
- quand une opération de création / mise à jour reçoit `tab`:
  - si l'appelant est U `tab` est déjà cryptée en `tabT`.
  - sinon `tab` est en clair et c'est l'opération qui crypte `tab` en `tabT`.
- quand l'opération de lecture reçoit une demande _getCase_:
  - si l'appelant est U, `tabT` est retourné et sera décrypté par la session de U.
  - sinon l'opération _décrypte_ `tabT` en `tab` et le retourne.

### Status d'un _case_
- 1 : _actif écrit par U_. Il peut être mise à jour et peut subir un traitement final (si `etc` l'autorise). C'est U qui l'a écrit en dernier.
- 2 : _actif écrit par H_. Il peut être mise à jour et peut subir un traitement final (si `etc` l'autorise). C'est une opération du service sollicitée par un _sponsor_ H qui l'a écrit en dernier.
- 3 : _finalisé_. Son traitement final a eu lieu, le cas est en lecture seule pour information jusqu'à expiration de son délai de fin de vie.
- 0 : _annulé_. Son traitement final N'A PAS eu lieu, le cas a été annulé par U et est en lecture seule pour information jusqu'à expiration de son délai de fin de vie.

### La table `ZZCASES` du Master Directory
Cette table partagée par tous les utilisateurs et services, sert à un utilisateur à être informé,
- de l'existence des cases existants entre lui et des _sponsors_, 
- soit de l'inscription d'un nouveau _case_ proposé par un _sponsor_,
- soit d'une nouvelle version d'un _case_ qu'il n'avait pas encore lue.

#### Propriétés
- `caseId` : ID universel généré aléatoirement à la création (clé primaire).
- `userId`: utilisateur de l'ardoise. Index de sélection (index).
- `v` : version du document dans la DB du service. Elle détermine aussi la limite de validité du cas.
- `data`:
  - `chk`: SHA raccourci des données immuables `userId topicId subject svc org`. Permet de vérifier que la demande vient bien d'un détenteur légitime (session ou opération).
  - `svc org` : service détenteur de l'ardoise.
  - `topicId` : topic du _cas_.
  - `subject`: sujet du cas si requis.
  - `status`: 0 1 2 3
  - `aboutU`: texte crypté de commentaire pour le seul usage de l'utilisateur.
  - `lv` : dernière version _lue_ par U. La comparaison avec `v` permet de savoir si U a eu connaissance de la dernière évolution produite par le service.

#### Opérations sur `ZZCASES`
- `mdCaseNew`: _preset_ de création du case avec ses propriétés immuables.
  - arguments: `caseId userId topicId subject svc org`
- `mdCaseSync`: synchronise les propriétés variables `v status` avec les valeurs du _document_.
  - arguments: `caseId chk`
- `mdCaseUser`: fixe les propriétés variables `lv aboutU` avec les valeurs fixées par l'utilisateur.
  - arguments: `caseId chk lv aboutU`
- `mdCaseDel`: suppression d'un case
  - arguments: `caseId chk`
- `mdCasePurge`: suppressions des cases dont `v` est inférieure à `limit`.
  - arguments: `limit`

> Une session ou une opération _légitime_ connaît les propriétés immuables: au delà de la _pré-création_ la fourniture de `chk` sert à vérifier cette légitimité.

### Cycle de vie d'un _case_ créé par un _sponsor_
Un _sponsor_ prend l'initiative de créer une _proposition_ en lançant une opération créant un _case_:
- elle dispose du `userId` de l'utilisateur ciblé, typiquement pour l'avoir obtenu depuis un de ses _alias_ publics. Elle connaît donc aussi la clé publique `CU` de cryptage de U.
- elle dispose d'un _credential_ `docCl docPk` qui matche un des credentials `ci` autorisé du topic: de forme `docCl/1` (`docPk` vaut `1`) ou `docCl/S` où `S` est le sujet du cas et égal à `docPk`.
- la configuration des topics en cache du service détient la propriété `topicDT` de _décryptage_ privée du topic.
  - elle calcule la clé `X` depuis `[topicDT, CU]`.
- elle créé le _document_ `Case`:
  - génère `caseId` depuis la date/heure (_epoch_) courante.
  - crypte le texte de l'ardoise tab par `X`.
  - `status` est 2.
  - `etc` est rempli depuis les arguments de l'opération de création du cas (données que U peut lire mais pas écrire).
- elle créé un row dans `ZZCASES` par invocation d'une opération `mdCaseNew` du Master Directory. 

Quand l'utilisateur U lira à l'ouverture de sa prochaine session (ou sur demande explicite) la table `ZZCASES` du Master Directory pour obtenir tous les cas modifiés / créés depuis sa dernière lecture, sa session obtiendra ce _nouveau_ cas en lisant le document depuis `svc org topicId caseId`.
- la session calcule `X` depuis `[DU, CT]`: 
  - `CT` figure dans la session qui a chargé la configuration (publique) des topics du service. 
  - `DU` est détenue par la session.

U peut activer l'opération `mdCaseUser` du Master Directory pour faire noter dans `ZZCASES` avoir lu cette nouvelle version (positionnant `lv` à `v`) et fixer le cas échéant une mise à jour de `aboutU`.

##### Mise à jour du _case_ par l'utilisateur
Après lecture en session du document _case_, des opérations sont possibles afin:
- de mettre à jour `tabX` crypté par `X`.
- communiquer `aboutU` (éventuellement) pour mise à jour par l'opération `CaseSet`.
  - mise à jour de `v` et `status`: cas _d'annulation_ et de _finalisation_.

C'est l'opération du service qui met à jour `ZZCASES` par l'opération `mdCaseUser`.

# _Chats_ entre utilisateurs disposant d'un credential sur un même document maître
Ce dispositif est autorisé ou non par classe de documents maître.

Cette classe peut être _virtuelle_.

Les utilisateurs détenteurs d'un credential d'un document maître `docCl docPk` forme de facto un _groupe_ dont les membres peuvent se connaître les uns les autres:
- par les données `opaque` que chacun a dans son credential et qui peut être décryptée par les autres qui disposent de la même clé `docKey`,
- par les autres propriétés dépendantes de la classe.

> Ces utilisateurs ont donc une vision _explicite_ des autres: _nom, carte de visite avec photo, autres propriétés libres, etc_

Chaque credential disposant d'une clé publique de cryptage, il peut s'établir des _chats_ entre deux membres de ce groupe.

Soit deux credentials A et B ayant un chat entre eux. Tout item de chat écrit par A est dédoublé:
- une copie cryptée par A avec la clé de cryptage de B et stockée dans le credential B.
- une copie cryptée par A avec sa propre clé publique de cryptage et stockée dans le credential de A.

> Un item peut être _multi-destinataire_: le même item est envoyé N fois (avec des CC), l'expéditeur n'en ayant qu'une copie (et non N).

### Quelques règles:
- un item peut être marqué _important_ par son destinataire.
- le nombre d'items de chat par credential est limité.
- le volume total des items par credential est aussi limité.
- quand le volume est excessif, les plus anciens disparaissent en essayant de conserver ceux _importants_.
- **A dispose d'une liste noire**. Si B est en liste noire de A,
  - il n'a droit qu'à un item chez A (les précédents s'effacent),
  - cet item est limité en taille (50 signes),
  - il est marqué _liste noire_ et est prioritaire à l'effacement en cas d'excès de volume.
- **A peut à l'inverse exprimer une liste blanche**, tous ceux non cités sont en liste noire.
