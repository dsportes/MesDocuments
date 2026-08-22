---
layout: page
title: Sessions de travail, modes
---

Les **modes** selon lesquels peut se dérouler une session de travail d'un utilisateur pour une _application_ donnée dépendent de deux facteurs:
- Internet accessible ou non,
- Droit à stocker des informations localement sur un terminal déclaré _de confiance_.

### Mode normal _synchronisé_
Internet est accessible et le terminal est de confiance.

Une base locale _Cache_ de nom `app-userId` existe pour chaque utilisateur `userId` ayant ouvert l'application `app`:
- elle mémorise des documents, des collections, et quelques autres données.
- elle maintient à jour certains documents / collections, afin qu'ils reflètent au plus tôt leurs évolutions _notifiées par les services cloud_.
- les données sont cryptées par la clé K de l'utilisateur, indéchiffrables pour tout autre.

### Mode _avion_
Le terminal est de confiance mais Internet n'est pas accessible (du moins déclaré comme tel).

La session après avoir authentifié l'utilisateur dispose de sa clé K et peut utiliser la base locale _Cache_ pour y **lire** les documents et collections qui y sont présents:
- leur mise à jour n'est pas possible.
- des _notes textuelles_ et des _fichiers_ peuvent toutefois y être stockés et retrouvés plus tard au cours d'une session _synchronisée_ (mais ne sont pas accessibles depuis d'autres terminaux).

Documents et collections n'y sont trouvés que pour autant qu'une session en mode _synchronisé_ ayant été ouverte antérieurement sur ce terminal les y ai stockés: ils sont disponible en mode _avion_ dans l'état laissé à la fin de celle-ci.

### Mode _incognito_
Internet est accessible mais le terminal n'est pas de _confiance_: la base locale _Cache_ n'existe pas (ou du moins non recherchée et ignorée).

Les documents et collections sont obtenus à la demande depuis les _services cloud_.

### Mode _calculette_ accessible sans authentification
Sans Internet ni accès aux documents et collections, l'application est très limitée, au plus pouvant afficher quelques informations d'aide inscrite dans le logiciel de l'application et effectuer quelques calculs depuis uniquement des données saisies interactivement (ce qui dépend de ce que l'application a implémenté).

### Avant authentification
**Une _session_ débute après authentification de l'utilisateur.**

Avant, hors session, un utilisateur anonyme peut accéder à quelques informations:
- S'il a accès à Internet sur l'accessibilité de services cloud pour une organisation donnée.
- sinon en mode _calculette_ il dispose des informations:
  - sur la page d'à propos: que fait l'application, comment y obtenir un compte...
  - sur les pages d'aides, des informations distribuées localement avec le logiciel de l'application ayant des liens sur une information / aide en ligne accessible seulement si l'accès Internet est disponible.

# Base locale _Cache_
Elle héberge les données suivantes.

### Options
Un enregistrement conservant les dernières options choisies en cours de session par l'utilisateur afin de les proposer par défaut ultérieurement.

### Préférences
Des enregistrements (identifiés par un nom donné par l'utilisateur) mémorisent les préférences d'affichage / comportement de l'application que l'utilisateur a saisi et décidé de conserver.

### Documents et collections _accessibles en mode avion_
Certaines classes de documents et de collections sont déclarées par l'application comme étant accessibles en mode _avion_:
- en début de session ils sont _synchronisés_ leur état connu en _Cache_ est mis à jour depuis les _services cloud_: un _abonnement_ étant déclaré pour eux, toute notification de changement reçue depuis les services donne lieu à la synchronisation de leur état, en mémoire de la session et en _Cache_, et ce jusqu'à la fin de la session.
- les autres _documents et collections_ non déclarés accessibles en mode avion, sont chargés depuis les _services cloud_ à première demande puis maintenus synchronisés à jour en mémoire de la session jusqu'à sa fin (mais ne sont pas stockés en _Cache_).

### Fichiers
Les documents peuvent avoir des fichiers attachés qui par défaut ne sont pas synchronisés en _Cache_.

L'utilisateur peut marquer certains d'entre eux pour être synchronisés, si la classe de leur document est marquée accessible en mode _avion_. Pour chaque fichier:
- en début de session il est obtenu du service cloud s'il n'est pas à jour et stocké en _Cache_: il sera maintenu à jour jusqu'à la fin de la session.
- en cours de session d'autres fichiers peuvent être marqués par l'utilisateur pour être accessibles en mode _avion_ (la _marque_ pouvant être effacée).
- ce choix ne vaut **QUE** pour ce terminal, pas pour les autres, chacun pouvant avoir des capacités de stockage très différentes (un _giga_ pour un mobile, un _tera_ pour un PC ...) .

### Notes textuelles et fichiers autonomes
L'utilisateur peut stocker en _Cache_ des notes textuelles et des fichiers à qui il a donné un nom.
- ces notes et fichiers NE SONT PAS communiqués aux _services cloud_ et ne sont disponibles que sur CE terminal.
- elles disparaissent si la _Cache_ est réinitialisée.

# Rôles et _périmètres_ de synchronisation
Pour une application donnée, un _credential_ définit un droit d'accès:
- `svc`: pour un des _services cloud_ que l'application a fixé,
- `org`: pour une organisation donnée,
- `docCl` : pour un document (réel ou virtuel) de classe donnée,
- `docPk`: ayant une clé primaire donnée.

Par exemple: `AS2/demo/Auteur/sh(Zola)`

## Périmètres de synchronisation
Un **périmètre** est toujours relatif à un couple `svc org` et est stocké dans le _DocStore_ correspondant.

Pour chaque couple `svc org` pour lesquels il existe un credential, la logique de l'application calcule depuis chaque credential de 0 à N **périmètres** de documents, chacun ayant:
- `id` : l'identifiant est fondé sur le credential source. 
  - Si depuis un credential depuis _Auteur_ (`docCl`) pour une primary key `sh(Zola)`, plus d'un périmètre sont générés, l'application les qualifie par des ID `Auteur@code1/sh(Zola)`, `Auteur@code2/sh(Zola)` ... pour les différencier.
  - s'il n'y en a qu'un `@...` est omis.
- `role`: un code qui correspond à sa _nature / fonctionnalité_ (et un libellé traduit).
- `plane`: un flag indiquant si les documents de ce périmètre sont consultables en mode _avion_.
- `defs`: une **liste** de définitions d'abonnements, chaque terme étant de la forme:
  - soit `classe`
  - soit `classe/pk`
  - soit `classe/collection/pk`

> Pour un credential `docCl docPk` sont des propriétés immuables, en conséquence ils produiront toujours la même liste de périmètres. Pour une session donnée, la liste des credentials peut changer en cours de session, ce qui changera la liste des `IDs` des périmètres gérés (pas le contenu de chacun).

L'application ne génère des périmètres que depuis certaines classes de credentials (pas de tous). C'est une méthode spécifique de chaque sous-classe applicative de _credential_ qui génère le ou les périmètres correspondants en retournant une liste de quadruplets `[id, role, plane, defs]`.

## Rôles
Un rôle peut être considéré comme une _grande fonctionnalité_ de l'application. Un _service_ `Boutique` peut par exemple avoir des rôles `vente, comptabilité, stock, RH`.

En scannant la liste des périmètres générés depuis les _credentials_ d'un utilisateur on obtient la liste des couples `[org role]` depuis lesquels des _groupes de documents synchronisés_ peuvent être obtenus.
- un utilisateur pourrait ainsi avoir des rôles _stock_ et _ventes_ pour une boutique B1, mais seulement un rôle _comptabilité_ pour B2.

> La connaissance des _roles_ que peut tenir un utilisateur a une influence directe sur les possibilités proposées par l'interface graphique: par exemple le UI ne proposera pas de _liens / boutons / pages_ associés aux fonctionnalités _comptables_ pour une boutique pour laquelle l'utilisateur n'a pas les credentials requis. 

### Liste _potentielle_ versus _effective_ de couples `[org, role]`
De l'ensemble des périmètres calculés on obtient un ensemble de couples `org role` (qu'il est possible de proposer à l'écran par exemple par organisation donnant la liste des rôles associés).

Pour une session donnée, possiblement en évolution au cours de la session, l'utilisateur peut déclarer une liste _effective_ de couples `org role`, sous liste de la liste _potentielle_.

Bien que pouvant potentiellement accéder à plusieurs boutiques B1, B2, B3 aux fonctionnalités _vente comptabilité_, l'utilisateur _peut_ par exemple restreindre sa session courante à _comptabilité de B2_ simplement parce qu'à cet instant c'est sa préoccupation. Se faisant,
- son interface ne lui montrera pas les pages / liens / options de menu des autres fonctionnalités (ni des autres organisations),
- seuls les documents des périmètres ayant un de ces couples seront accessibles et synchronisés ce qui va réduire à la fois les temps de chargement et de calcul _cloud_, rendre la session plus légère et plus fluide et moins consommatrice de ressources.

## _Options_
Au cours d'une session un enregistrement `options` détient:
- les listes _potentielle_ et _effective_ des couples `[org, role]` accessibles: seule la liste _effective_ peut être éditée.
- le code / nom de la _préférence_ d'affichage / comportement de l'interface, le code _default_ pouvant être utilisé pour appliquer toutes les valeurs / choix par défaut.

L'enregistrement `options`:
- est présent en mémoire d'une session,
- le dernier état utilisé est sauvegardé dans la _Safe Box_ de l'utilisateur, s'il le souhaite, en modes _synchronisé / incognito_,
- le dernier état utilisé sur un terminal est sauvegardé, s'il le souhaite, en _Cache_ de l'utilisateur en modes _synchronisé / avion_.

**Options selon les différents modes:**
- en mode _incognito_, `options` est obtenu de la _Safe Box_.
- en mode _avion_, `options` est obtenu de _Cache_.
- en mode _synchronisé_, `options` est obtenu par fusion des enregistrement obtenus de _Cache_ et de _Safe Box_, l'état fusionné est sauvegardé en _Cache_ à l'ouverture de la session.

**Règles de _fusion_:**
- la liste _potentielle_ étant calculée depuis les credentials depuis _Safe Box_ n'est pas éditable, celle de _Cache_ est ignorée.
- si la liste _effective_ en _Cache_ existe elle est préférée à celles de _Safe Box_: elle est _normalisée_, les couples `[org, role]` absents de la liste potentielle sont supprimés de la liste _effective_.

# Synchronisation des _périmètres_
La liste des périmètres est calculée depuis les _credentials_ obtenus de la _Safe Box_ : toutefois en mode _avion_ elle est lue de _Cache_.
- en session _synchronisée_ cette liste est sauvegardée en _Cache_ afin d'être rendue disponible en mode _avion_ sur ce terminal.

### États d'un périmètre en _DocStore_  
- `stand-by`: il n'a jamais fait l'objet d'une demande `fetch`, ses documents ne sont pas disponibles en _DocStore_ (du moins pas tous).
- `loading`: il a fait l'objet d'une demande `fetch`,
  - un abonnement a été enregistré pour ses documents / collections auprès du _service cloud_ correspondant,
  - MAIS en _DocStore_ tous ses documents / collections n'ont pas tous encore fait l'objet d'une _synchronisation_ depuis le début de la session: leur présence et _date de dernier rafraîchissement_ est incertaine.
- `ready`: tous ses documents / collections ont fait l'objet d'un abonnement, sont disponibles en _DocStore_ et ont tous encore fait l'objet d'une _synchronisation_ depuis le début de la session.

> Un document (ou une collection) a deux dates: 
> - sa **version** qui est la date-heure de sa modification la plus récente, 
> - sa **date-heure d'assertion** (_de dernier rafraîchissement_) qui est la date-heure de l'opération la plus récente ayant vérifié qu'il n'en existait pas de plus récente. Un document peut avoir une **version** très ancienne et une **date d'assertion** très récente.

Le rôle d'un périmètre peut être déclaré _avion_:
- en mode _synchronisé_ il fera l'objet en début de session d'un `fetch` (avant toute demande) pour autant que son couple `org role` fasse partie des options: l'objectif est de _charger en Cache_ toutes les données utilisables en mode _avion_.
- dans les autres modes on attend les demandes de `fetch` provenant de l'interface avec l'utilisateur sans préchargement en début de session.

La liste des périmètres est calculée avant d'ouvrir une session, mais peut être recalculée en cours de session si la liste des _credentials_ change. 
- Des périmètres peuvent être _ajoutés_ par rapport à la liste actuelle et donner lieu à modification d'abonnement et à des demandes de synchronisation.
- Des périmètres peuvent être _supprimés_: toutefois les documents / collections devenus _inutiles_ ne sont pas purgés de _Cache_. Ils pourraient être ultérieurement être redemandés et au pire finiront par disparaître par _obsolescence_ en ayant une date d'assertion trop vieille.

La mémoire de la session et le _Cache_ sont en conséquence plus ou moins chargés selon l'historique des sessions exécutées sur le terminal:
- l'état de la mémoire de la session peut être affiché sur demande en cours de session.
- HORS session, le _volume utile_ de chaque _Cache_ des utilisateurs ayant déclaré ce terminal de confiance est calculable. Ceux-ci sont censés se connaître et avoir confiance entre eux:
  - ils peuvent supprimer des bases quand ils jugent que le volume occupé sur le terminal est excessif, ce qui ne détruit aucun document, ceux-ci étant toujours disponibles dans le _services cloud_
  - ils ne peuvent en aucune façon accéder aux contenus (cryptés).
- en cours de session, l'utilisateur peut afficher, outre le volume total, un état détaillé du la _Cache_ par périmètre.

# Implémentation

## Authentification et choix d'options
Le choix du mode est fait AVANT authentification, en cochant / décochant les cases _accès à Internet_ et _accès à la mémoire locale du terminal_: dans un _browser_ cette mémoire persistante est cantonnée dans l'espace dédié au _domaine_ du site hébergeant l'application (et d'accès direct difficile, mais quoi qu'il en soit crypté).

**Après authentification**, un choix est proposé si le terminal est considéré de confiance et que l'accès à Internet n'a pas été décoché:
- **mode _sync_ sans reset** de la base _Cache_: coché par défaut.
- **mode _sync_ AVEC reset** de la base _Cache_: cocher ce choix produit une alerte et demande confirmation avant ré-initialisation du _Cache_.

Si le terminal n'est pas de confiance, le mode _incognito_ est le seul choix.

Remarque: en mode _sync_ et _incognito_ la Safe Box est en mémoire, pas en mode _avion_.

### Phase (A) _ouverture / création_ du _Cache_ de l'application
En mode _sync_ à tout moment le choix de la sélection _RESET Cache_, relance cette phase.

#### Mode _sync_
- récupération depuis les credentials de la _Safe Box_ de la liste des couples `svc org`. Pour chaque couple, ouverture d'un _DocStore_.
- calcul des `périmètres` depuis les credentials de la _Safe Box_ et mémorisation en _Cache_.
- récupération des `prefs` de la _Safe Box_ et inscription en _Cache_.
- récupération des `options` de la _Safe Box_ et inscription en _Cache_.

#### Mode _avion_
Si l'item _options_ n'existe pas en base _Cache_, c'est qu'elle vient d'être créée vide: alerte, mode impraticable faute de données et retour au _login_.
- récupération des `périmètres`, des `prefs` et des `options` depuis _Cache_.

#### Mode _incognito_
- récupération depuis les credentials de la _Safe Box_ de la liste des couples `svc org`. Pour chaque couple ouverture d'un _DocStore_.
- calcul des `périmètres` depuis les credentials de la _Safe Box_.

### En fin de cette phase
La mémoire de la session dispose:
- des options:
  - `[org roles]`: liste _potentielle / effective_, 
  - `pref`: dernière préférence choisie.
- des **préférences**,
- des **périmètres** calculés,
- des **credentials**.

### Phase (B) : sélection des options
Il est proposé à l'utilisateur d'éditer s'il le souhaite,
- la sélection effective des `[org role]` proposée,
- le choix de la préférence d'affichage / comportement proposée,
- puis de valider ses choix.

> Dans le cas le plus simple, après authentification l'utilisateur a un seul appui sur un bouton pour lancer sa session.

Depuis cette page en mode _sync_ et _incognito_ l'utilisateur peut ouvrir sa _Safe Box_ et y mener plusieurs actions. Certaines d'entre elles provoquent le retour à la phase (B) et le ré-affichage de la sélection des options:
- suppression de credentials,
- édition d'une ou plusieurs préférences.

## Initialisation de la session
### Mode _sync_
L'objectif est de charger le maximum de données requises en mode _avion_ dans _Cache_ (et en _DocStore_).

#### Interprétation de la liste des périmètres
De cette liste et des couples `[org role]` de `options`, on obtient la liste des _defs_ des souscriptions _à abonner et synchroniser_,
- pour chaque `def` les paramètres éventuels de la notification à produire (titre, texte ...) sont présents.

Les _document / collection_ des périmètres _actifs_ sont lus de _Cache_ et inscrits en _DocStore_ qui en détient désormais la version connue en session.

**Phase 2 : génération de la souscription**. Pour chaque `def`:
- un item de souscription est généré en fonction:
  - de la version du document / collection,
  - du _message_ éventuel associé à `def`:
    - la `docCl` détermine si oui ou non ce message est à générer,
    - la présence et la personnalisation éventuelle du message par l'application peut faire intervenir: a) le champ _name_ du credential associé par `docPk`, b) des flags / libellés inscrits par l'utilisateur dans ses préférences.
- une souscription globale est émise pour chaque couple `svc org` au site du service cloud qui le gère.
- les demandes de synchronisation sont poussées dans la _syncQueue_ de `svc org` pour chaque item marqué _pas encore synchronisé_ (tous en fait dans le cas d'initialisation de la session).

Le retour de chaque synchronisation est inscrit,
- dans son _DocStore_,
- dans _Cache_.

### Mode _incognito_
Même scénario que pour le mode _sync_ avec les variantes ci-après:
- les lectures à _Cache_ retourne un résultat négatif (_pas trouvé_), donc il n'y a pas de chargement initial de _DocStore_ depuis _Cache_.
- les écritures en _Cache_ ne font rien.
- aucun périmètre n'est _actif_, ils seront chargés à première demande.

### Mode _avion_
Principes: 
- aucun périmètre n'est _actif_: le _DocStore_ n'est pas préchargé, mais se charge au fur et à mesure des demandes de `fetch`.

## DocStore `fetch get...` : accès aux _document / collection_

Quand une vue a besoin d'afficher des _document / collection_ (ou effectuer des calculs sur ceux-ci) il faut qu'elle s'assure de la présence en _DocStore_ du _périmètre_ les incluant en invoquant un `fetch` l'identifiant:
- s'il s'agit d'un périmètre _actif_ (ceux marqués _avion_ l'étant toujours en mode _synchronisé_), tous ses `defs` sont déjà inscrits comme synchronisés.
- si tous les `def` du périmètre demandé sont déjà inscrits en _DocStore_ comme synchronisé (ou sont en _syncQueue_), le périmètre peut être accédé. Au pire certaines parties sont incomplètes et seront mises à jour par réactivité en retour d'une synchronisation  suite à notification.
- en mode _synchronisé_ et _incognito_,
  - la souscription actuelle _peut_ être augmentée pour les inclure et retransmise au service cloud,
  - des requêtes de synchronisation des `defs` qui ne le sont pas encore sont inscrite en _syncQueue_.

Un périmètre ajouté devient _actif_. Si l'utilisateur modifie ultérieurement ses options, de nouveaux périmètres peuvent être générés, certains `actif`, les autres `loading`.

Dans une vue UI, après la prise de précaution d'un _fetch_ d'un ou plusieurs périmètres, les _document / collection_ du _DocStore_ affichables / calculables sont simplement obtenus par un `getDoc / getColl` qui en retourne l'état courant (un `getDoc` d'un `def` non synchronisé échoue).

### Variante en mode _avion_
Par principe en _DocStore_ rien n'est marqué _synchronisé_ à l'initialisation.

Sur demande d'un _périmètre_,
- s'il est déjà marqué _actif_, rien à faire.
- sinon,
  - le _périmètre_ est marqué _loading_,
  - puis les _document / collection_ sont chargés depuis _Cache_.

## Mise à jour des _options_ en cours de session
### Mode _avion_ 
Aucune influence vis à vis du _DocStore_ ni sur l'état des souscriptions.

La liste des organisations peut être réduite / étendu ce qui ne sera visible que sur les menus de choix des organisations.

La liste des rôles a aussi une influence sur les options proposées à l'écran.

La mémorisation ou non de la liste `[org role]` dans _Cache_ est une option qui peut être cochée / décochée par l'utilisateur.

### Mode _sync_ et _incognito_
**Régénération de la souscription**. Pour chaque `def` des périmètres un item de souscription est généré en fonction:
- de la version du document / collection,
- du _message_ éventuel associé à `def`:
  - la `docCl` détermine si oui ou non ce message est à générer,
  - la présence et la personnalisation éventuelle du message par l'application peut faire intervenir: a) le champ _name_ du credential associé par `docPk`, b) de flags / libellés inscrits par l'utilisateur dans ses préférences.

La souscription globale est émise pour chaque couple `svc org` au site du service cloud qui le gère: les périmètres _actifs_ sont inclus.
- normalement si la liste des périmètres n'a pas changé on pourrait penser que la souscription antérieure convient encore.
- toutefois même dans ce cas, les _messages_ de notification pour chaque `def` _peuvent_ avoir évolué si la préférence courante a été changé ou éditée. Si les _messages_ antérieurs sont identiques, la souscription n'est pas ré-émise et aucune resynchronisation n'est à effectuer.

**En mode _sync_ seulement**, pour les seuls périmètres ajoutés / modifiés marqués _avion_, les _document / collection_ sont lus de _Cache_ et inscrits en _DocStore_. Les demandes de synchronisation sont poussées dans la _syncQueue_ de `svc org` pour chaque item marqué _loading_.

## Clôture de la session
Pour le mode _avion_ il n'y a rien de particulier à faire.

Dans les autres modes il faut basculer les souscriptions vers celles qui sont à activer ou rester actives _hors ligne_.

L'utilisateur _peut_ fixer à cette occasion des **options de clôture**:
- ne plus émettre que certaines des notifications (ou aucune),
- fixer certains messages ...

**Régénération de la souscription**. Pour chaque `def`un item de souscription est généré en fonction:
- de la version du document / collection,
- du _message_ éventuel associé à `def`:
  - la `docCl` détermine si oui ou non ce message est à générer,
  - la présence et la personnalisation éventuelle du message par l'application peut faire intervenir: a) le champ _name_ du credential associé par `docPk`, b) de flags / libellés inscrits par l'utilisateur dans ses préférences, c) les options de clôture.

Pour chaque couple `svc org` la souscription régénérée (ou supprimée) est émise au service cloud en charge.
