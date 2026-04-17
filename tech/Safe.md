---
layout: page
title: Utilisateurs et "coffres forts"
---

Un _utilisateur_ désigne ici une personne physique utilisant une application depuis un _terminal_.

Les _utilisateurs_ sont **anonymes** dans la mesure où rien, aucun identifiant, n'effectue une corrélation entre un utilisateur et la personne réelle dans la vraie vie.
- un utilisateur est identifié par un `userId` généré aléatoirement à son enregistrement en tant qu'utilisateur.

Tout utilisateur dispose d'un _coffre fort_ (_safe_) contenant principalement ses données confidentielles d'authentification et ses droits d'accès aux applications.
- l'identifiant d'un _coffre fort_ est le `userId` de son propriétaire.

## Safe stores
Les _safe stores_ sont des services Web cryptés / sécurisés de stockage du _safe_ des utilisateurs.
- le code du service est `SAFE`.
- **le _code_ d'un safe store est celui de l'opérateur qui l'héberge**: l'URL d'accès est celle déclarée pour l'opérateur et le service SAFE.
- l'interface d'accès HTTP comporte une vingtaine de requêtes.
- un script PHP en donne, à titre de contribution libre sans obligation d'usage, une implémentation pour un site Web hébergé utilisant une base MySQL d'une seule table.

Le _safe store_ **STANDARD** est une implémentation dont l'URL d'accès est fixée dans le source des applications: c'est par défaut celle proposée lors de la création de son _safe_ par un utilisateur.

> Le contenu du _safe_ d'un utilisateur U est _crypté_ et personne d'autre que U ne peut, ni lire son contenu, ni lui adresser des requêtes.

**Exception à cette règle**: un utilisateur A disposant de l'ID d'un utilisateur U et d'un de ses _alias_, **peut**,  y faire enregistrer dans son _safe_ des _invitations spontanées_.
- chacune a une durée de vie de quelques jours.
- in fine chaque invitation doit être validée par l'utilisateur (ou déclinée) pour être effective. Il n'y a pas de validation par défaut.

### Backups, transfert d'un _safe_ d'un store à un autre
L'utilisateur U peut:
- effectuer un backup de son propre _safe_ dans un format standardisé indépendant de l'opérateur assurant son stockage. L'utilisateur doit prouver en être le propriétaire pour pouvoir effectuer cette opération.
  - un backup est un fichier externe crypté par un mot de passe long donné par l'utilisateur.
- importer un _backup_ de son _safe_ dans un autre store (géré par un autre opérateur):
  - il doit prouver être propriétaire du _safe_ extrait du backup.
  - l'ancien opérateur gérant son _safe_ est invalidé et le nouveau validé; un utilisateur n'a qu'un seul _safe_ utilisable en ligne à un instant donné.

Un utilisateur peut aussi détruire son propre _safe_, après avoir prouvé en être vraiment le propriétaire.

> L'utilisateur peut _lire_ l'intégralité du contenu de son _safe_, sauf ses phrases secrètes qui ne résident en clair que dans sa tête: lire est un bien grand mot, l'essentiel des données y étant des données cryptographiques binaires non interprétables humainement.

### Données _publiques_ du safe d'un utilisateur
> N'importe quel utilisateur A peut lire et utiliser les informations **publiques** d'un autre utilisateur U sans aucun risque pour U.

Parmi ces quelques données:
- **ID, clé C et clé V** sont immuables, fixées à la création et ne peuvent pas changer,
- **alias 1, alias 2, opérateur gérant le _safe_**, peuvent être modifiées de par la seule libre initiatives de leur propriétaire U.

#### Immuables fixées à la création
L'ID est une donnée publique immuable, non interprétable humainement puisque générée aléatoirement à la création.

A la création sont deux couples de clés sont générés pour un utilisateur U:
- **un couple de clés de cryptage C / décryptage D:**
  - la clé de cryptage C est **publique**: elle permet à un autre utilisateur A de crypter un texte que seul U peut décrypter s'il a accès à la clé publique C de A et à sa propre clé privée D.
  - la clé de décryptage D est **privée**: seul U en a connaissance.
- **un couple de clés de vérification V / signature S:**
  - la clé de vérification V est **publique**: elle permet à un autre utilisateur A de vérifier qu'un texte donné a bien été signé par U.
  - la clé de signature S est **privée**: seul U en a connaissance.

#### Opérateur gestionnaire du _safe_ de U
Cette donnée est publique et peut changer sur décision d'importation d'un backup de son _safe_ par U.

#### Alias _modifiables_ de l'ID
Un utilisateur U peut déclarer à un instant donné jusqu'à 2 alias de son ID:
- un alias a au moins 10 signes.
- à un instant donné un alias donné ne peut correspondre qu'un seul utilisateur U.
- seul U peut changer / supprimer ses alias, du moment que les nouveaux proposés ne sont pas déjà les alias d'un autre utilisateur.

> Un alias _peut_ être un texte relatif à l'utilisateur comme son numéro de mobile, une de ses adresses e-mail, son prénom / nom etc. Mais il peut aussi n'être qu'un pseudo sans aucune relation avec le propriétaire de l'ID. Connaître un alias d'une ID ne donne aucune indication utilisable sur son propriétaire dans le vrai monde.

## Master Directory
Le _master directory_ est un service standard unique:
- son URL d'accès figure _en dur_ dans les applications déployées.
- ce directory a une _entrée_ par utilisateur, son ID est celle de l'utilisateur qui est aussi celle de son _safe_ actif stocké chez l'opérateur choisi par l'utilisateur.

> L'entrée pour U dans le _Master Directory_ mémorises les 6 données _publiques_ listées ci-avant.

### Propriétés d'une entrée du Master Directory
Chaque entrée correspondant au _safe_ d'un utilisateur U:
- `id` : ID de l'utilisateur (et clé primaire d'accès).
- `hsha1`: SHA raccourci du Strong Hash de l'alias 1 de U (index unique d'accès) ou une chaîne vide.
- `hsha2`: SHA raccourci du Strong Hash de l'alias 2 de U (index unique d'accès) ou une chaîne vide.
- `C` : clé C publique de cryptage de U.
- `V` : clé V publique de vérification de U.
- `store`: code de l'opérateur hébergeant actuellement le safe de U, ou une chaîne vide si c'est le storage STANDARD.

#### Un alias est enregistré
- **crypté pour U seulement dans son _safe_**: l'utilisateur U peut lire ses propres alias courants mais même le piratage de la DB hébergeant le _safe_ ne permet pas d'y accéder.
- **haché dans le Master Directory**: un utilisateur A peut accéder à l'entrée de U dans le Master Directory en saisissant un des alias de U, mais le Master Directory n'enregistre pas la valeur en clair de l'alias, même le piratage de la DB hébergeant le _master directory_ ne permet pas d'y accéder.

Un utilisateur A peut accéder à ces données de U en fournissant un de ses alias de U, que U a plus ou moins _publié / distribué_ et peut ainsi:
- **crypter un texte** que seul U pourra lire ultérieurement en utilisant sa clé D. En général A joint au texte crypté, soit sa propre clé C pour que U puisse décrypter le texte transmis.
- **vérifier la signature d'un texte** censé avoir été signé par U l'effectivement été.
- **envoyer au _safe_ de U une _invitation spontanée_**: _à rejoindre un groupe, à créer un compte, à détenir des droits d'accès_, etc.

## Contenu d'un _safe_
Cet enregistrement comporte plusieurs sections:
- une section **d'authentification** réunissant les données cryptographiques requises à authentifier son propriétaire.
- une section **terminaux de confiance** conservant les données cryptographiques permettant de certifier qu'un terminal est _de confiance_ afin,
  - de s'authentifier par un code PIN plus léger que le couple _fort_ (alias et phrase secrète) requis sur un terminal non certifié de confiance.
  - d'y disposer de _caches de données_ cryptées sur le terminal pour pouvoir accélérer l'initialisation des applications et leur accès en **mode AVION** sans réseau.
- une section **droits d'accès** conservant les données cryptographiques et descriptives de chaque droit d'accès aux opérations des services pouvant être sollicités.
- une section **profils** enregistrant pour chaque application des _profils_ (nommés _accès limité à Bob et Alice, responsable d'IDF ..._) chacun contenant la sous-liste de ses droits d'accès adaptée à un usage spécifique d'une application.
- une section **préférences** (_settings_) où l'utilisateur peut enregistrer les jeux nommés (_mobile, écran large, expert, simplifié ..._) de paramètres de préférence de comportement et d'affichage de ses applications favorites.
- une section **invitations** où l'utilisateur enregistre ses souhaits à recevoir des invitations et les propositions faites par d'autres utilisateurs _sponsors_ pour les satisfaire.

#### Authentification d'un utilisateur
Un utilisateur peut s'authentifier, depuis n'importe quel terminal (et même en mode _avion_) de manière **forte** en fournissant le couple d'un _alias_ et d'une _phrase secrète_.
- pour éviter les effets dramatiques des oublis, 2 alias at 2 phrases secrètes sont possibles (mais l'oubli des deux est irrémédiable).
- le nombre de tentative n'est pas limité mais une attaque par force brute n'a aucune chance de réussir en suivant un minimum de règles de bon sens.

Depuis un terminal **certifié** par lui-même, un utilisateur peut s'authentifier (mais pae moe _avion_) par un code PIN d'au moins 8 signes. Il n'a toutefois droit qu'à deux tentatives infructueuses.

# Gestion des _credentials / droits d'accès_ dans les applications

Voir dans _[Architecture pour des applications réactives](../Applications.html)_ le détail sur le mécanisme des _credentials_ et leur signature.

## Discussion: _Signature / Vérification_ versus _pass-phrase_
#### Mode _pass-phrase_
L'application joint à sa requête un jeton contenant une _pass-phrase_, typiquement le strong hash d'une phrase secrète connue seulement de l'utilisateur:
- le service en calcule le SHA et le compare avec la valeur stockée pour valider ce droit d'accès.
- pour ce droit la _pass-phrase_ reçue par le service **est toujours la même**: le service _peut_ la mémoriser et la dérouter vers un hacker qui peut l'employer depuis une autre application (pirate) et faire croire au service que l'utilisateur est à l'origine de la phrase secrète dont le hash a été transmis.
- la phrase secrète de l'utilisateur est dans tous les cas inviolée.

> Il faut en conséquence avoir confiance dans les services sur le fait qu'ils ne mémorisent / déroutent pas les _pass-phrases_ reçues.

#### Mode _Signature / vérification_
L'application terminale _signe_ par la clé `S` un texte _challenge_ transmis dans la requête **et garanti différent à chaque fois**.
- l'application serveur utilise la clé `V` pour vérifier que le challenge reçu a bien été signé par la clé correspondante à la clé `V` qu'il détient.
- comme le _challenge_ est différent à chaque requête et ne peut pas être présenté deux fois, un service _indélicat_ ne pourrait que mémoriser des signatures _passées_ : même avec une mauvaise intention il ne pourrait rien transmettre d'utile à un hacker qui ne parviendra jamais à se faire passer pour l'utilisateur faute d'en connaître la clé `S`. 

Tout repose sur le caractère _inédit_ des challenges présentés, ce qu'on obtient en y mettant une estampille datée à la milliseconde et une durée de validité courte des tokens. En cas de paranoïa, il faut enregistrer dans une base ou un service de _share memory_ le dernier `time` utilisé par chaque `userId`.

**Le mode _pass-phrase_ a donc une confidentialité _dégradée_ par rapport au mode _signature / vérification_** et impose d'accorder sa confiance au service dans le fait qu'il ne mémorisera / déroutera pas les _jetons_ d'authentification.

#### Inviolabilité des clés _Signature / Vérification_
Ce protocole plus _sécuritaire_ repose toutefois sur le fait que seul l'utilisateur est en état de délivrer la clé S: 
- celle-ci fait environ 400 caractères aléatoires, il est impensable pour un humain standard de la connaître de mémoire et d'une manière ou d'une autre il va la stocker dans une _sorte de fichier_ externe.
- soit ce dernier réside sur un support physique amovible détenu physiquement par l'utilisateur: la sécurité repose sur la détention de ce support.
- soit il est _crypté_:
  - soit il n'est lisible que par quelqu'un donnant sa clé de cryptage, plus courte que 350 bytes et surtout basée sur un texte _qui fait sens_ pour l'utilisateur et qu'il peut connaître _de mémoire_.
  - soit la clé de cryptage résulte d'une caractéristique physique de l'utilisateur (empreinte, ...).

Le protocole **reporte** la sécurité globale un cran au-dessus, dans une application _tierce_ comme l'application **Safe** indépendante des applications terminales, capable de gérer la confidentialité des clés de signature. Encore faut-il avoir confiance ... dans l'application **Safe** (ce qui limite le nombre d'applications dans lesquelles on doit avoir _confiance_).

> L'utilisation d'un dépôt spécifique pour son Safe décale le problème: il faut avoir confiance dans le Web master ayant installé le site correspondant.

## Applications _légitimes / officielles_ versus _pirates_
Si une application est une application _pirate_ lancée par exemple depuis un lien envoyé par un e-mail frauduleux, 
- elle peut demander à l'utilisateur ses justificatifs de droits d'accès en _singeant_ l'application légitime. 
- elle peut envoyer ces clés usurpées à un serveur pirate où elles seront à disposition de pirates pour se faire délivrer des données par l'application serveur légitime.

Il n'existe aucun procédé logiciel _universel_ qui permette de connaître l'origine d'une application, de quelle _source_ elle vient, si elle est _légitime_ ou _pirate_. 

Depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle de l'application pirate,
- s'être assuré que le CDN correspondant distribue bien le source _officiel_ et non pas un _source modifié_. Pour cela il faut,
  - comparer le hash des fichiers sources distribués par le CDN avec les hash des fichiers source du repository _officiel_,
  - avoir obtenu d'un expert indépendant l'assurance que le code _officiel_ est bien légitime et n'a pas été patché pour redistribuer pas les clés d'accès,

> Ces conditions sont possibles à vérifier pour une _application_ Web, en revanche il n'existe aucun procédé technique permettant à un utilisateur de savoir si le logiciel d'un service est bien celui dont les sources (en Open Source) seraient disponibles dans un repository public: il faut _faire confiance_ à l'opérateur délivrant ce service.

## Le module _safe terminal_ et le service _safe générique_

Le module _safe terminal_ est embarqué dans les applications, comme _module utilitaire_.

Le service _safe_ est mis à disposition par un opérateur indépendant des applications et reçoit des requêtes émises depuis les modules _safe terminal_ embarqués dans les applications.

Après avoir lancé l'application _myApp1_ depuis son terminal, le module _safe terminal_ accède au _Master Directory_ qui lui indique pour l'alias fourni quel est l'opérateur gérant son _safe_.

L'URL d'accès au _Master Directory_ est une constante (publique) déployée avec l'application. 

# Sessions et _profils_ de sessions

Quand un utilisateur lance une application _myapp1_ depuis un _terminal_ il ouvre une session, identifiée de manière unique pour cette application: sur un _terminal donné_, une seule session peut s'exécuter à un instant donné pour l'application _myapp1_.

A l'ouverture d'une session de l'application _myapp1_ l'utilisateur dispose potentiellement de la liste de **tous** les droits acquis antérieurement pour cette application:
- chaque droit _peut_ être associé à une remontée importante de données des services associés à l'organisation qu'il cite. Si par exemple un utilisateur a 3 droits lui permettant _d'agir en tant qu'employé [Bob, Alice, Charles]_ il va récupérer les données correspondantes aux trois.
- mais en pratique l'utilisateur peut travailler la plupart du temps sur l'un de ceux-là (par exemple _agir en tant qu'employé [Bob]_). D'où l’intérêt pour lui de définir un profil _employé Bob_ ne reprenant QUE ce droit et conduisant à ne charger QUE les données de Bob.

Ainsi ultérieurement quand l'utilisateur voudra ré-ouvrir une session en tant _qu'employé Bob_, il désignera ce _profil_ dans la liste de ses profils enregistrés.

# _Préférences_ d'un utilisateur
Au cours d'une session d'une application _myapp1_, l'utilisateur peut fixer un certain nombre de _préférences_,
- la langue de travail,
- le mode clair ou foncé,
- des flags de présentation divers (portrait / paysage etc.),
- des options comme _mode expert sur la liste des randos_,
- des nombres de lignes d'affichages, etc.

Un _objet_ de préférence stocke ces paramètres (_settings_).

Un utilisateur peut enregistrer pour chaque application quelques jeux de préférences en leur donnant un code comme `mobile tablette PC simple expert ...`, chaque jeu étant adapté à la fois au profil technique du terminal et au mode de travail souhaité par l'utilisateur.

En ré-ouvrant une session de l'application _myapp1_ l'utilisateur peut de cette façon utiliser,
- soit le jeu des préférences par défaut,
- soit celui choisi dans la courte liste qui lui est présenté.

# Terminaux certifiés _de confiance_
Un utilisateur qui veut utiliser une application depuis un _terminal_ est placé devant deux cas de figure:
- **soit il n'a pas confiance dans ce _terminal_** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il a déjà utilisé ce même appareil antérieurement pour y retrouver un _cache_ déjà rempli de documents chargés la fois précédente et certainement pas des données facilitant son authentification.
- **soit il juge le terminal _de confiance_** et l'a _certifié_,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - les sessions qu'il y exécute peuvent laisser _en cache_ des informations cryptées et les applications peuvent espérer raisonnablement les retrouver plus tard.

Un utilisateur peut déclarer sa _confiance_ au _terminal_ qu'il utilise:
- son _coffre fort_ enregistre ce _terminal_ comme étant de confiance,
- le _terminal_ enregistre localement la référence à cette déclaration de confiance avec des éléments cryptographiques utilisables par ce seul utilisateur.

Lancer une application depuis un appareil _de confiance_ a plusieurs avantages:
- **authentification simplifiée** de l'utilisateur en donnant un code PIN court pour accéder au _coffre fort_ au lieu du couple plus long d'authentification _forte_ (alias et phrase secrète).
- **disposer sur ce terminal de _mémoires caches persistantes et cryptées de documents_** en **épinglant** un _profil_ de session sur le terminal ce qui lui permet d'ouvrir une session,
  - en mode _réseau_ en minimisant le nombre de documents à récupérer des serveurs,
  - en mode _avion_ (sans accès au réseau) avec accès en lecture aux documents dans l'état où ils se trouvaient lors de la dernière fin de cette session en mode _réseau_ sur ce _terminal_.

# Détail des sections d'un _objet safe_
La base de données du service _Safe_ enregistre les données de chaque _safe_ dans un _objet sérialisé_ accessible selon un format indépendant de la technologie de stockage utilisé par le service _safe_ ce qui permet une exportation / importation de son contenu entre opérateurs de _safe_.

Cet _objet_ a plusieurs sections:
- `auth` : la section réunissant les données cryptographiques requises à authentifier son propriétaire.
- `devices` : la section conservant les données cryptographiques permettant de certifier un terminal comme _de confiance_,
- `creds` : la section conservant les données cryptographiques et descriptives de chaque droit d'accès relatifs aux services pouvant être sollicités.
- `profiles` : la section enregistrant les _profils_ conservés pour chaque application.
- `prefs` : la section des _settings_ où l'utilisateur peut enregistrer les jeux de paramètres de préférence de comportement et d'affichage de ses applications favorites.
- `invits` : cette section conserve un _pointeur_ vers chaque demande d'invitation enregistrée par l'utilisateur, sachant que chacune a une durée de vie de quelques jours avant destruction (voir les _Invitations_ dans le document _Application_.)

## Section `auth`

A la création du _safe_ d'un utilisateur U **certaines propriétés invariantes** sont générées:
- `id` : identifiant unique généré aléatoirement.
- `K` : clé symétrique AES majeure de l'utilisateur.
- `C D` : couple de clés de cryptage / décryptage.
- `S V` : couple de clés de signature / validation.

Les propriétés suivantes sont fournies par U à la création mais U _peut_ les changer quand il le souhaite:
- `p1` : phrase secrète 1, d'au moins 24 signes.
- `p2` : phrase secrète 2, d'au moins 24 signes.
- `a1` : alias 1 d'au moins 10 signes, garanti unique par le _Master Directory_.
- `a2` : alias 2 d'au moins 10 signes, garanti unique par le _Master Directory_.

A un instant donné il existe toujours:
- au moins une des deux phrases `p1` ou `p2`,
- au moins un deux alias `a1` ou `a2`.

**Rappel:** pour une ID donnée, le _Master Directory_ détient une **copie** des données suivantes:
- `userId`
- `hshK`: le SHA raccourci du Strong Hash de la clé K.
- `hsha1`: le SHA raccourci du Strong Hash de l'alias 1 (s'il existe).
- `hsha2`: le SHA raccourci du Strong Hash de l'alias 2 (s'il existe).
- `C` et `V`: les clés publiques de cryptage et de vérification de U.
- `llq`: _last login quarter _. Numéro du trimestre de dernier login, 0 étant le premier de l'an 2000.
- `store`: le code du store où est stocké à l'instant actuel le _safe_ de U.

#### Problèmes de discordance potentielle entre _safe_ et _master directory_
Les deux DB étant distinctes, la mise à jour rigoureusement synchronisée des deux est impossible: l'une peut avoir été mise à jour et l'autre pas encore. En cas d'incident (certes improbable) entre ces deux étapes, les données sont discordantes. 

Le protocole suivant permet de rétablir la cohérence entre elles en cas de changement des alias par U.
- **Phase 1**: U change ses alias, son _safe_ détient pour ceux-ci deux groupes de _valeurs_
  - _Actuel_: les alias 1 et 2 cryptés par la clé K (`a1k a2k`) et le SHA raccourci de leur Strong Hash (`hsha1 hsha2`)
  - _Future_: idem pour les valeurs _futures_.
- **Phase 2**: le _safe_ envoie au _master directory_ `hsha1 hsha2` pour remplacer les valeurs détenues. Mais ceci _peut_ échouer si l'un des  nouveaux alias 1 ou 2 a déjà été réservé par un autre utilisateur en tant qu'alias 1 ou 2.
- **Phase 3-ok**: le _master directory_ a accepté `hsha1 hsha2`.
  - le _safe_ supprime _l'actuel_ et renomme le _futur_ en _actuel_.
  - _safe_ et _master directory_ sont cohérents entre eux.
- **Phase 3-ko**: le _master directory_ a refusé l'opération.
  - le _safe_ supprime le _futur_.
  - _safe_ et _master directory_ sont cohérents entre eux.
  - l'opération retourne un avis d'échec à U.

_**Inconsistances possibles:**_
- la phase 1 a eu lieu mais pas la 2.
- les phases 1 et 2 ont eu lieu mais pas la 3.

Un utilisateur qui utiliserait l'alias 1 pour obtenir une ID (au _login_ ou sur _invitation spontanée_) se trouverait face à un _safe indécis_ ayant deux valeurs possibles actuelle et future.
- ces opérations sur le _safe_ requiert qu'il sorte de l'indécision.
- Pour ce faire le _safe_ demande au _Master Directory_ de lui retourner son état les hash des alias qu'il détient.
  - Si _l'opérateur courant_ n'est pas celui du _safe_ demandeur, l'opération est refusée.
- le _safe_ sort de l'indécision en basculant soit sur _l'actuel_ soit sur le _futur_ en considérant que la _vérité_ est celle du _Master Directory_: l'utilisateur est informé de cette décision.

> Le _login_ et _l'invitation spontanée_ sont les seules opérations dont le point d'entrée est le _Master Directory_ (depuis un alias), puis ensuite le _safe_ par l'ID ainsi récupérée. 

#### Transfert du _safe_ de U de son opérateur actuel à un autre
U doit effectuer un _backup_ depuis son opérateur actuel, celui inscrit dans le _Master Directory_, et doit fournir l'une de ses phrases secrètes: le fichier correspondant est crypté par une phrase fournie par l'utilisateur.

U doit ensuite chez son _nouvel_ opérateur, **importer** ce fichier, après avoir prouvé qu'il en était le propriétaire en fournissant la phrase secrète présente dans le fichier:
- le _safe_ est créé / importé chez le nouvel opérateur,
- en cas de succès cet opérateur est inscrit comme _opérateur courant_ dans le _Master Directory_.

Enfin U doit supprimer son _safe_ chez son _ancien_ opérateur, sa phrase secrète autorisant cette opération.

Cette séquence n'empêche pas plusieurs _safe_ d'être existants et joignables depuis plusieurs opérateurs MAIS un login ne pourra s'effectuer QUE depuis un seul, le dernier où l'importation a été faite.

> L'utilisateur a de facto les moyens de corrompre lui-même ses _safes_: il en est informé à chaque étape et doit fournir sa phrase secrète ... ça devient alors un geste volontaire. Quoi qu'il en soit les _droits d'accès_ dans les applications restent inviolables dans les services applicatifs.

#### Contrôle de _safe courant_
Les opérations suivantes mettent en jeu à la fois le _safe_ et le _Master Directory_:
- **login**: l'entrée s'effectue depuis le _Master Directory_ par un alias. Le _safe_ adressé est obligatoirement celui qui y est inscrit comme _courant_, l'indécision éventuelle sur les alias est résolue.
- **invitation spontanée**: mêmes raisons.
- **changement d'alias**: l'opération part du _safe_. Elle ne peut aboutir QUE si ce _safe_ est bien celui indiqué comme _safe courant_ dans le _Master Directory_.
- **importation**: l'opération met à jour le _Master Directory_.

> Les autres opérations résolvent automatiquement une indécision quand le _safe_ est indécis et à cette occasion s'assure que le _safe_ est bien le courant (sinon se bloquent). En l'absence d'indécision, elles ne vérifient pas que le _safe_ depuis où elles opèrent est bien le courant.

#### Synthèse des propriétés de l'entête d'un _safe_
- `userId` : identifiant de l'utilisateur
- `llq` : _last login quarter_, trimestre du dernier login. Permet une _purge_ périodique des _safe_ obsolètes / fantômes.
- `lm` : _epoch_ en secondes de dernière mise à jour.
- `C` : clé de cryptage en clair (en base 64).
- `D` : clé de décryptage cryptée par la clé `K` (en base 64).
- `S` : clé de signature cryptée par la clé `K` (en base 64).
- `V` : clé de vérification en clair (en base 64).
- `hshK` : SHA du Strong Hash de la clé K.
- `admins` : liste des couples `SVC1.$OP1 / SVC2.$OP2 / ...` dont l'utilisateur a _déclaré_ être l'administrateur (cryptée par sa clé K et en base 64). La véracité de la _déclaration_ est vérifiée mais l'utilisateur peut se voir retiré cette qualité par l'opérateur sans que cette liste ne change.
- `pseudo` : dernier pseudo crypté par la clé K du _safe_ (en base 64) utilisé à la certification d'un terminal.

**Phrases secrètes:**
- `hshp1` : SHA raccourci du Strong Hash de la phrase 1 (en base 64).
- `K1` : clé K cryptée par le Strong Hash de la phrase 1.
- `hshp2 K2` : idem pour la phrase secrète 2.

**Alias**
- Deux sous-objets, `actual futur`. Sauf phase _indécise_ `futur` est `null` contenant les propriétés suivantes:
- `a1K`: alias 1 crypté par la clé K (en base 64).
- `hsha1`: SHA raccourci du Strong Hash de l'alias 1.
- `a2K hsha2`: idem avec l'alias 2.

#### Vérification selon les opérations
Une opération de mise à jour du service `SAFE` retourne le contenu du _safe_ à condition que les vérifications suivantes soient un succès.

**Login _fort_**
- login depuis un alias ET une phrase secrète.
- `ID` est donnée par le _Master Directory_ depuis l'alias.
- `shp1`: l'utilisateur ayant saisi la phrase 1 ou 2, le Strong Hash est calculé et passé en argument. Il doit correspondre, passé en SHA raccourci à `hshp1` ou `hshp2` inscrit dans le _safe_. 

**Login par PIN**
- voir la section correspondante dans _devices_.

**Invitation spontanée**
- un utilisateur A a saisi l'alias 1 (ou 2) et récupéré l'ID de U et l'opérateur gérant son _safe_. Il envoie à ce _safe_ une requête **Invitation spontanée** avec les arguments suivants:
  - `ID`
  - `sha1` : le Strong Hash de l'alias 1 (ou 2) est calculé par la session de A et transmis en argument de la requête au _safe_. 
- Le service de _safe_ peut ainsi vérifier que A a saisi un alias valide (et non pas simplement récupéré l'ID de U). L'alias n'étant pas contraint à être _long_, une attaque par force brute est _concevable_: le seul effet serait de pouvoir envoyer à U une proposition d'invitation spontanée / non sollicitée sans aucun risque pour la sécurité de U:
  - U devra la valider pour qu'elle prenne effet,
  - elle s'auto-détruira au bout de quelques jours.

**Autres opérations**
- elles n'ont pu être émises que depuis une session ayant reçu au login (_fort_ ou par PIN) le contenu du _Safe_, donc connaissant K. Les requêtes sont vérifiée depuis:
  - `ID`,
  - `shK`: la clé K étant décodée, son Strong Hash est calculé par la session et le service du _safe_ vérifie que son SHA raccourci est bien égal à `hshk` stocké dans le _safe_.

#### Affichage dans la session
L'utilisateur U authentifié peut afficher en clair dans sa session ses deux alias 1 et 2 (et les changer). 

> Ses phrases secrètes 1 et 2 ne sont pas lisibles mais il peut les changer.

Ces données ne sont JAMAIS disponibles en clair dans aucune base ni échangées sur le réseau,
- elles sont inattaquables par force brute en raison de leur cryptage par K (aléatoire longue) cryptée par leurs phrases secrètes longues.

Casser un alias reste difficile et coûteux, son Strong Hash exige un temps calcul important, la longueur minimal de l'alias n'est pas négligeable. C'est surtout inutile, la seule opération possible depuis un alias seul est de poster une **invitation spontanée** ne présentant aucun danger.

## Section `devices`
Chaque _terminal de confiance_ à une entrée  dans cette section est identifié par `devid` (un identifiant généré aléatoirement):
- `devName` : code / texte court **crypté par la clé K du _safe_** et encodé en base 64 donné par l'utilisateur pour qualifier le _terminal_ (par exemple `PC d'Alice`).
- `{ Va, cy, sign, nbe }` : propriétés permettant de certifier que ce _terminal_ est de confiance (voir plus loin).

Après avoir authentifié son accès à son _safe_, l'utilisateur peut retirer sa confiance à n'importe lequel des terminaux cités dans la liste en en supprimant l'entrée.

### Section `creds`
Cette section est un objet _map_,
- _clé_ : id de l'application,
- _valeur_: objet / map des droits:
  - _clé_: service + `.` + id du droit,
  - _valeur_: couple sérialisé [comment, data]
    - comment: commentaire **crypté par la clé K** et encodé en base 64.
    - data: contenu du droit sérialisé et **crypté par la clé K** et encodé en base 64.

### Section `profiles`
Cette section est un objet _map_,
- _clé_ : id de l'application,
- _valeur_: objet / map des droits:
  - _clé_: id du profil,
  - _valeur_: sérialisation de `{ profId, about, crIds }` encodé en base 64, où:
    - `about` : commentaire / à propos du profil crypté par la clé de l'utilisateur et encodé en base 64.
    - `crIds`: liste des ids des credentials (service + `.` + id du credential) inclus dans le profil.

### Section `prefs`
Cette section est un objet _map_,
- _clé_ : id de l'application,
- _valeur_: objet / map des droits:
  - _clé_: `code` de la préférence,
  - _valeur_: sérialisation de `[time, obj]` encodé en base 64, où:
    - `time` : _epoch_ de dernière mise à jour de la préférence_.
    - `obj`: objet de préférence `{ n1: v1, n2: v2 ... }` dont tous les `vi` sont des string ou entier (sur 32 bits).
  
### Section `invits`
C'est une map avec une entrée par _invitation_ (clé: `invitId`) conservée pendant quelques jours.

Cet objet a les propriétés suivantes:
- `invitId`: ID aléatoire générée à la création.
- `svc`: service concerné.
- `org`: organisation concernée.
- `time`: date-heure (_epoch_ en secondes) de création.
- `major`: code _majeur_ de l'objet de l'invitation.
- `minor`: code _mineur_ (facultatif) complémentation de l'objet de l'invitation.
- `status`: _déposée, acceptée, refusée, validée, déclinée, annulée_.
- `comment`: commentaire pour le seul usage de l'utilisateur U ayant fait la demande.

## Accès d'une application terminale à un _safe_
### Depuis n'importe quel _terminal_ (de confiance ou non)
Le module _safe terminal_ demande à l'utilisateur un alias et une phrase secrète:
- l'alias permet d'obtenir du _Master Directory_ l'ID de l'utilisateur et l'opérateur qui en gère le _safe_,
- l'accès à ce _safe_ permet de vérifier que la phrase secrète est bien l'une deux déclarées.

### Micro base locale IDB `safe` d'un terminal
Un terminal qui a été déclaré _de confiance_ par au moins un utilisateur a une micro base de données IDB nommée `safe` ayant les tables suivantes.

#### `header`
Cette table _singleton_ a deux colonnes:
- `devId`: un identifiant généré aléatoirement à la première déclaration de confiance faite sur ce terminal.
- `devName`: le _nom_ du _terminal_, par exemple `PC d'Alice`, saisi par le premier déclarant de confiance.

#### `trustings`
Chaque row est associé à UN _utilisateur_ ayant déclaré le _terminal_ de confiance:
- `userId`: identifiant de l'utilisateur (clé primaire).
- `pseudo`: par exemple `Bob`.
- `cx`: un challenge aléatoire (random de 24 bytes en base 64).
- `K1`: clé K du safe de l'utilisateur cryptée par le Strong Hash de la phrase secrète 1.
- `K2r`: idem pour phrase 2.
- `Kp`: clé K du safe de l'utilisateur cryptée par le Strong Hash de `PIN / cx / cy)` en base 64 où,
  - `PIN` est le code PIN fixé par l'utilisateur à la déclaration de confiance,
  - `cx cy` sont des _challenges_ générés aléatoirement à ce moment (des random de 24 bytes en base 64).

#### `tsessions`
Chaque row décrit une _session épinglée_:
- `app`: code l'application correspondante.
- `userId`: identifiant de l'utilisateur.
- `profId`: id du profil de la session ou `*` pour le profil par défaut contenant tous les droits.
- `about`: texte significatif pour l'utilisateur **crypté par la clé de l'utilisateur** et encodé en base 64 décrivant l'usage de sa session (par exemple `Revue des notes d'Alice et Jules`).
- `size`: `[s1, s2 ...]` volumes _utile_ des données de la base IDB lors de la dernière session ouverte sur ce _terminal_.
- `time`: dernière date-heure d'ouverture de cette session sur ce terminal.
- `prefCode`: code de la "préférence" utilisée la dernière fois.
- `prefTime`: _epoch_ date-heure de la dernière mise à jour de cette préférence.
- `prefObj`:  sérialisation (en binaire) de cet objet de "préférence" utilisé la dernière fois.

Il existe une base de données IDB de nom `app_x` où `x` est le hash court de `userId + '/' + profId`: elle contient les **documents en cache** de cette session.

#### Déclaration d'un _terminal_ de confiance
Depuis le _terminal_ à déclarer de confiance, l'utilisateur doit s'authentifier de manière _forte_: il dispose en mémoire du contenu du _safe_ et en particulier de sa clé K.

Dans sa déclaration de confiance il saisit:
- son `pseudo` et `devName` le nom qu'il donne à ce _terminal_: les valeurs par défaut sont proposées, par exemple `Bob` et `PC d'Alice`.
  - le _pseudo_ proposé est le dernier utilisé par U pour une certification antérieure.
- un code `PIN` (d'au moins 8 signes).

Le module _safe terminal_,
- génère aléatoirement `devId` si cette donnée ne figure pas encore dans le `header`.
- génère les challenges aléatoires `cx cy`.
- calcule `Kp`, cryptage par la clé `K` du Strong Hash de   `PIN / cx /  cy`.
- génère un couple `Sa Va` de clés de signature / vérification.
- calcule `sign`, signature par `Sa` du Strong Hash de `PIN / cx)`.
- enregistre dans la table `trustings` de la base IDB `Safe` un row avec les colonnes `userId pseudo cx K1 K2 Kp`.
- transmet au service _safe_ `userId, devId, shK, devName(crypté par K), Va, cy, sign` (où `shK` est le Strong Hash de K) qui,
  - accède au _safe_ dont l'id est `userId` et vérifie que `hshK` est bien le SHA raccourci de `shK` (s'assure que _safe terminal_ détient un _safe_ bien authentifié.
  - y créé dans la section `devices` une entrée `devId` avec les données `devName Va cy sign nbe = 0`.

> Remarque: `Sa` a servi à générer la signature `sign` mais n'est plus utilisé ensuite et n'est pas mémorisé alors que `Va` l'est et servira à authentifier la signature d'un PIN saisi par l'utilisateur.

Après ce calcul,
- le _safe_ a été mis à jour par le service _safe_ avec un nouveau terminal de confiance avec les données cryptographiques permettant à l'utilisateur de s'authentifier par un code PIN.
- sur le _terminal_ la base locale IDB _safe_ contient une entrée relative à ce _safe_ avec en particulier la clé K du _safe_ cryptée en `K1` `K2` et `Kp`. 

#### Authentification par code PIN depuis un _terminal déclaré de confiance_
Le module _safe terminal_ lit la base IDB _safe_ et, 
- propose à l'utilisateur de désigner la ligne de `trustings` dont la propriété `pseudo` (par exemple `Bob`) lui correspond. Le module dispose ainsi des données `userId cx Kp`.
- demande à l'utilisateur de saisir le PIN associé et calcule `z` comme Strong Hash de`PIN / cx`.
- transmet au service _safe_ `userId, devId, z` qui,
  - accède au _safe_ dont l'id est `userId`.
  - accède dans la section `devices` à l'entrée `devId` ce qui lui donne les propriétés `Va cy sign nbe`. Si cette entrée n'existe pas c'est que le _terminal_ N'EST PAS / PLUS de confiance pour ce _safe_,
    - soit n'a jamais été déclaré comme tel,
    - soit la confiance en lui a été retirée explicitement par l'utilisateur,
    - soit qu'il a été supprimé du fait d'un nombre excessif d'essai erroné de code PIN.
  - vérifie par `Va` que `sign` est bien la signature de `z`. En cas de succès, il met à 0 `nbe` s'il ne l'était pas déjà et sinon incrémente `nbe`.
  - retourne le challenge `cy` au module _safe terminal_ qui peut ainsi calculer la clé Strong Hash de `PIN / cx / cy` qui décrypte `Kp` ce qui lui donne la clé K du _safe_.

##### Échecs
SI la signature `sign` n'est pas vérifiée par `Va`, c'est que le code PIN n'est pas le bon. `Va` correspond au `Sa` qui a été utilisé à sa signature, `cx` était bien celui fixé à la déclaration. **Le nombre d'erreurs `nbe` est incrémenté**.

Si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais, l'entrée `devId` est supprimée. L'utilisateur devra refaire une _déclaration de confiance_ de ce terminal avec un code PIN ce qui exigera une authentification _forte_ de sa part.

### Accès d'une application en mode _avion_ (pas d'accès au réseau)
Depuis la table `tsessions` de la base IDB _Safe_ on liste les sessions qui ont été ouvertes sur ce _terminal_ pour cette application avec pour chacune,
- le texte `about` de son profil, par exemple `Revue des notes d'Alice et Jules`,
- le pseudo du _safe_ correspondant, par exemple `Bob`.

L'utilisateur désigne la session qu'il souhaite ré-ouvrir ce qui lui donne:
- le `userId` de cette session,
- le `profId` du profil de cette session,
- `K1 K2` la clé K de ce _safe_ mais cryptée par les phrases secrètes 1 et 2 d'authentification du _safe_.
- `prefCode prefObj` les valeurs de _préférences_ de la sessionK.
- le nom de la base IDB cache des documents.

L'utilisateur saisit sa phrase secrète 1 (ou 2) pour obtenir sa clé K depuis `K1` ou `K2`:
- le succès du décryptage authentifie sa propriété du _safe_.
- ses préférences d'ouverture de la session sont accessibles et sa base IDB est lisible.
- la session peut être ouverte, en lecture seulement.

> En mode _avion_ l'authentification par code PIN n'est pas possible.

### Sécurité de l'authentification par code PIN depuis un _terminal de confiance_
Sur un terminal NON déclaré de confiance l'utilisateur doit fournir un couple d'un alias (relativement court) et d'une phrase secrète longue, soit une bonne trentaine de signes ce qui est considéré comme inviolable par force brute avec un minimum de précaution dans le choix de la phrase secrète.

Sur un terminal déclaré de confiance par l'utilisateur il _suffit_ d'un code PIN de 8 signes (ou plus), donc _a priori_ beaucoup plus facile à craquer. Mais, 
- en l'absence de piratage technologique, l'utilisateur a droit à deux essais infructueux, le code PIN s'auto-détruisant passé ce seuil. Les essais multiples sont voués à l'échec.
- le code PIN est spécifique de l'utilisateur ET de chacun des terminaux qu'il a déclaré de confiance: il faut s'être connecté (par login _système_) sur un terminal déclaré de confiance préalablement pour pouvoir l'utiliser.

Ceci veut dire que la _vraie_ sécurité repose sur,
- la connaissance d'un compte de login du terminal qui est déjà sensé être particulièrement protégé,
- le fait que le terminal ait été préalablement déclaré de confiance et protégé par un code PIN,
- le fait qu'au delà du second essai infructueux, la confiance dans ce terminal est retirée.

Le **code PIN** n'est jamais stocké ni passé en clair sur le réseau au service _safe_: 
- il ne peut pas être détourné ou être lu depuis la base de données.
- il ne figure que temporairement en mémoire dans le module _safe terminal_ inclus dans l'application _myApp1 terminal_ durant la phase d'authentification du _safe_ de l'utilisateur.

Pour tenter depuis les données de la base de données du service _safe_ d'obtenir le code PIN par force brute, il faut effectuer une vérification de `sign` par `Va` avec le _challenge_ Strong Hash de `PIN / cx` mais `sign` est crypté par la clé privée de cryptage général du service _safe_.

Pour que cette dernière attaque pour trouver le PIN de `Bob` par force brute avec des chances de succès, il faut que le hacker ait obtenu frauduleusement:
- (1) le contenu en clair de l'objet _safe_ en base et pour cela il lui faut conjointement,
  - avoir accès à la base en lecture ce qui requiert, soit une complicité auprès du fournisseur de la base de donnée, soit **la complicité de l'administrateur technique**.
  - avoir la clé de décryptage des contenus de celle-ci inscrite dans la configuration de déploiement du service. Ceci suppose la **complicité de l'administrateur technique** effectuant ces déploiements.
- (2) ait obtenu le challenge `cx` stocké dans la base IDB _safe_ du terminal ce qui suppose,
  - d'avoir une session ouverte sur le terminal (mot de passe du login sur un PC, sur un mobile avoir le mobile _déverrouillé_).
  - d'ouvrir une application pour pouvoir lire en _debug_ la base de données IDB _safe_.

Ayant obtenu le challenge `cx`, il faut ensuite écrire une application dédiée pour craquer par force brute le code PIN en tentant la vérification de signature `sign` par la clé `Va` du challenge Strong Hash de `PIN / cx`.

> Ce double _piratage / complicité_ donne accès à la clé `K` du _safe_ de `Bob` et d'en lire le contenu (dont les clés des droits d'accès). Toutefois, les phrases secrètes restent inviolées puisque ne résidant que dans la mémoire de l'utilisateur, le hacker ne peut toujours pas effectuer un login _normal_, ni changer de ce fait les phrases secrètes, ni enregistrer quoi que soit dans le _safe_ piraté dont il a fini par obtenir la clé K.

> Cracker le code PIN d'un _terminal de confiance_ de l'utilisateur Bob ne compromet pas les autres utilisateurs.

> Pour craquer **tous** les codes PIN, il faudrait pouvoir accéder à tous les appareils de confiance **déverrouillés / sessions ouvertes** et casser par force brute le PIN de _chaque safe pour chaque terminal_. 

#### Durcir (un peu) le code PIN
Si le code PIN fait une douzaine de signes et qu'il évite les mots habituels des _dictionnaires_ il est quasi incassable dans des délais humains: pour être mnémotechnique il va certes s'appuyer sur des textes intelligibles, vers de poésie, paroles de chansons etc. Mais de nombreux styles de saisie mènent au code PIN depuis la phrase `allons enfants de la patrie`: avec ou sans séparateurs, des chiffres au milieu, des alternances de mots en minuscules / majuscules, un mot sur deux, etc. La seule _bonne intuition_ d'un texte est loin de donner le code PIN correspondant.

> Un _login_ des appareils un peu conséquent et un code PIN _un peu durci_ constituent en pratique une barrière **très coûteuse** à casser. Tant qu'à être un _délinquant_ une forte pression directe sur Bob permet en général de lui extorquer ses phrases / PIN à moindre coût 😈.

# Opérations d'un service _safe_
Elles sont implémentées dans le service _safe_ générique (en Javascript) mais aussi en PHP pour un service _safe_ spécifique utilisant un site Web et une base MySQL.

La signature des opérations est la même bien évidemment, un module _safe terminal_ ne sait d'ailleurs à quel service _safe_ il s'adresse (générique ou spécifique).

## Signature des opérations 
TODO

## Schéma de la base MySQL
TODO
