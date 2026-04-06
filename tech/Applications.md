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
- **une partie conservée dans la base de donnéesdu service pour l'organisation souhaitée** qui permet à l'opération de _vérifier_ que le jeton reçu en paramètre de l'opération est effectivement valide et contient bien les données qu'il prétend détenir.

Ce mécanisme détaillé par ailleurs permet,
- de ne pas stocker dans la base de données les éléments de _signature_,
- de pouvoir refuser des _jetons usurpés_, c'est à dire ayant déjà été présentés une fois et représentés plus tard.

### _Coffre fort_ / _safe_ d'un utilisateur
Chaque _droit d'accès_ est un texte long, comportant des textes d'apparence aléatoire, bref impossibles à mémoriser (et à inventer par _force brute_). 

L'utilisateur pourrait certes disposer d'un fichier personnel où il les rangerait mais la sécurité et l'accès depuis plusieurs terminaux à ce fichier exposerait ces données de sécurité _critiques_ aux pertes et aux vols.

Chaque utilisateur dispose à cet effet d'un _coffre-fort_ personnel où ses droits d'accès sont rangés, cryptés et sécurisés. 

Le _coffre-fort_ d'un utilisateur a pour identifiant celui de l'utilisateur (ou à  l'inverse un utilisateur est identifié par le numéro de son coffre). Il comporte plusieurs _rubriques_:
- son **entête** qui détient les éléments cryptographiques techniques nécessaires à son fonctionnement.
- la **liste de ses droits d'accès**.
- une **liste de terminaux de confiance**, c'est à dire des terminaux d'où il pourra s'identifier par un code PIN plus simple que son identification _forte_ et sur lesquels chaque application pourra laisser des _documents en mémoire cache_ locale cryptée permettant un usage en _mode avion_.
- une **liste de préférences** de comportement et d'affichage de son choix afin de retrouver en lançant une nouvelle session, l'organisation de l'écran qu'il souhaite, les options de son choix, sa langue de travail, etc.

#### Dépôts des coffres-forts : _standard_  ou _personnels_
Un **dépôt _standard_** est géré: tout utilisateur peut y disposer de son _coffre-fort_.

Mais certains utilisateurs sont prudents / paranoïaques et peuvent ne pas vouloir que leurs droits d'accès soient conservés par une entité qu'ils ne maîtrisent pas ou qui pourrait être défaillante ... 

Chaque utilisateur (ou groupes d'utilisateurs) peut installer son propre dépôt de _coffres-forts_ dans une base de données MySQL d'un site Web de son choix (et sous son entière responsabilité d'administration) muni d'un script PHP standard mais dont il peut lire le texte et s'assurer de sa non nocivité. 

Des moyens sont données pour basculer du dépôt _standard_ vers un _dépôt spécifique_ (et réciproquement), ainsi que pour effectuer des _backup_: l'image d'un _coffre-fort_ peut être exportée crypté par une clé détenue par le seul utilisateur.

> Le _contenu_ d'un coffre-fort est lisible _en clair_ **pour son propriétaire et seulement lui**, ... mais étant plein de données cryptographiques le terme _en clair_ est un peu une vue de l'esprit.

### Exécution d'une application en _mode AVION_
Quand un utilisateur a déclaré un ou des terminaux **de confiance** (les a _certifiés_) quand il y lance une session d'une application celle-ci peut utiliser une **mémoire cache de documents et fichiers**, cryptée et sécurisée sur le terminal.

Depuis ce même terminal, l'utilisateur peut rouvrir une session qui s'est antérieurement exécutée sur ce terminal, elle y a été _épinglée_:
- s'il a accès au réseau Internet, le lancement sera rapide du fait que beaucoup de documents n'auront pas à être redemandés aux services, étant déjà _en cache_ (cryptée) dans le terminal.
- s'il n'a pas accès au réseau Internet il peut rouvrir son application en **mode AVION** et accéder (en lecture seulement) aux documents disponibles en cache du terminal du fait d'une exécution antérieure.

### Synthèse
Les **applications** s'exécutent sur le terminal de l'utilisateur où elles ont été chargées depuis leur URL (ou un magasin d'applications).

Elles font appel à des **services de traitement des données** distants, chacun ayant un jeu **d'opérations** pouvant lire / écrire SA **base de données** (éventuellement SES bases en cas de volume excessif) et SON **storage de fichiers** (éventuellement SES) partitionnés par **organisation**.

Chaque requête à une opération est dédiée à UNE organisation et n'accède qu'à la partition de la base de données dédiée à cette organisation et / ou du storage dédié à cette organisation.

Tout **utilisateur** dispose d'un **coffre-fort** détenant en particulier ses _droits d'accès_ requis à l'appel des opérations des services par une application. Un utilisateur peut décider de confier la gestion de SON coffre-fort, soit au **dépôt standard**, soit à un **dépôt spécifique** géré par le site Web de son choix.

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

> Une base de données MASTERDIR détient une petite table `SAFEURLS` ayant une ligne par _service_ qui donne la liste des opérateurs proposant ce service et son URL.

Une **application déployée** dispose dans sa configuration de l'URL d'accès à MASTERDIR ce qui lui permet d'obtenir pour le service `randos` par exemple les URLS pour les opérateurs **Rouge** ou **Bleu**.

Un utilisateur _terminal_ a les moyens techniques de vérifier que l'application déployée qu'il entend utiliser,
- correspond bien au logiciel _officiel_ (et non pirate) mis en ligne en _source_ par son _éditeur_: sa version a pu être certifiée par une autorité de sécurité indépendante.
- accède bien aux _opérateurs officiels_ prévus et non à des sites pirates.

> Il est possible d'accorder sa confiance à une application déployée d'un éditeur la rendant accessible en _open source_ parce qu'il est possible à une entité de certification externe à l'éditeur de vérifier la conformité de ses déploiements.

## Une application terminale peut accéder à plus d'une organisation
> La table `SAFEORGS` du MASTERDIR dispose d'une ligne par _organisation_ donnant _pour chaque service_ le code de l'opérateur choisi par l'organisation.

Dans le cas de l'application `randos`, un utilisateur peut être membre de plus d'une association de randonneurs: une pour ses randonnées près de chez lui, une autre pour les randonnées de montagne et une troisième pour les treks lointains. Depuis la même application il peut basculer d'une organisation à une autre.

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
Une application donnée sur un appareil donné est identifiée par un _jeton_ qui une sorte de numéro de téléphone universel: tout service ayant connaissance de ce jeton peut envoyer des _notifications_ à l'application correspondante sur le terminal correspondant.

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
Les applications **_sourdes_** classiques ne peuvent afficher des écrans que sur sollicitation de l'utilisateur. 

L'écran ne se remet à jour que suite à une action de l'utilisateur: si ce dernier ne fait rien, l'écran ne change pas et affiche des données plus ou moins anciennes qui ont pu être déjà modifiées par l'action d'autres utilisateurs, du temps qui passe, etc.

Les applications **_écoutantes_** peuvent remettre à jour leurs écrans et données détenues localement même sans action d'un utilisateur simplement en fonction des _notifications_ poussées vers elles par les serveurs. Elles _peuvent_ rester à l'écoute (même non lancée) et l'utilisateur peut, à réception d'une notification, rouvrir l'application d'un clic.

# Les stockages des données

Pour un _service donné_ assuré par un opérateur donné il existe deux stockages dédiés:
- une base de données,
- un _storage_ de fichiers.

Les stockages sont _partitionnés_ par _organisation_, une partition pour chaque organisation hébergée par ce service.

### La base de données
Elle gère les documents selon un mode _transactionnel_ (ACID).

Elle gère aussi les _abonnements_ des applications terminales aux _documents (synchronisables)_ qui les intéressent: chaque application sur un appareil a un _token_ qui l'identifie de manière unique. 

> Sur un terminal _certifié de confiance_ par un utilisateur, une _micro base de données locale_ pour chaque session d'application _épinglée_ détient en _cache_ les _documents_ récemment demandés et les _abonnements_ en cours de l'application. 

Quand un ou des documents évoluent par exécution d'une opération, celle-ci retrouve toutes les applications terminales abonnées et effectue une publication de notifications vers elles.

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
- `value`: JSON des deux clés publiques de cryptage et vérification (avec les bannières ---BEGIN ...).

    [ "---BEGIN PUB.. ---", "---BEGIN PUB.. ---" ]

#### Table `SAFE`
Pour les utilisateurs dont le safe est hébergé dans le MASTERDIR (générique).

C'est un document avec les colonnes: `id hp0 hr0 hct lam data`

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
- certifié par vérification de la signature du _challenge_ par usage de la clé publique de vérification de cet utilisateur obtenu de la table `SAFEPEMS`,
- par présence du `userId` dans,
  - la liste `ADMINUSERS` du déploiement d'UN service SVC / $OP.
  - la liste `MASTERDIRADMINUSERS` pour l'Administrateur du MASTERDIR.

### Liste des rôles ADMIN s'un utilisateur
Pour pouvoir afficher la page _Administration Technique_, un utilisateur doit auto-déclarer dans son _safe_ la liste des couples `SVC $OP` pour lesquels il a ce pouvoir:
- quand il en ajoute un, le fait qu'il le soit réellement est vérifié.
- si son `userId` a été retiré de la configuration, il doit remettre à jour cette liste.
- sil ne s'inscrit pas de lui-même de facto il ne peut pas atteindre la page d'administration.

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

## Credentials
UN _credential_ gère UN droit pour,
- UN utilisateur `userId`,
- accéder aux opérations d'UN service `SVC`
- accédant à UNE organisation `org`.

La cible du credential est identifiée par le couple `role/docId`:
- `role` : UN rôle est matérialisé par le couple `classe.role`,
  - d'UNE classe de document (par exemple `Redacteur.`),
  - et éventuellement d'un rôle complémentaire `Redacteur.chef` (quand l'entité correspondante peut être abordée selon plusieurs rôles).
- `docId` est l'identifiant du document correspondant.
  - quand `docId` est vide, le credential a pour cible TOUS les documents de la classe. C'est typiquement le cas pour `Org.` n'ayant qu'un document _son ID 1_ n'est pas donnée.

> L'ID d'un _credential_ est le hash court de `userId/SVC/org/role/docId`. Pour une ID donnée **plusieurs versions successives** peuvent SUCCESSIVEMENT, chacune étant repéré par son `time` la date-heure en secondes de sa déclaration.

### Les _faces_ "safe" et "document" d'un credential
**La face _safe_ est enregistrée dans le _Safe_ de l'utilisateur**, avec:
- la **clé privée de signature** de cette version du credential (une version _ultérieure_ aura une nouvelle de signature),
- quelques propriétés détaillées ci-après.
- au détail près du _commentaire_ facultatif donné par l'utilisateur lui-même, cet objet est immuable.

**La face _document_ est un document enregistré dans la base de données du service**, avec:
- la **clé publique de vérification** de cette version du credential.
- quelques propriétés détaillées ci-après qui peuvent évoluer au cours du temps.

#### Dans une session de l'application
- toutes les faces _safe_ sont lues depuis le _safe_ de l'utilisateur pour les credentials relatifs à un des services accédés par l'application.
- les _documents_ credential de tous les services accédés par l'application relatifs au `userId` de l'utilisateur sont lisibles / synchronisables.
- une application dispose des deux faces de chaque version de credential, mais:
  - ne peut mettre à jour QUE la propriété `comment` de sa face application (celle stockée dans son _Safe_).
  - ne peut QUE lire les _documents_ credential correspondant.

#### Dans une opération d'un service
- tous les _documents_ credential citées par l'objet `AuthRecord` attaché à la requête sont lisibles.
- les faces _safe_ ne sont pas accessibles.

### Cycle de vie d'un credential
Pour simplifier on se fixe sur UN credential (pour un service, une organisation et un utilisateur) identifié par `role / docId`.
- sa création est faite par **inscription conjointe** dans le safe de l'utilisateur et le document dans la base du service: le couple de clés de signature (safe) et de vérification (BD du service) est cohérent. Sa date-heure `time` (de création) est fixée.
- des opérations peuvent faire évoluer le document en DB:
  - modification des conditions d'application de la propriété cond, offrant plus ou moins de _pouvoirs_ au détenteur du credential.
  - inscription / effacement de la date-heure `limit` invalidant or revalidant l'usage du credential **dans le futur**.
- une **suppression**,
 - soit **conjointe** dans le safe et la DB du service demandée par l'utilisateur (qui efface les deux),
 - soit par fixation par une opération de `limit` au jour J (ou dans le passé) correspondant à une suppression (logique puis à terme physique) du _document_.

> Au début d'une opération, un jeton émis par la session est vérifié, la signature du challenge  par la clé de signature _safe_ est vérifiée par la clé de vérification détenue la DB du service. Mais le credential peut être marqué hors limite: la session n'en n'a pas été informée. 

#### Credentials _brisés_
Un credential est _brisé_ quand,
- il est connu côté _safe_ et inconnu côté _service_: typiquement par une action en deux phases safe / service, la seconde ayant techniquement échoué.
- il a une limite inférieure au jour J du côté _service_: le _safe_ n'en n'a pas été informé.

### En réflexion: synchronisation des credentials _service_
Dans cette optique, les documents _Credential_ sont synchronisés en début (et en cours) de session par abonnement:
- la mise à jour du credential _safe_ associée se limiterait à sa suppression du fait de suppression du _document_ (`limit` en deçà du jour courant).
- le credential _safe_ ne sert qu'à,
  - porter la clé de signature,
  - porter un commentaire fixé par l'utilisateur.

### Création d'une version d'un credential
Elle peut être effectuée selon deux modes:
- **attribution directe** à un utilisateur U par un utilisateur A _attributaire_.
- **auto-attribution après invitation** par l'utilisateur U suite à une **invitation** générée par un utilisateur SP _sponsor_.

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

#### Process d'invitation de U par un _sponsor_ SP
Ce process est décrit en détail dans le chapitre _Invitations_. Il a plusieurs phases:
- **dépôt d'une demande motivée par U:**
  - l'invitation est _déposéee_.
  - dans cet état U peut avoir un remord et l'annuler: elle sera en état _annulée_.
- **acceptation par un sponsor SP:**
  - le sponsor a attaché quelques conditions à la demande qui passe en état _acceptée_.
  - le sponsor peut aussi rejeter la demande avec un court texte explicatif, la demande passe en état _rejeté_.
- **validation par l'utilisateur U**. Au vu des conditions inscrites par le sponsor ayant répondu, U peut:
  - valider l'invitation qui passe en état _validée_.
  - décliner l'invitation qui passe en état _déclinée_.

### Propriétés de l'objet face _safe_
Parmi les propriétés _communes_ `ID userId SVC org role docId time`,
la propriété `userId` n'est pas stockée, puisque le _safe_ est dédié à ce `userId`.

Les autres propriétés sont:
- `pems`: la clé privée de signature générée pour cette version du credential.
- `comment`: un commentaire libre et facultatif de l'utilisateur qui peut l'aider quand il constitue un profil de session ne reprenant que _certains_ des credentials.
- `name`: le `docId` cible est un code ininterprétable humainement. Toutefois au moment de la création, le créateur peut connaître un _nom / libelle /etc._ explicitement lisible par un humain. Ceci peut aussi aider U quand il constitue un profil de session ne reprenant que _certains_ des credentials.
- `skey` : une clé symétrique AES (facultative). Lire ci-après.

DPour certaines classes de documents, certaines propriétés sont _lisibles_ par le service, d'autre peuvent être _opaques_ pour le service.
- **propriétés lisibles**: c'est l'état normal et le logiciel du service peut les utiliser dans ses traitements.
- **propriétés opaques**: elles sont _cryptées_ par une clé qui n'est disponible QUE dans les applications terminales et sont en conséquences inutilisables par une opération du service.

Dans certains cas des données peuvent être considérées comme _confidentielles_, de lisibilité restreinte, par exemple à un _comité directeur de .._, à un _agent_ pour ses données personnelles, etc. Le principe est que la _clé AES_ qui a rendu ces données _opaques_ aux opérations du service, n'est PAS stockée dans la base de données du service: même en cas de piratage de celle-ci elles restent inviolées, illisibles.

Chaque utilisateur ayant un droit d'accès à ce _comité directeur_ par exemple disposera de cette clé dans la propriété `skey` du credential correspondant et pourra ainsi décrypter ces données _opaques_ pour les opérations du service.

`skey` a été transmise:
- dans le cas d'une attribution directe par A: A en est détenteur lui-même.
- dans le cas d'une invitation, le sponsor doit être lui-même détenteur de cette clé, et peut l'inscrire dans l'invitation (cryptée par la clé SP / U) lisible que par U.

> Il est toutefois possible que certaines clés d'opacité soient présentes dans un _document_ et non pas uniquement dans les _safes_: elles se trouvent alors elles-mêmes cryptées dans des propriétés _opaques_. Une clé _maîtresse_ `skey` peut servir à _opacifier_ un jeu peut-être important de clés _secondaires_ accessibles de facto dès qu'un utilisateur détient la `skey` _maîtresse_.

### Propriétés du _document_ stocké en base du service
Parmi les propriétés _communes_ `ID userId SVC org role docId time`,
les propriétés `SVC org` ne sont pas stockées explicitement _dans_ le document (`org` est stockée à part et tous les documents de la base de données sont spécifiques du service `SVC`).

Les autres propriétés sont:
- `pemv`: la clé publique de vérification générée pour cette version du credential.
- `limit`: date-heure en secondes de la limite de validité de cette version. Si absente la validité est éternelle. Elle peut être mise à jour par une opération de prolongation / révocation.
- `cond`: c'est un objet dépendant du rôle du credential contenant les données explicitant les conditions d'exercice du credential: _seuils divers, liste de permissions, etc._

## L'objet `AuthRecord`
Toute opération requérant la présence d'au moins un credential est sollicitée en passant en arguments un objet de classe `AuthRecord`, construit par l'application et ayant les propriétés suivantes:
- `userId`: de l'utilisateur.
- `sessionId`: identifiant de session.
- `time`: date-heure en seconde de création du record.
- _challenge_: propriété virtuelle _userId + '/' + time_
- `userSign`: signature par la clé privée de signature de l'utilisateur, du _challenge_.
- `signatures`: objet ayant une propriété par ID de credential inscrit dans le record donnant la signature du challenge par la clé privée de signature du credential.

Au démarrage d'une opération, le `AuthRecord` joint est scanné:
- si la signature du `userId` n'est pas validé, c'est un échec.
- une map des  `docClass.role/docId` dont la signature a été vérifiée et validée est établie et pointe sur le document _credential_ correspondant.
- la liste de ceux en échec de vérification est également générée afin de documenter l'exception de rejet de l'opération associée.

Dans le cours du traitement de l'opération, cette map est consultable par la logique de l'application pour déterminer si l'opération peut ou non être acceptable en fonction de ses propres paramètres, de l'état des documents et des données issues du `cond` attaché au credential validé.

# Invitations

Un utilisateur est (sauf à la limite pour les Administrateurs Techniques) partie prenante d'une ou plusieurs _positions / rôles_: _auteur, relecteur, employé, dirigeant d'un département, gestionnaire d'un stock ..._ chaque position / rôle est matérialisé par l'existence d'un _document_ identifié sur lequel l'utilisateur a un _credential_ qui lui confère des possibilités d'actions plus ou moins large, voire la connaissance d'une clé de cryptage symétrique attachée à ce rôle.

La création double d'un document **et** du credential associé s'effectue par le mécanisme des invitations qui met en jour deux intervenants:
- l'utilisateur U qui sollicite cette création et le credential associé,
- un utilisateur SP _sponsor_ qui va lui donner satisfaction (ou non).

### Protocole général
- **(1) l'utilisateur U créé une _demande d'invitation_** qui exprime ses attentes: pourquoi il souhaite voir accepter son invitation avec quelles caractéristiques ...
  - l'invitation est un document déposé dans la base de données du service correspondant pour l'organisation souhaitée. Son status est **déposée**.
  - le _safe_ de U en garde un _pointeur_ lui permettant de la lire.
- **(2) un utilisateur SP _sponsor_ à l'écoute des nouvelles invitations déposées, traite celle-ci:** c'est le traitement d'acceptation.
  - ceci passe la status de l'invitation à **acceptée**, lequel status est répercuté dans le _safe_ de U.
- **(3) l'utilisateur U voit, à sa prochaine connexion ou sur _rafraîchissement_ en cours de session que son invitation est acceptée et il l'ouvre:**
  - il y découvre le détail de la proposition du sponsor: en général il en est satisfait et la _valide_.

#### Traitement _d'acceptation_
Le sponsor définit, depuis un formulaire spécifique du type d'invitation, les conditions d'acceptation de la demande qui s'appliqueront à sa validation finale:
- si oui ou non des documemts seront à créer, avec quels ID, etc.
- quels _credentials_ seront à enregistrer à cette occasion: les données afférantes sont stockées dans la demande d'invitation par le service et un résumé textuel en est généré afin que U puisse en comprendre les termes.

#### Traitement de _validation_
La _validation_ correspond à un traitement réparti entre application et service:
- (1) l'application génère les couples de clés signature / vérification qui ont été requises par le sponsor pour les credentials à enregistrer (s'il y en a). Seules les clés publiques de vérification sont transmises à l'opération de validation par le service.
- (2) l'opération de validation par le service:
  - créé / enregistre les documents prévus et stockés dans l'invitation dans sa base,
  - enregistre les _credentials_ prévus en y adjoignant les clés publiques de vérification reçues de l'application,
- (3) l'application finalise la validation:
  - enregistre dans son _safe_ les credentials éventuellement créés avec les clés privées de signature qui viennent d'être générées à l'étape (1).
  - effectue un traitement éventuel vis à vis des documents créés (abonnement de synchronisation, affichage, etc.): il a les droits pour y accéder, ceux-ci ayant été enregistrés par le service à l'étape (2).

#### Le sponsor qui prend en charge une invitation peut ne pas donner suite
- l'invitation passe en status **rejetée** avec un motif textuel explicatif, le _safe_ de U étant informé de ce changement de status et du motif. Son _pointeur_ dans le _safe_ de U est auto-destructible après un délai court.
- dans le service le document _invitation_ n'a plus que quelques jours à vivre (et ne peut plus changer).

#### L'utilisateur U peut ne pas être satisfait des conditions de l'acceptation
- l'invitation passe en status **déclinée** avec un motif textuel explicatif pour le sponsor, le _safe_ de U est mis à jour de ce changement de status. Son _pointeur_ dans le _safe_ est auto-destructible après un délai court.
- dans le service le document _invitation_ n'a plus que quelques jours à vivre (et ne peut plus changer).

> Dans ce mode par invitation U n'a inscrit que des droits qu'il a sollicités et approuvés. En contrepartie, ça l'oblige à a) solliciter une invitation, b) en attendre l'acceptation par un sponsor, c) la valider. En mode attribution directe l'attributaire pouvait inscrire directement le droit de U.

Une `Invitation` est un document,
- créé par un utilisateur U par une demande d'invitation à la date-heure `time`,
- complété par un sponsor SP lors de son acceptation,
- et qui s'auto-détruit quelques jours après time (quelque soit son état).

Une invitation est stockée deux fois:
- _un pointeur réduit_: dans le _safe_ de l'utilisateur demandeur U cet enregistrement permet l'utilisateur d'accéder au document _complet_.
- _complète_: dans la base de données du service avec des propriétés spécifiques.

### _Pointeur_ vers l'invitation en _safe_
Cet objet a les propriétés suivantes:
- `svc`: service concerné.
- `org`: organisation concernée.
- _userId_: implicite (c'est le _safe_ de U qui le stocke)
- `invitId`: ID aléatoire générée à la création.
- `time`: date-heure (_epoch_ en secondes) de création.
- `major`: code _majeur_ de l'objet de l'invitation.
- `minor`: code _mineur_ (facultatif) complémentation de l'objet de l'invitation.
- `status`: _déposée, acceptée, refusée, validée, déclinée, annulée_.
- `comment`: commentaire pour le seul usage de l'utilisateur U ayant fait la demande.

Cet objet est **invariant** après création, SAUF son _status_ qui évolue au cours de sa courte vie.

### Document `Invitation` en base de données du service
Il a les propriétés suivantes:
- _svc_ : implicite (il estdans la base de données du service).
- _org_: (implicite: chaque document est toujours attaché à son organisation).
- `userId`: string = '' // ID de U (demandeur)
- `invitId`
- `major` 
- `minor`
- `time`
- `status`

Propriétés fixées par U lors du dépôt de la demande d'invitation (ou quand elle est déclinée):
- `safeStore`: URL du store hébergeant le safe de U s'il est spécifique. Permet à l'opération acceptation/rejet du sponsor de faire évoluer le status de l'invitation.
- `skeyK`: clé symétrique requise ou non selon le `major`:
  - soit générée par U, cryptée par sa clé K (donc _opaque_ aux opérations du service).
  - soit délivrée par le sponsor cryptée par la clé SP / U, récupérée depuis un document quelconque lors de l'opération d'acceptation.
- `pemU`: clé publique C de U, nécessaire au sponsor pour crypter ses textes à destination de U.
- `txtm`: texte (en clair) de motivation de la demande d'invitation écrite par U. 
- `txtx`: quand déclinée, texte (en clair) d'explication de U.
- `label`: pour les codes `major` qui en exige un, _label_ en clair à faire figurer dans le document à créer.

Données fixées par le **sponsor SP**:
- `pemS`: clé publique de cryptage du sponsor traitant l'invitation.
- `txti`: texte de réponse du sponsor, crypté par pemS / pemU.
  - si acceptation: texte généré donnant _en langage humain_ les conditions d'application (à validser / décliner par U).
  - si rejet: justificatif textuel de rejet rédigé par le sponsor.
- `role`: classe du document _principal_ associé à la demande et _rôle_ de son credential associé.
- `docId`: ID de ce document.
  - `role / docId` ne sont pas obligaoires: une invitation peut avoir pour objet autre chose que la création d'un document et d'un credential associé.
  - si l'invitation correspond à la création  / communication de plus d'un document, les données correspondantes sont dans `etc`. 
- `cond`: à faire figurer en `cond` du credential du document principal associé à l'invitation.
- `etc`: autres données nécessaires pour créer le ou les documents associés, les credentials à enregistrer, etc. 

### Sponsors
Un _sponsor_ est un utilisateur qui a un (des) credential de _sponsoring_:
- soit le credential `Org.manager` (qui donne droit aux opérations qualifiées de management général): il est _sponsor universel_, il peut répondre à **toutes** les demandes d'invtation.
- soit le credential `Sponsor.` avec un docId de la forme major ou major/minor.

### Major
Une invitation a une cible fonctionnelle bien délimitée dont la liste, fermée, dépend de l'application, représentant en quelque sorte une _classe_ d'invitations. Par exemple:
- `Auteur` : invitation à pouvoir se comporter comme _Auteur_, avoir un document `Auteur` et un credential d'accès (le cas échéant avec des variantes de pouvoir différent).
- `Codir` : invitation à agir en tant que membre du _Comité directeur_ et de prendre les décisions afférentes.
- `Forum` : invitation à faire partie / gérer un _forum de discussion_.

La liste des codes **major** est définie et fermée pour chaque _service_.

### Minor
Une invitation a un code major et **peut spécifier un code minor** selon son major.

Certains major n'ont pas de minor: `Codir` par exemple, on est membre ou non du _comité directeur_ et il n'y en a qu'un dans l'organisation.

**Certains majors ont une liste fermée de minors possibles**: par exemple un _Auteur_ peut avoir une (ou des) prérogatives de sélection d'auteurs selon un _thème_: _science, politique, sociologie ..._ Cette liste,
- peut évoluer au cours du temps: des _thèmes_ nouveaux peuvent apparaître, des thèmes obsolètes disparaître, etc. mais pas à une fréquence frénétique.
- pour un major donné la liste de ses minors déclarés à un instant donné est assez courte pour permettre d'un désigner un dans une liste à l'écran.

**Enfin certains majors ont une liste ouverte de minors possibles:** par exemple des centaines de forums sont possibles et un utilisateur peut désigner celui de son choix par un code qu'il a obtenu quelque part.
- un minor peut aussi être un code _promotion / campagne_, ayant une durée de vie limitée. Les utilisateurs _sponsor_ ont alors un code major/minor comme `Vente/PROMO5J`.
- soit ces codes ont été publiés quelque part, ou diffusés par un media externes, soit ils se sont transmis de bouche à oreille par un mécanisme de cooptation personnelle.

### Régles
- un **manager** est _sponsor_ universel, peut traiter toutes les demandes de tous major.
- un **sponsor** n'ayant qu'un major peut traiter toutes les demanandes d'invitation spécifiant ce major quelque soit le minor spécifié dans la demande (ou l'absence de minor).
  - un _sponsor_ Auteur peut traiter toutes demandes spécifiant _Auteur, Auteur/science Auteur/politique_.
- un **sponsor** ciblé major/minor ne peut traiter que les demandes d'invitation spécifiant exactement ce code major/minor. Par exemple:
  - un sponsor _Auteur/science_ ne peut pas traiter les demandes _Auteur_ ni _Auteur/politique_ mais uniquement celles spécifiant _Auteur/science_.
  - un sponsor _Forum/randojuin26_ ne peut traiter que les demandes spécifiant une volonté de participation au _Forum/randojuin26_.

#### Usage du texte de _motivation_
Quand un utilisateur U fait une demande d'invitation en spécifiant un major ou même un major/minor, le texte de motivation va aider le _sponsor_ la traitant à personnaliser son acceptation. Par exemple:
- _Forum/randojuin26_ : mais le cas échéant il peut y avoir plusieurs forums pertinents, lequel choisir ?
- pour assumer quelle fonction: _simple participant, organisateur, etc._
- ceci va influer sur,
  - le choix du ou des documents à créer : _incription, etc._
  - pouvoirs à conférer dans le ou les credentials accordés: par exemple droit à être sponsor soi-même, à avoir des droits d'animatiuon ou non, etc.

### Retrait des droits de sponsoring
Un _sponsoring_ correspondant à un credential, la logique applicative peut avoir des opérations invalidant un credential de `Sponsor.` (comme de tout autre rôle).

## Contrôle et invalidation de credentials _obsolètes_ ou _indésirables_

Un utilisateur dont le _safe_ a enregistré un _credential_ `C1` n'est jamais assuré que ce credential est toujours valide pour le service / organisation correspondant: 
- il a pu être invalidé (ou son pouvoir modifié).
- il ne le saura que quand il tentera de l'utiliser. 

Un _report_ sur demande de l'utilisateur U liste, à cet instant, les conditions d'exercice et de validité des credentials figurant dans son _safe_: il pourra alors _purger_ ceux obsolètes et dont il estime qu'ils ne pourront pas être re-validés ou leurs pouvoirs modifiés dans le sens qui lui convient.
