---
layout: page
title: Sessions de travail, modes
---

Les **modes** selon lesquels peut se dérouler une session de travail d'un utilisateur pour une _application_ donnée dépendent de deux facteurs:
- Internet accessible ou non,
- Droit à stocker des informations localement sur le terminal, celui-ci ayant été déclaré _de confiance_.

### Mode normal _synchronisé_
Internet est accessible et le terminal est de confiance.

Une base locale _Cache_ de nom `app-userId` existe pour chaque utilisateur `userId` ayant ouvert l'application `app`:
- elle mémorise des documents, des collections, et quelques autres données.
- elle maintient à jour certains documents / collections, afin qu'ils reflètent au plus tôt leurs évolutions _notifiées par les services cloud_.
- les données sont cryptées par la clé K de l'utilisateur, indéchiffrables pour tout autre.

### Mode _avion_
Le terminal est de confiance mais Internet n'est pas accessible (du moins déclaré comme tel).

La session, ayant authentifié l'utilisateur (donc disposant de sa clé K), peut utiliser la base locale _Cache_ pour y **lire** les documents et collections qui y sont présents:
- leur mise à jour n'est pas possible.
- des _notes textuelles_ et des _fichiers_ peuvent toutefois y être stockés et retrouvés plus tard au cours d'une session _synchronisée_ (mais ne sont pas accessibles depuis d'autres terminaux).

Documents et collections n'y sont trouvés que pour autant qu'une session en mode _synchronisé_ ait été ouverte antérieurement sur ce terminal: ils y sont dans l'état laissé à la fin de celle-ci.

### Mode _incognito_
Internet est accessible mais le terminal n'est pas de _confiance_: la base locale _Cache_ n'existe pas (ou du moins ignorée et cherchée).

Les documents et collections sont obtenus à la demande depuis les _services cloud_.

### Mode _calculette_
Sans Internet ni accès aux documents et collections, l'application est très limitée, au plus pouvant afficher quelques informations d'aide inscrite dans le logiciel de l'application et effectuer quelques calculs depuis uniquement des données saisies interactivement (ce qui dépend de ce que l'application a implémenté).

# Base locale _Cache_
Elle héberge les données suivantes.

### options
Un enregistrement conservant les dernières options choisies en cours de session par l'utilisateur afin de les proposer par défaut ultérieurement.

### préférences
Des enregistrements (identifiés par un nom donné par l'utilisateur) mémorisent les préférences d'affichage / comportement de l'application que l'utilisateur a saisi et décidé de conserver.

### documents et collections _accessibles en mode avion_
Certaines classes de documents et de collections sont déclarés par l'application comme étant accessibles en mode _avion_:
- en début de session ils sont _synchronisés_ leur état connu en _Cache_ est mis à jour depuis les _services cloud_: un _abonnement_ étant déclaré pour eux, toute notification de changement reçue depuis les services donne lieu à la synchronisation de leur état, en mémoire de la session et en _Cache_, et ce jusqu'à la fin de la session.
- les autres _documents et collections_ non déclarés accessibles en mode avion, sont chargés depuis les _services cloud_ à première demande puis maintenus synchronisés à jour en mémoire de la session jusqu'à sa fin (mais ne sont pas stockés en _base locale Cache_).

### fichiers
Les documents peuvent avoir des fichiers attachés qui par défaut ne sont pas synchronisés base locale _Cache_.

L'utilisateur peut marquer certains d'entre eux pour être synchronisés, si la classe de leur document est marquée accessible en mode _avion_. Pour chaque fichier:
- en début de session il est obtenu du service cloud s'il n'est pas à jour et stocké en _Cache_: il sera maintenu à jour jusqu'à la fin de la session.
- en cours de session d'autres fichiers peuvent être marqués par l'utilisateur pour être accessibles en mode _avion_ (la _marque_ pouvant être effacée).
- ce choix ne vaut **QUE** pour ce terminal, pas pour les autres, chacun pouvant avoir des capacités de stockages très différentes (un _giga_ pour un mobile, un _tera_ pour un PC ...) .

### Notes textuelles et fichiers autonomes
L'utilisateur peut stocker dans la _base locale Cache_ des notes textuelles et des fichiers à qui il a donné un nom.
- ces notes et fichiers NE SONT PAS communiqués aux _services cloud_, et ne sont disponibles que sur le terminal.
- elles disparaissent quand la _base locale Cache_ est réinitialisée.

# _Périmètres_ de synchronisation
Pour une application donnée, un _credential_ définit un droit d'accès:
- `svc`: pour un des _services cloud_ que l'application a fixé,
- `org`: pour une organisation donnée,
- `docCl` : pour un document (réel ou virtuel) de classe donnée,
- `docPk`: ayant une clé primaire donnée.

Par exemple: `AS2/demo/Auteur/sh(Zola)`

La logique de l'application calcule depuis chaque _credential_ détenu par l'utilisateur, une liste de **périmètres** de documents. Chaque périmètre défini a:
- un `type` qui correspond à sa _nature / fonctionnalité_ (et un libellé traduit).
- des paramètres,
  - `svc`: le service cloud concerné.
  - `org`: l'organisation concernée,
  - `S1 S2 ...` de 0 à N identifiants de documents, le paramètre `$i` étant l'identifiant d'une document d'une classe précise.
- une **liste** de définitions d'abonnements, chaque terme `def` étant de la forme:
  - soit `classe`
  - soit classe/$i
  - soit classe/collection/$j
- par exemple pour un credential `Auteur/$1`,
  - def: `Auteur/$1` : le document Auteur lui-même de `$1`.
  - def: `Article/auteurs/$1` : tous les articles dont un des auteurs est $1.

Selon son _type_, un périmètre est _accessible en mode avion_ ou non. 

_Remarque_: l'application ne génère des périmètres qu'autour de certains credentials (pas de tous).

Ainsi en début de session, compte tenu de la liste des _credentials_ détenus par l'utilisateur, une liste des _périmètres_ correspondant est calculable, chacun étant ou non _accessible en mode avion_.
- la liste des périmètres est ré-évaluée si des credentials sont ajoutés ou retirés en cours de session.

## Filtrage des périmètres par les _rôles_
L'application fixe une liste fermée de **rôles**:

L'utilisateur peut sélectionner en début de session (puis en cours de session) certains de ces _rôles_, à la limite tous.
- le calcul des _périmètres_ exclut ou inclut ceux-ci en fonction de cette liste,
- chaque type de périmètre est configuré avec une liste de rôles, et n'est généré que si l'un de ces rôles a été cité par l'utilisateur pour sa session.

La liste des documents / collections synchronisables est plus ou moins réduite en fonction de la tâche spécifique que l'utilisateur souhaite entreprendre pour cette session:
- les vues sont simplifiées en ne montrant que ce qui est utile pour cette tâche,
- le chargement des données est plus rapide et économique en évitant de charger des documents dont l'utilisateur n'aura pas besoin pour ce qu'il compte faire à ce moment,
- l'utilisateur peut facilement ajuster son choix en cours de session.

### Mémorisation dans `options` de la base locale Cache et la Safe Box
La dernière liste de _rôles_ fixée par l'utilisateur est mémorisée,
- dans _la base locale Cache_, sauf en mode _incognito_ (il n'y pas de _Cache_),
- sur demande explicite d'une case à cocher, dans la _Safe Box_ de l'utilisateur, sauf en mode _avion_ (pas d'Internet).

## Filtrage des périmètres par les _organisations_
L'utilisateur peut cocher / décocher les organisations pour lesquelles il a un ou des credentials:
- l'application au cours de son calcul des _périmètres_ ignore les credentials portant sur les organisations décochées.
- les vues sont réduites, focalisées sur les seules organisations ciblées par l'utilisateur pour la session qui s'ouvre.
- l'occupation de la mémoire et les accès réseau sont réduits en conséquence, la session est accélérée et moins consommatrice.
- l'utilisateur peut changer son choix à tout instant en cours de session ce qui provoquera un recalcul des _périmètres_.

### Mémorisation dans `options` de la base locale Cache et la Safe Box
La dernière liste des _organisations_ fixée par l'utilisateur est mémorisée,
- dans _la base locale Cache_, sauf en mode _incognito_ (il n'y pas de _Cache_),
- sur demande explicite d'une case à cocher, dans la _Safe Box_ de l'utilisateur, sauf en mode _avion_ (pas d'Internet).

# Mémorisation des _options_
Outre, a) la dernière liste de _rôles_ utilisée, b) la dernière liste des organisations utilisée, _options_ mémorise **le nom de la dernière _préférence_** d'affichage / comportement sélectionnée par l'utilisateur
en _base locale Cache_ (sauf mode _incognito_).

En effet les _préférences_ dépendent beaucoup du type de terminal sur lequel l'utilisateur travaille.

# Synchronisation des _périmètres_
Les périmètres sont calculés avant d'ouvrir une session, mais ils peuvent être recalculés en cours de session sur demande de l'utilisateur à changer ses _options_.

Quand les _périmètres_ changent,
- les abonnements à être notifiés des changements des documents / collections aux différents _service cloud_ pour chaque organisation concernée sont renvoyés aux services.
- les documents / collections marqués _accessibles en avion_ sont re-synchronisés au plus tôt afin que la _base locale Cache_ soit rechargée au maximum des données accessibles en mode _avion_.
- a contrario ceux marqués non accessibles en mode _avion_,
  - soit continuent à être synchronisés s'ils l'étaient avant le changement,
  - soit ne le seront qu'à l'occasion de la prochaine demande de ces données.

La mémoire de la session et la _base locale Cache_ sont en conséquence plus ou moins chargées selon l'historique des sessions exécutées sur le terminal:
- l'état de la mémoire de la session peut être affiché sur demande en cours de session.
- HORS session, le _volume utile_ de chaque _base locale Cache_ des utilisateurs ayant déclaré ce terminal de confiance est calculable. Ttous les utilisateurs ayant déclaré ce terminal de confiance sont censés se connaître et avoir confiance entre eux:
  - ils peuvent supprimer des bases quand ils jugent que le volume occupé est excessif, ce qui ne détruit aucun documents toujours disponibles dans le _services cloud_
  - ils ne peuvent en aucune façon accéder aux contenus (cryptés).
- en cours de session, l'utilisateur peut afficher, outre le volume total, un état détaillé de la _base locale Cache_ par périmètre.

# Implémentation

## Authentification et choix d'options
Le choix du mode _avion_ est fait à ce moment en cas de sélection du non accès à Internet.

Après authentification, un choix est proposé si le terminal est considéré de confiance et que l'accès à Internet n'a pas été décoché:
- mode _sync_ sans reset de la base _Cache_: coché par défaut.
- mode _sync_ AVEC reset de la base _Cache_: cocher ce choix produit une alerte et demande confirmation.

Si le terminal n'est pas de confiance, le mode _incognito_ est le seul choix.

Remarque: en mode _sync_ et _incognito_ la Safe Box est en mémoire, pas en mode _avion_.

### Phase (A) _ouverture / création_ de la base _Cache_ de l'application
En mode _sync_ à tout moment le choix de la sélection _RESET Cache_, relance cette phase.

#### Mode _sync_
- récupération depuis les credentials du _Safe_ de la liste des couples `svc org`. Pour chaque couple,
  - ouverture d'un _DocStore_,
  - stockage d'un résumé (`docCl docPk`) pour chaque credential relatif au couple `svc org`.
- stockage dans `creds` de _Cache_ de la liste des résumés des credentials (`svc org docCl docPk`).
- récupération des `prefs` du _Safe_ inscription en _Cache_.
- récupération des `options` du _Safe_ et inscription en _Cache_, seulement si _Cache_ n'en n'a pas:
  - `orgs`: de la dernière sélection des organisations,
  - `roles`: de la dernière sélection des rôles: un rôle est relatif à un service cloud `svc/role`.
  - `pref`: du nom de la dernière préférence choisie.

#### Mode _avion_
Si l'item _options_ n'existe pas en base _Cache_, c'est qu'elle vient d'être créée vide: alerte, mode impraticable faute de données et retour au _login_.

Actions:
- récupération des résumés des credentials de _Cache_ et ouverture d'un _DocStore_ pour chaque couple `svc org`.
- récupération des `prefs` et des `options`.

#### Mode _incognito_
- récupération depuis les credentials du _Safe_ de la liste des couples `svc org`. Pour chaque couple ouverture d'un _DocStore_.

A la fin de cette phase, la mémoire de la session dispose:
- des options:
  - `orgs`: liste des organisations, 
  - `roles`: liste des rôles, 
  - `pref`: dernière préférence choisie.
- de la map des **préférences**,
- d'un résumé de tous les **credentials**.

### Phase (B) : sélection des options
Il est proposé à l'utilisateur de changer à son gré les trois sélections `orgs roles pref` proposées par défaut, puis de valider son choix.

> Dans le cas le plus simple, après authentification l'utilisateur a un seul appui sur un bouton pour lancer sa session.

Depuis cette page en mode _sync_ et _incognito_ l'utilisateur peut ouvrir sa _Safe Box_ et y mener plusieurs actions. Certaines d'entre elles provoquent le retour à la phase (B) et le ré-affichage de la sélection des options:
- suppression de credentials,
- édition d'une ou plusieurs préférences.

## Initialisation de la session
### Mode _sync_
L'objectif est de charger le maximum de données requises en mode _avion_ dans la _Cache_ (et en _DocStore_).

#### Calcul des périmètres, souscription
Une classe de l'application effectue ce calcul, pour chaque _DocStore_ `svc org` depuis:
- les résumés des credentials stockés dans le _DocStore_ à l'initialisation,
- les options `orgs` et `roles`.

De cette liste, on obtient la liste des _defs_ des souscriptions _requises_ et _optionnelles_ et pour chacune les paramètres éventuels de la notification à produire (titre, texte ...).

Cette liste est conservée en _DocStore_.

Les _document / collection_ des périmètres sont lus de _Cache_ et inscrits en _DocStore_ qui en détient désormais la version connue en session.

**Phase 2 : génération de la souscription**. Pour chaque `def`:
- un item de souscription est généré en fonction:
  - de la version du document / collection,
  - du _message_ éventuel associé à `def`:
    - la `docCl` détermine si oui ou non ce message est à générer,
    - la présence et la personnalisation éventuelle du message par l'application peut faire intervenir: a) le champ _name_ du credential associé par docPk, b) de flags / libellés inscrits par l'utilisateur dans ses préférences.
- une souscription globale est émise pour chaque couple `svc org` au site du service cloud qui le gère.
- les demandes de synchronisation sont poussées dans la _syncQueue_ de `svc org` pour chaque item marqué _pas encore synchronisé_ (tous en fait dans le cas d'initialisation de la session).

Le retour de chaque synchronisation est inscrit,
- dans son _DocStore_,
- dans _Cache_.

### Mode _incognito_
Même scénario que pour le mode _sync_ avec les variantes ci-près:
- les lectures à _Cache_ retourne un résultat négatif (_ps trouvé_),
- les écritures ne font rien.

### Mode _avion_
Principe: le _DocStore_ n'est pas préchargé, mais se charge au fur et à mesure des demandes de `fetch`.

#### Calcul des périmètres, souscription
Une classe de l'application effectue ce calcul, pour chaque _DocStore_ `svc org` depuis:
- les résumés des credentials sont obtenus de _Cache_ et stockés dans le _DocStore_,
- les options `orgs` et `roles`.

De cette liste, on obtient la liste des _defs_ des souscriptions: celles _requises_ sont inscrites comme _optionnelles_. Les paramètres éventuels des notifications ne sont pas générés.

Cette liste est conservée en _DocStore_ où pour l'instant les périmètres _optionnels_ (tous) sont marqués non chargés.

## DocStore `fetch get...` : accès aux _document / collection_
Quand une vue a besoin d'afficher des _document / collection_ (ou effectuer des calculs sur ceux-ci) il faut qu'elle s'assure de la présence en _DocStore_ du _périmètre_ les incluant en invoquant un `fetch` avec son type et ses arguments:
- s'il s'agit d'un périmètre _NON optionnel_ ou d'un _optionnel_ marqué comme _synchronisé_, tous ses `defs` sont déjà inscrits comme synchronisés.
- si tous les `def` du périmètre sont déjà inscrits en _DocStore_ comme synchronisé (ou sont en _syncQueue_), le périmètre peut être accédé. Au pire certaines parties sont incomplètes et seront mises à jour par réactivité en retour d'une synchronisation  suite à notification.
- mais certains `def` peuvent ne pas être synchronisés:
  - la souscription actuelle est augmentée pour les inclure et retransmise au service cloud.
  - des requêtes de synchronisation des `defs` qui ne le sont pas encore sont inscrite en _syncQueue_.
- le périmètre (_optionnel_ donc) est inscrit dans la liste des périmètres optionnels _synchronisés_. Si l'utilisateur modifie ultérieurement ses options, les périmètres seront régénérés et les périmètres optionnels _synchronisés_ seront ajoutés d'office.

Après la prise de précaution d'un _fetch_, les _document / collection_ du _DocStore_ sont simplement obtenu par un `getDoc getColl` qui en retourne l'état courant (un `getDoc` d'un `def` non synchronisé échoue).

### Variante en mode _avion_
Par principe en _DocStore_ rien n'est marqué _synchronisé_ à l'initialisation.

Sur demande d'un _périmètre_,
- s'il est marqué comme chargé retour immédiat.
- sinon,
  - le _périmètre_ est marqué _chargé_
  - puis les _document / collection_ sont chargés depuis _Cache_.

## Mise à jour des _options_ en cours de session
### Mode _avion_ 
Aucune influence vis à vis du _DocStore_ ni sur l'état des souscriptions.

La liste des organisations peut être réduite / étendu ce qui ne sera visible que sur les menus de choix des organisations.

La liste des rôles a aussi une influence sur les options proposées à l'écran.

La mémorisation ou non des nouvelles listes orgs / roles dans _Cache_ est une option qui peut être coché par l'utilisateur.

### Mode _sync_ et _incognito_
**Régénération de la souscription**. Pour chaque `def`un item de souscription est généré en fonction:
- de la version du document / collection,
- du _message_ éventuel associé à `def`:
  - la `docCl` détermine si oui ou non ce message est à générer,
  - la présence et la personnalisation éventuelle du message par l'application peut faire intervenir: a) le champ _name_ du credential associé par `docPk`, b) de flags / libellés inscrits par l'utilisateur dans ses préférences.

La souscription globale est émise pour chaque couple `svc org` au site du service cloud qui le gère: les périmètres _optionnels_ chargés sont inclus.
- normalement si la liste des périmètres n'a pas changé on pourrait penser que la souscription antérieure convient encore.
- toutefois même dans ce cas, les _messages_ de notification pour chaque def _peut_ avoir évolué si la préférence courante a été changé ou édité. Si les _messages_ antérieurs sont identiques, la souscription n'est pas ré-émise et aucune resynchronisation n'est à effectuer.

**En mode _sync_ seulement**, pour les seuls périmètres ajoutés, les _document / collection_ sont lus de _Cache_ et inscrits en _DocStore_.

Les demandes de synchronisation sont poussées dans la _syncQueue_ de `svc org` pour chaque item marqué _pas encore synchronisé_.

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

