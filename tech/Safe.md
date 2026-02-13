---
layout: page
title: Utilisateurs et "coffres forts"
---

Un _utilisateur_ désigne ici une personne physique utilisant une application depuis un _terminal_.

Les _utilisateurs_ sont **anonymes** dans la mesure où rien, aucun identifiant, n'effectue une corrélation entre un utilisateur et la personne réelle dans la vraie vie.
- un utilisateur est identifié par un `userId` généré aléatoirement à son enregistrement en tant qu'utilisateur.

Tout utilisateur dispose d'un _coffre fort_ contenant principalement ses données confidentielles d'authentification et ses droits d'accès aux applications.
- l'identifiant d'un _coffre fort_ est le `userId` de son propriétaire.

Les coffres forts sont stockés dans des **dépôts** cryptés et sécurisés:
- un **dépôt générique**, où tout utilisateur peut ranger son coffre fort, à condition qu'il ait accordé sa confiance dans l'opérateur technique du service du dépôt générique pour avoir déployé le logiciel _open source_ en assurant la confidentialité et la sécurité.
- des **dépôts spécifiques** hébergés dans une base de données MySQL d'un site Web gérée par un script PHP standard. Tout utilisateur peut opter pour un dépôt spécifique, pour autant qu'il ait l'agrément de son administrateur.

> Les **dépôts spécifiques** existent pour les utilisateurs ou organisations ne souhaitant pas faire confiance au service de _dépôt générique_ et souhaitant avoir un contrôle total du logiciel gérant leurs _coffres forts_.

## Coffre fort _safe_
Cet enregistrement comporte plusieurs sections:
- une section d'authentification réunissant les données cryptographiques requise à authentifier son propriétaire.
- une section **terminaux de confiance** conservant les données cryptographiques permettant de considérer un terminal comme _de confiance_ et permmettant,
  - de s'authentifier par un code PIN plus léger que le double code _fort_ requis sur un terminal non déclaré de confiance.
  - d'y disposer de _caches de données_ cryptées sur le terminal pour pouvoir accélérer l'initialisation des applications et leur accès en **mode AVION** sans réseau.
- une section **droits d'accès** conservant les données cryptographiques et descriptives de chaque droit d'accès aux opérations des services pouvant être sollicités.
- une section **profils** permettant à l'utilisateur d'enregistrer des _profils_ chacun citant une liste des droits restrictives de ses droits d'accès en fonction d'un usage précis d'une application.
- une section **préférences** (_settings_) où l'utilisateur peut enregistrer le ou les jeux de paramètres de préférence de comportement et d'affichage de ses applications favorites.

Un coffre fort peut:
- être exporté d'un dépôt dans un fichier crypté par une clé saisie par l'utilisateur à des fins de sécurité ou de transfert dans un _dépôt_.
- être importé dans un dépôt depuis un fichier, sous réserve d'avoir prouvé en être le propriétaire et de non collision avec les coffres forts déjà enregistrés dans le dépôt.

La suite du document suppose l'usage du _dépôt générique_.

### Authentification _forte_ d'un utilisateur
Un utilisateur qui s'enregistre le fait en fournissant,
- un couple _pseudo / phrase secrète_ **principal**,
- un couple _pseudo / phrase secrète_ **secondaire**.

**Remarques:**
- le couple secondaire peut être employé par l'utilisateur en _secours_ quand il a _oublié_ le principal.
- si l'utilisateur a _oublié_ les deux, son coffre fort est définitivement perdu inaccessible mais aussi tous les éventuels _backups externes_ faits alors qu'il n'avait pas encore oublié ses codes.
- les pseudos comme les phrases secrètes ne sont connus en clair QUE de l'utilisateur: ils ne sont pas stockés et les retrouver par _force brute_ par un pirate est impossible, du moins pour une phrase assez longue et respectant quelques règles simples.
- l'utilisateur peut choisir sans aucun risque un pseudo simple qui lui est familier comme un numéro de mobile, une adresse e-mail, son prénom et nom, etc. Il n'y a que lui qui en aura connaissance.
- l'utilisateur peut changer ses codes d'accès, principal et secondaire, à condition d'en connaître un des deux.
- l'utilisateur n'a aucune raison de confier à qui que se soit son _pseudo principal_.

### Transmission d'un droit d'accès
Depuis une application, un utilisateur B peut **transmettre un droit d'accès** à un utilisateur A en utilisant le pseudo _secondaire de A_, que ce dernier lui a obligeamment transmis:
- rien n'empêche A de changer aussitôt après son _pseudo secondaire_ afin de ne pas recevoir d'autres droits de la part d'autres utilisateurs.
- pour A recevoir un droit _non sollicité_ est par lui-même un acte sans risque:
  - il n'est pas obligé de s'en servir,
  - il peut le supprimer à sa guise de sa liste des droits,
  - ça donne à A (le récepteur) des possibilités supplémentaires mais n'en donne aucune à B (le transmetteur).
- c'est un des procédés permettant de recevoir des _invitations_.

# Gestion des _credentials / droits d'accès_ dans les applications

Une application `myApp1` met en jeu deux couches de logiciels:
- `myApp1` l'application terminale s'exécutant typiquement dans un browser Web et dont le source est lisible et délivré par un serveur statique / CDN.
- `svc1 svc2 ...` des services s'exécutant dans le _cloud_ en exécutant des opérations de mise à jour et d'extractions / synchronisations de données.

Les opérations d'un service exécutées dans le _cloud_ comme les données qu'il peut synchroniser avec l'application qui y est abonnées, sont soumises à des **droits d'accès**: d'une manière ou d'une autre l'utilisateur derrière l'application terminale doit prouver qu'il possède effectivement les _droits requis_ pour solliciter une opération afin de mettre à jour et / ou obtenir des données centrales. Par exemple:
- `cpt` : droit à lire les documents d'un compte,
- `mbr` : droit de gestion des accès des membres d'un groupe,
- `trf` : droit de modification tarifaire ...

## Vérification des _droits d'accès_ par _jetons signés_
### Droit d'accès / _credential_
Un _droit d'accès_ est matérialisé par les données suivantes:
- `appId` : Un droit est spécifique d'UNE application.
- `org` : un droit est spécifique d'UNE organisation. Par exception, le code `*` indique un droit _d'administration technique_ du service.
- `type`: ce code détermine le traitement à appliquer pour enregistrer / valider un droit d'accès (techniquement assuré par une classe): par exemple `cpt, mbr, trf ...`.
- `scope`: un set de couples clés / valeurs (string) qui permettent au serveur de déterminer à quelles entités et pour quelles actions le droit s'applique. Exemples:
  - _empId_ : identifiant d'un employé.
  - _storeId_ : identifiant d'un magasin.
  - _role_ : role / habilitations de l'employé à exercer des actions dans le cadre de ce magasin.

> Un droit d'accès est immuable, ne se met pas à jour.
> - son identifiant `crId` est le hash de `[type, c1, v1, c2, v2 ...]`, les clés `ci` étant prises dans l'ordre lexicographique du `scope`. Le serveur peut vérifier la sincérité de l'`id` d'un droit depuis son contenu.

#### Stockage des droits d'accès dans le _safe_
Les droits sont stockés dans le **coffre fort / safe** de l'utilisateur détenteur (regroupés par application). Chaque droit y est mémorisé, accessible par son `id` avec les propriétés suivantes:
- `about` : un commentaire permettant à l'utilisateur de comprendre la portée / usage du droit: ce commentaire peut être mis à jour.
- `org type scope` : la définition du droit.
- `sign` : _clé privée de signature_ (environ 130 bytes), ou _pass-phrase_. Dans le cas d'une _pass-phrase_ c'est une suite de 32 bytes,
  - soit ayant été générée plus ou moins aléatoirement (mémorisation impossible).
  - soit correspondant au _strong hash_ d'une phrase longue humainement intelligible (capable d'être mémorisé).

> Pour un droit mémorisé dans le safe d'un utilisateur, seul `about` peut être mis à jour, les autres propriétés sont immuables.

A la création d'un droit d'accès un couple de clés de signature / vérification est généré:
- `sign`: la clé _privée_ de signature est stockée avec le droit d'accès dans le _safe_ de l'utilisateur.
- `verif`: la clé _publique_ de vérification est stocké dans la base de données du service. Un service ne reçoit **jamais** les clés privées de signature.

#### Stockage en base de données du serveur des droits d'accès validés par le serveur
Quand une opération d'un service _valide_ un droit d'accès il le mémorise dans un document `CREDENTIAL`:
- de clé primaire `id crId` ou `id` est l'identifiant de l'utilisateur pour cette application / organisation.
- les autres propriétés étant:
  - `type scope` : le descriptif du droit d'accès.
  - `verif`:
    - si le droit d'accès a une clé de signature, `verif` en est la clé de vérification.
    - si c'est une pass-phrase, c'est le SHA de la pass-phrase du droit.

> Ce document étant immuable, les services peuvent disposer d'un cache en mémoire évitant d'accéder à la base quand il a été récemment référencé.

Le _cache_ en mémoire des serveurs des droits d'accès conserve le `time` de sa dernière vérification. Toute requête de validation d'un droit est accompagnée d'une date-heure,
- qui ne doit pas être trop antérieure à la date-heure courante,
- qui doit être strictement supérieure au `time` en mémoire cache pour ce `id / crId`. 

> Toute requête doit en conséquence être accompagnée d'une date-heure récente et toujours en progression.

#### Opérations de _validation_ d'un droit d'accès par le serveur
Un utilisateur peut toujours générer autant de droits d'accès qu'il le souhaite mais seuls ceux ayant été **validés / enregistrés** par le service sont utilisables.
Pour cela l'utilisateur émet une requête de _validation_ qui a pour objectif de faire enregistrer le droit d'accès dans un document CREDENTIAL correspondant aux droits _validés_.

La requête transmet:
- `id`: l'identifiant de l'utilisateur vis à vis du service.
- `crId type scope` : le descriptif du droit (`crId` peut être recalculé par le serveur - ou ne pas être transmis).
- `verif`: la clé de vérification ou le SHA de la pass-phrase.
- **d'autres arguments** nécessaires à l'opération pour s'assurer que ce droit est _valide_. Par exemple, un code d'invitation, des justificatifs divers ...

L'opération s'assure qu'en fonction de ces données le droit d'accès peut être accepté:
- si c'est le cas, le droit d'accès est enregistré dans un document CREDENTIAL.
- sinon l'opération retourne une erreur, le droit n'est pas validé (et l'application terminale demanderesse le fait effacer du _safe_ de l'utilisateur).

#### Vérification des droits d'accès lors d'une requête quelconque
Toute requête peut référencer un ou plusieurs droits d'accès prouvant que l'utilisateur est fondé à exécuter l'opération correspondante.
- `id`: son identifiant pour l'application / organisation.
- `time` : la date-heure de la requête, en progression par rapport à la précédente émise.
- une liste de couples `[crId, s]` où chaque couple donne:
  - _crId_: l'identifiant du droit d'accès.
  - _s_: la _signature_ par la clé de signature du droit par l'application terminale du couple `id time`. Dans le cas d'une _pass-phrase_ _s_ est la pass-phrase elle-même.

Le service est alors en mesure pour chaque droit cité,
- d'accéder au document CREDENTIAL correspondant (très souvent _en cache_),
- vérifier la signature s du _challenge_ `id time` (ou la correspondance du SHA de la pass_phrase mémorisée).
- d'enregistrer dans le contexte de la requête le `type scope` du droit qui permettra au traitement de savoir ce qu'il peut ou non faire et sur quoi. 

#### Cas d'un droit de nature _Signature / Vérification_
Les services des applications ne détiennent des _droits d'accès_ que la clé `V` mais n'ont PAS accès à le clé `S` correspondante. Un _service_ vérifie la validité d'un droit transmis par _l'application terminale_ de la manière suivante:
- l'application terminale génère un texte _challenge_ qui n'a jamais été généré et ne sera jamais plus présenté au service.
- elle _signe_ ce _challenge_ par sa clé `S` et transmet au service le couple du challenge et de sa signature.
- le service utilise sa clé `V` correspondante à la clé `S` utilisée à la signature et peut vérifier que la signature reçue est bien celle du challenge transmis.

> Cette technique permet au service de s'assurer de la validité d'un droit sans avoir eu en mémoire la clé `S` de signature: c'est un avantage de confiance par rapport aux solutions basées sur une pass-phrase qui, à un moment ou à un autre, a besoin d'être présent dans la mémoire du serveur: certes si celle-ci a été obtenu par un hachage fort (type PBKDF) d'un mot de passe long ceci limite le risque.

> En cas de soumissions de nombreuses requêtes d'une application depuis un device requérant les mêmes droits, leur _validation_ ne requiert qu'un calcul en mémoire sans accès à la base pour obtenir les clés de vérification.

### Différences de confidentialité entre _Signature / Vérification_ et _pass-phrase_
#### En mode _pass-phrase_
L'application joint à sa requête un jeton contenant la _pass-phrase_ (typiquement le strong hash d'une phrase secrète connue seulement de l'utilisateur):
- le service en calcule le SHA et le compare avec la valeur stockée pour valider ce droit d'accès.
- pour ce droit la _pass-phrase_ reçue par le serveur **est toujours la même**: le serveur _peut_ la mémoriser et la dérouter vers un hacker qui peut l'employer depuis une autre application (pirate) et faire croire au service que l'utilisateur est à l'origine de la phrase secrète dont le hash a été transmis.
- la phrase secrète de l'utilisateur est dans tous les cas inviolée.

> Il faut en conséquence avoir confiance dans l'application terminale et le fait qu'elle ne mémorise / déroute pas les _pass-phrases_ reçues.

#### En mode Signature / vérification
L'application terminale _signe_ par la clé `S` un texte _challenge_ transmis dans la requête **et garanti différent à chaque fois**.
- l'application serveur utilise la clé `V` pour vérifier que le challenge reçu a bien été signé par la clé correspondante à la clé `V` qu'il détient.
- comme le _challenge_ est différent à chaque requête et ne peut pas être présenté deux fois, un service _indélicat_ ne pourrait que mémoriser les signatures _passées_ : même avec une mauvaise intention il ne pourrait rien transmettre d'utile à un hacker qui ne parviendra jamais à se faire passer pour l'utilisateur faute d'en connaître la clé `S`.

**Le mode _pass-phrase_ a donc une confidentialité _dégradée_ par rapport au mode _signature / vérification_** et impose d'accorder sa confiance au service dans le fait qu'il ne mémorisera / déroutera pas les _jetons_ d'authentification.

#### Inviolabilité des clés _Signature / Vérification_
Ce protocole comme plus _sécuritaire_ repose toutefois sur le fait que seul l'utilisateur est en état de délivrer la clé S: 
- celle-ci fait environ 350 bytes aléatoires, il est impensable pour un humain standard de la connaître de mémoire et d'une manière ou d'une autre il va la stocker dans une _sorte de fichier_ externe.
- soit ce dernier réside sur un support physique amovible détenu physiquement par l'utilisateur: la sécurité repose sur la détention de ce support.
- soit il est _crypté_:
  - soit il n'est lisible que par quelqu'un donnant sa clé de cryptage, plus courte que 350 bytes et surtout basée sur un texte _qui fait sens_ pour l'utilisateur et qu'il peut connaître _de mémoire_.
  - soit la clé de cryptage résulte d'une caractéristique physique de l'utilisateur (empreinte, ...).

Le protocole **reporte** la sécurité globale un cran au-dessus, dans une application _tierce_ comme l'application **Safe** indépendante des applications terminales, capable de gérer la confidentialité des clés de signature. Encore faut-il avoir confiance ... dans l'application **Safe** (ce qui limite le nombre d'applications dans lesquelles on doit avoir _confiance_).

### "Un" droit, "plusieurs" clés
Le serveur _peut_ mémoriser pour _un_ droit non pas _une_ clé `V` mais _une liste de clés_ `V1 V2 ...`: pour être validé, un droit d'un jeton d'accès doit fournir une signature qui a été établie par la clé `Si` correspondante à l'une des `[V1 V2 ...]`. 

Gérer un _changement de clé_ pour un droit attribué trop généreusement ou pour un temps limité s'effectue en créant une nouvelle clé dont la distribution est restreinte aux seuls détenteurs souhaitables dans le futur. Pendant un certain temps les deux peuvent être admises, puis _l'ancienne_ supprimée.

## Applications _légitimes / officielles_ versus _pirates_
Si une application est une application _pirate_ lancée par exemple depuis un lien envoyé par un e-mail frauduleux, 
- elle peut demander à l'utilisateur ses justificatifs de droits d'accès en _singeant_ l'application légitime. 
- elle peut envoyer ces clés usurpées à un serveur pirate où elles seront à disposition de pirates pour se faire délivrer des données par l'application serveur légitime.

Il n'existe aucun procédé logiciel _universel_ qui permette de connaître l'origine d'une application, de quelle _source_ elle vient, si elle est _légitime_ ou _pirate_. Toutefois depuis un browser _sain_, l'appel d'une URL en HTTPS reste le procédé le _plus fiable_, encore faut-il,
- lui avoir transmis la bonne URL et non celle de l'application pirate,
- s'être assuré que le CDN correspondant distribue bien le source _officiel_ et non pas un _source modifié_. Pour cela il faut,
  - comparer le hash des fichiers sources distribués par le CDN avec les hash des fichiers source du repository _officiel_,
  - avoir obtenu d'un expert indépendant l'assurance que le code _officiel_ est bien légitime et ne redistribue pas de clés d'accès,

> Ces conditions sont possibles à vérifier pour une _application_ Web, en revanche il n'existe aucun procédé technique permettant à un utilisateur de savoir si le logiciel d'un service est bien celui dont les sources (en Open Source) seraient disponibles dans un repository public: il faut _faire confiance_ à l'opérateur délivrant ce service.

# Le module _safe terminal_ et le service _safe générique_

Le module _safe terminal_ est embarqué dans les applications, comme _module utilitaire_.

Le service _safe générique_ est mis à disposition par un opérateur indépendant des applications et reçoit des requêtes émises par le module _safe terminal_ embarqué dans les applications.

Ils ont pour objet de gérer le _coffre fort_ des utilisateurs.

Après avoir lancé l'application _myApp1_ depuis son terminal, un utilisateur va lui indiquer quel est son _coffre fort_ afin d'accéder en toute sécurité aux données confidentielles qui le concerne.
- soit localisé dans le _dépôt générique_ qui a une URL publique,
- soit localisé dans un _dépôt spécifique_ dont l'URL est celle du site Web le gérant: l'utilisateur devra en conséquence citer cette URL:
  - il peut utiliser une variante de l'application dont la configuration a enregistré celle-ci à la place de celle du _dépôt générique_.
  - il peut utiliser un _raccourci_ dont l'URL est semblable à celle-ci:
  
    https://lesbellesapps.github.io/myapp1?https%3A%2F%2Fmodepot.truc.com%3A8087%2Fsafe.php
    la partie après ? correspond à l'URL du dépôt:
    https//modepot.truc.com:8087/safe.php


### Sessions et _profils_ de sessions
Quand un utilisateur lance une application _myapp1_ depuis un _device_ il ouvre une session, identifiée de manière unique pour cette application: sur un _device donné_, une seule session peut s'exécuter à un instant donné pour l'application _myapp1_.

A l'ouverture d'une session de l'application _myapp1_ l'utilisateur dispose potentiellement de la liste de tous les droits déjà acquis antérieurement pour cette application:
- chaque droit _peut_ être associé à une remontée importante de données du serveur associé à l'organisation qu'il cite.
- afin d'éviter une surcharge inutile pour le travail qu'il souhaite engager, l'utilisateur peut définir un _profil_ de session: 
  - dans la liste de tous les droits il _coche_ ceux qui l'intéresse,
  - il donne un _nom / commentaire_ à ce profil qui va être enregistré dans son _coffre fort_ comme `accès à mes randos lointaines`.

Ainsi ultérieurement quand l'utilisateur voudra ré-ouvrir une session disposant de ces seuls droits, il désignera ce _profil_ dans la liste de ses profils enregistrés.

### _Préférences_ d'un utilisateur
Au cours d'une session d'une application _myapp1_, l'utilisateur peut fixer un certain nombre de _préférences_,
- la langue de travail,
- le mode clair ou foncé,
- des flags de présentation divers (portrait / paysage),
- des options comme _mode expert sur la liste des randos_,
- des nombres de lignes d'affichages, etc.

Un _objet_ de préférence stocke ces paramètres.

Un utilisateur peut enregistrer pour chaque application quelques jeux de préférences en leur donnant un code comme `mobile tablette PC simple expert ...`, chaque jeu étant adapté à la fois au profil technique du terminal et au mode de travail souhaité par l'utilisateur.

En ré-ouvrant une session de l'application _myapp1_ l'utilisateur peut de cette façon utiliser,
- soit le des préférences par défaut,
- soit le jeu de préférences choisi dans la courte liste qui lui est présenté.

### Terminaux _de confiance_
Un utilisateur qui veut utiliser une application depuis un _terminal_ est placé devant deux cas de figure:
- **soit il n'a pas confiance dans ce _terminal_** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver un _cache_ déjà rempli de documents chargés la fois précédente et certainement pas des données facilitant son authentification.
- **soit il juge le terminal _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - les sessions qu'il y exécute peuvent laisser _en cache_ des informations cryptées et espérer raisonnablement les retrouver plus tard.

Un utilisateur peut déclarer sa _confiance_ au _terminal_ qu'il utilise:
- son _coffre fort_ enregistre ce _device_ comme étant de confiance,
- le _terminal_ enregistre localement la référence à cette déclaration de confiance avec des éléments cryptographiques utilisables par ce seul utilisateur.

Lancer une application depuis un appareil _de confiance_ a plusieurs avantages:
- **authentification simplifiée** de l'utilisateur en donnant un code PIN court pour accéder à _coffre fort_ ai lieu du couple plus long d'authentification _forte_ (pseudo et phrase secrète).
- **disposer sur ce terminal de _mémoires caches persistantes et cryptées de documents_** pour chaque _profil_ de session ce qui lui permet d'ouvrir une session,
  - en mode _réseau_ en minimisant le nombre de documents à récupérer des serveurs,
  - en mode _avion_ (sans accès au réseau) avec accès en lecture aux documents dans l'état où ils se trouvaient lors de la dernière fin de session en mode _réseau_ sur ce _device_.

## Sections des _objet coffre fort_
La base de données du service _Safe_ enregistre les données de chaque _safe_ dans un _objet sérialisé_ de format indépendant de la technologie utilisé par le service _safe_ ce qui permet une exportation / importation de son contenu entre dépôts générique et spécifiques.

Cet _objet_ a plusieurs sections:
- `auth` : la section réunissant les données cryptographiques requise à authentifier son propriétaire.
- `devices` : la section conservant les données cryptographiques permettant de considérer un terminal comme _de confiance_,
- `creds` : la section conservant les données cryptographiques et descriptives de chaque droit d'accès aux opérations des services pouvant être sollicités.
- `profiles` : la section enregistrant les _profils_ conservés pour chaque application.
- `prefs` : la section des _settings_ où l'utilisateur peut enregistrer les jeux de paramètres de préférence de comportement et d'affichage de ses applications favorites.

### Section `auth`

#### Création d'un _safe_ d'un utilisateur
L'identifiant userId pour représenter l'utilisateur est généré aléatoirement `shaS(random(32))`.

Une clé AES `K` de 32 bytes est tirée aléatoirement: elle ne pourra pas changer et est la clé de cryptage du _safe_.

Un couple de clés `C` (cryptage - publique) / `D` (décryptage - privée).
- la clé `C` (un PEM) est stockée en clair (elle est _publique_).
- la clé `D` (un PEM) est stockée cryptée par la clé K et encodée en base 64.

Un couple de clés `S` (signature - publique) / `D` (vérification - privée).
- la clé `S` (un PEM) est stockée en clair (elle est _publique_).
- la clé `V` (un PEM) est stockée cryptée par la clé K et encodée en base 64.

L'utilisateur donne:
- un _couple_ `p0, p1` (qui pourra être changé) _primaire_:
  - `p0` est un pseudo / prénom-nom / adresse mail / numéro de téléphone / etc. qui identifie de manière unique le _safe_ (`hp0` le SH de `p0` est un index unique).
  - `p1` est une phrase secrète _longue_ d'au moins 24 signes.
  - `hhp1` est le SHA court de `SH(p1)`.
- un _couple_ `r0, r1` (qui pourra être changé) _secondaire (récupération)_:
  - `r0` est un pseudo / prénom-nom / adresse mail / numéro de téléphone / etc. (12 signes au moins) qui identifie de manière unique le _safe_ (`hr0` le SH de `r0` est un index unique) et qui peut être égal à `p0`.
  - `r1` est une phrase secrète _longue_ d'au moins 24 signes. Il n'est pas judicieux qu'elle soit égale à `p1` puisqu'elle permet justement la récupération du safe en cas d'oubli de `r0, p0`.
  - `hhr1` est le SHA court de `SH(r1)`.
- `pseudo`: un nom court compréhensible par les propriétaires des _terminaux_ de confiance, par exemple `Bob`, crypté par la clé K et encodé en base 64.

La clé `K` du safe est stockée,
- dans `Ka` et `Kr` cryptages respectifs par  `SH(p0, p1)` et `SH(r0, r1)` et encodés en base 64.
- `hhk` : SHA court du `SH(K)` permettant au service _safe_ de vérifier sur chaque opération demandée par un module _safe terminal_ que celui-ci a bien authentifié l'utilisateur et en détient la clé K.

A aucun moment les propriétés `p0 p1 r0 r1` ne sont ni stockées ni transmises _en clair_: elles ne sont _lisibles_ que très temporairement lors la saisie par l'utilisateur dans le module _safe terminal_ et cryptées dès la fin de la saisie.

Pour changer `p0, p1` et/ou `r0, r1` l'utilisateur doit fournir,
- soit le couple actuel `p0, p1` OU `r0, r1`.
- les nouveaux couples `p0, p1` et `r0, r1`. 

#### Synthèse des propriétés de la section `auth`
- `id` : identifiant de l'utilisateur
- `lam` : dernier mois d'accès YYYYMM au _safe_: toute utilisation recule cette date qui permet une _purge_ périodique des _safe_ obsolètes / fantômes.
- `lm` : _epoch_ en secondes de dernière mise à jour.
- `C` : clé de cryptage en clair.
- `D` : clé de décryptage cryptée par la clé `K` et mise en base 64.
- `S` : clé de signature cryptée par la clé `K` et mise en base 64.
- `V` : clé de vérification en clair.
- `hp0` : index unique, `SH(p0)` en base 64.
- `hr0` : index unique, `SH(r0)` en base 64.
- `hhp1` : SHA court de `SH(p1)`.
- `hhr1` : SHA court de `SH(r1)`.
- `hhk` : SHA court de `SH(K)`.
- `Ka` : clé `K` du safe cryptée par `SH(p0, p1)` en base 64.
- `Kr` : clé `K` du safe cryptée par `SH(r0, r1)` en base 64.
- `pseudo` : pseudo crypté par la clé K du _safe_ et mis en base 64.

### Section `devices`
Chaque _device de confiance_ à une entrée  dans cette section identifiée par `devid` (un identifiant généré aléatoirement):
- `about` : code / texte court **crypté par la clé K du _safe_** et encodé en base 64 donné par l'utilisateur pour qualifier le _device_ (par exemple `PC d'Alice`).
- `{ Va, cy, sign, nbe }` : propriétés permettant de valider que ce _device_ est de confiance (voir plus loin).

Après avoir authentifié son accès à son _safe_, l'utilisateur peut retirer sa confiance à n'importe lequel des devices cités dans la liste en en supprimant l'entrée.

### Section `creds`
Cette section est un objet _map_,
- _clé_ : id de l'application,
- _valeur_: objet / map des droits:
  - **variante 1 (normale)**
    - _clé_: id du droit,
    - _valeur_: contenu du droit sérialisé et **crypté par la clé K de l'utilisateur** et encodé en base 64.
  - **variante 2 (droit transmis)**
    - _clé_: `$` + id du droit,
    - _valeur_: sérialisation du couple `[crobj, pubCT]`
      - `pubC` : clé publique de cryptage de l'utilisateur ayant transmis le droit.
      - `crobj` : contenu du droit sérialisé par la clé `AES` suivante et encodé en base 64.
        - la clé `AES` est obtenu depuis le couple `[pubCT, privDR]` ou `pubCT` est la clé publique de cryptage du transmetteur et `privDR` la clé privée de décryptage de l'utilisateur récepteur.
        - le transmetteur a obtenu cette même clé `AES` depuis `[pubCR, privDT]` où `pubCR` est la clé publique de cryptage du récepteur et `privDT` sa clé privée de décryptage. Le transmetteur a obtenu la clé pubCR depuis l'application ou depuis le dépôt _safe_ en fournissant le pseudo secondaire du destinataire.

Un _droit transmis_ est décrypté à première utilisation par son destinataire qui le réinsère après cryptage par sa clé K comme droit _normal_: cette procédure authentifie bien mutuellement transmetteur et destinataire sans jamais exposer leurs propres propriétés cryptographiques privées.

> Une application qui prévoit de _transmettre un droit_ entre utilisateurs doit être en mesure de disposer de leurs clé publiques, typiquement depuis leur _pseudos secondaires.

**L'identifiant** est le hash _court_ de ses propriétés `[org, type, n1, v1, n2, v2 ...]`:
- les couples `ni / vi` sont les propriétés de l'objet scope du droit;
- la sérialisation les prend dans l'ordre lexicographique de leurs nom `ni`.

La sérialisation (décryptée) d'un objet _credential_ permet de construire l'objet correspondant avec les propriétés suivantes:
- `id`: re-calculable depuis `org type scope`.
- `about`: commentaire / à propos du credential. Seule propriété pouvant être mise à jour après création du credential.
- `org`: organisation (ou '*' exceptionnellement)
- `type`:  type de credential
- `scope`: scope fonctionnel `{ n1: v1, n2: v2 ... }` dont les valeurs `vi` ne sont QUE des strings.
- `sign`: clé de signature (ou pass-phrase) encodée en base 64.

### Section `profiles`
Cette section est un objet _map_,
- _clé_ : id de l'application,
- _valeur_: objet / map des droits:
  - _clé_: id du profil,
  - _valeur_: sérialisation de `{ profId, about, crIds }` encodé en base 64, où:
    - `about` : commentaire / à propos du profil crypté par la clé de l'utilisateur et encodé en base 64.
    - `crIds`: liste des ids des credentials inclus dans le profil.

### Section `prefs`
Cette section est un objet _map_,
- _clé_ : id de l'application,
- _valeur_: objet / map des droits:
  - _clé_: `code` de la préférence,
  - _valeur_: sérialisation de `[time, obj]` encodé en base 64, où:
    - `time` : _epoch_ de dernière mise à jour de la préférence_.
    - `obj`: objet de préférence { n1: v1, n2: v2 ... } dont tous les `vi` sont des string ou entier (sur 32 bits).

## Accès d'une application terminale à un _safe_
### Depuis n'importe quel _device_ (de confiance ou non)
Le module _safe terminal_ demande à l'utilisateur `p0 p1` (ou `r0, r1`) et transmet `SH(p0) SH(p1)` au module _safe server_ qui,
- accède au document _safe_ depuis le `SH(p0)` (index unique).
- vérifie que `hhp1` est bien le SHA court de `SH(p1)` reçu en argument.
- retourne `Ka Kr`: le module _safe terminal_ décodant `Ka` (ou `Kr` selon le cas) par `SH(p0, p1)` (ou `SH(r0, r1)`). En cas d'échec c'est que `p0 / p1` (ou `r0 / r1` était incorrect).

### Micro base locale IDB `safe` d'un terminal
Un device qui a été déclaré _de confiance_ par au moins un utilisateur a une micro base de données IDB nommée `safe` ayant les tables suivantes.

#### `header`
Cette table _singleton_ a deux colonnes:
- `devId`: un identifiant généré aléatoirement à la première déclaration de confiance faite sur ce terminal.
- `devName`: le _nom_ du _device_, par exemple `PC d'Alice`, saisi par le premier déclarant de confiance.

#### `trustings`
Chaque row est associé à UN _utilisateur_ ayant déclaré le _device_ de confiance:
- `userId`: identifiant de l'utilisateur (clé primaire).
- `pseudo`: par exemple `Bob`.
- `cx`: un challenge aléatoire (random de 24 bytes en base 64).
- `Ka`: clé K du safe de l'utilisateur cryptée par `SH(p0, p1)` où `p0` et `p1` sont les termes d'authentification du safe de l'utilisateur (en base 64).
- `Kr`: clé K du safe de l'utilisateur cryptée par `SH(r0, r)` (en base 64).
- `Kp`: clé K du safe de l'utilisateur cryptée par `SH(PIN + cx, cy)` en base 64 où,
  - `PIN` est le code PIN fixé par l'utilisateur à la déclaration de confiance,
  - `cx cy` sont des _challenges_ générés aléatoirement à ce moment (des random de 24 bytes en base 64).

#### `tsessions`
Chaque row décrit une _session épinglée_:
- `app`: code l'application correspondante.
- `userId`: identifiant de l'utilisateur.
- `profId`: id du profil de la session ou * pour le profil par défaut contenant tous les droits.
- `about`: texte significatif pour l'utilisateur **crypté par la clé de l'utilisateur** et encodé en base 64 décrivant l'usage de sa session (par exemple `Revue des notes d'Alice et Jules`).
- `size`: `[s1, s2 ...]` volumes _utile_ des données de la base IDB lors de la dernière session ouverte sur ce _device_.
- `time`: dernière date-heure d'ouverture de cette session sur ce terminal.
- `prefCode`: code de la "préférence" utilisée la dernière fois.
- `prefTime`: _epoch_ date-heure de la dernière mise à jour de cette préférence.
- `prefObj`:  sérialisation (en binaire) de cet objet de "préférence" utilisé la dernière fois.

Il existe une base de données IDB de nom `app_x` où `x` est le hash court de `userId + '/' + profId`: elle contient les **documents en cache** de cette session.

#### Déclaration d'un _device_ de confiance
Depuis le _device_ à déclarer de confiance, l'utilisateur doit s'authentifier de manière _forte_ en donnant son couple _pseudo / phrase secrète_ (principal ou secondaire).

Dans sa déclaration de confiance il saisit:
- son `pseudo` et `devName` le nom qu'il donne à ce _device_: les valeurs par défaut sont proposées, par exemple `Bob` et `PC d'Alice`.
- un code `PIN` (d'au moins 8 signes).

Le module _safe terminal_ demande au module _safe server_ d'accéder au safe de l'utilisateur identifié par `SH(p0)` et de lui retourner le `userId` et `Ka` (ou `Kr`)associé:
- disposant du couple `p0 p1` (ou `r0 r1`), le module _safe terminal_ obtient la clé `K` du safe de l'utilisateur en décryptant `Ka` (ou `Kr`) par le `SH(p0, p1)` (ou `SH(r0, r1)`).

Le module _safe terminal_,
- génère aléatoirement `devId` si cette donnée ne figure pas encore dans le `header`.
- génère les challenges aléatoires `cx cy`.
- calcule `Kp`, cryptage de cryptage de la clé `K` par le `SH(PIN + cx, cy)`.
- génère un couple `Sa Va` de clés asymétriques signature / vérification.
- calcule `sign`, signature par `Sa` du `SH(PIN, cx)`.
- calcule `sh1p / sh1r` comme `SH(p1) / SH(r1)`.
- enregistre dans la table `trustings` de la base IDB `Safe` un row avec les colonnes `userId pseudo cx Ka Kr Kp`.
- transmet au service _safe_ `userId, devId, sh1p, sh1r, devName(crypté par K), Va, cy, sign` qui,
  - accède au _safe_ dont l'id est `userId` et vérifie que `hhp1 / hhr1` est bien le SHA de `sh1p / sh1r` (s'assure que _safe terminal_ détient le bon `p1 / r1`).
  - y créé dans la section `devices` une entrée `devId` avec les données `devName Va cy sign nbe = 0`.

> Remarque: `Sa` a servi à générer la signature `sign` mais n'est plus utilisé ensuite et n'est pas mémorisé alors que `Va` l'est et servira à authentifier la signature d'un PIN saisi par l'utilisateur.

Après ce calcul,
- le _safe_ a été mis à jour par le service _safe_ avec un nouveau device de confiance avec les données cryptographiques permettant à l'utilisateur de s'authentifier par un code PIN.
- sur le _device_ la base locale IDB _safe_ contient une entrée relative à ce _safe_ avec en particulier la clé K du _safe_ cryptée en `Ka` `Kr` et `Kp`. 

#### Authentification par code PIN depuis un _device déclaré de confiance_
Le module _safe terminal_ lit la base IDB _safe_ et, 
- propose à l'utilisateur de désigner la ligne de `trustings` dont la propriété `pseudo` (par exemple `Bob`) lui correspond. Le module dispose ainsi des données `userId cx Kp`.
- demande à l'utilisateur de saisir le PIN associé et calcule `z = SH(PIN, cx)`.
- transmet au service _safe_ `userId, devId, z` qui,
  - accède au _safe_ dont l'id est `userId`.
  - accède dans la section `devices` à l'entrée `devId` ce qui lui donne les propriétés `Va cy sign nbe`. Si cette entrée n'existe pas c'est que le _device_ N'EST PAS / PLUS de confiance pour ce _safe_,
    - soit n'a jamais été déclaré comme tel,
    - soit la confiance en lui a été retirée explicitement par l'utilisateur,
    - soit qu'il a été supprimé du fait d'un nombre excessif d'essai erroné de code PIN.
  - vérifie par `Va` que `sign` est bien la signature de `z`. En cas de succès, il met à 0 `nbe` s'il ne l'était pas déjà et sinon incrémente `nbe`.
  - retourne le challenge `cy` au module _safe terminal_ qui peut ainsi calculer la clé `SH(PIN + cx, cy)` qui décrypte `Kp` ce qui lui donne la clé K du _safe_.

##### Échecs
SI la signature `sign` n'est pas vérifiée par `Va`, c'est que le code PIN n'est pas le bon. `Va` correspond au `Sa` qui a été utilisé à sa signature, `cx` était bien celui fixé à la déclaration. **Le nombre d'erreurs `nbe` est incrémenté**.

Si ce nombre est égal à 2, il y présomption de recherche d'un code PIN par succession d'essais, l'entrée `devId` est supprimée. L'utilisateur devra refaire une _déclaration de confiance_ de ce device avec un code PIN (ce qui exigera une authentification _forte_ de sa part par `p0` et `p1`).

### Accès d'une application en mode _avion_ (pas d'accès au réseau)
La table `tsessions` de la base IDB _Safes_ permet de lister les sessions qui ont été ouvertes sur ce _device_ pour cette application avec pour chacune,
- le texte `about` de son profil, par exemple `Revue des notes d'Alice et Jules`,
- le pseudo du _safe_ correspondant, par exemple `Bob`.

L'utilisateur désigne la session qu'il souhaite ré-ouvrir ce qui lui donne:
- le `userId` de cette session,
- le `profId` du profil de cette session,
- `Ka` la clé K de ce _safe_ mais cryptée par `p0 p1` d'authentification du _safe_.
- `prefCode prefObj` les valeurs de _préférences_ de la sessionK.
- le nom de la base IDB cache des documents.

L'utilisateur saisit son couple `p0 p1` pour obtenir sa clé K depuis `Ka` ou `Kr`:
- le succès du décryptage authentifie sa propriété du _safe_.
- ses préférences d'ouverture de la session sont accessibles et sa base IDB est lisible.
- la session peut être ouverte, en lecture seulement.

> En mode _avion_ l'authentification par code PIN n'est pas possible.

### Sécurité de l'authentification par code PIN depuis un _terminal de confiance_
Sur un terminal NON déclaré de confiance l'utilisateur doit fournir un couple `p0 p1` ou `p0` est un pseudo / nom / etc et `p1` une phrase secrète longue, soit une bonne trentaine de signes ce qui est considéré comme inviolable par force brute avec un minimum de précaution dans le choix de `p1`.

Sur un terminal déclaré de confiance par l'utilisateur il _suffit_ d'un code PIN de 8 signes (ou plus), donc _a priori_ beaucoup plus facile à craquer. Mais, 
- en l'absence de piratage technologique, l'utilisateur a droit à deux essais infructueux, le code PIN s'auto-détruisant passé ce seuil. Les essais multiples sont voués à l'échec.
- le code PIN est spécifique de l'utilisateur ET de chacun des terminals qu'il a déclaré de confiance (sauf s'il donne toujours le même): il faut s'être connecté (par login _système_) sur un terminal déclaré de confiance préalablement pour pouvoir l'utiliser.

Ceci veut dire que la _vraie_ sécurité repose sur,
- la connaissance d'un compte de login du terminal qui est déjà sensé être particulièrement protégé,
- le fait que le terminal ait été préalablement déclaré de confiance et protégé par un code PIN,
- le fait qu'au delà du second essai infructueux, la confiance dans ce terminal est retirée.

Le **code PIN** n'est jamais stocké ni passé en clair sur le réseau au service _safe_: 
- il ne peut pas être détourné ou être lu depuis la base de données.
- il ne figure que temporairement en mémoire dans le module _safe terminal_ inclus dans l'application _myApp1 terminal_ durant la phase d'authentification du _safe_ de l'utilisateur.

Pour tenter depuis les données de la base de données du service _safe_ d'obtenir le code PIN par force brute, il faut effectuer une vérification de `sign` par `Va` avec le _challenge_ `SH(PIN, cx)` mais `sign` est crypté par la clé privée de cryptage général du service _safe_.

Pour que cette dernière attaque pour trouver le PIN de `Bob` par force brute ait des chances de succès, il faut que le hacker ait obtenu frauduleusement:
- (1) le contenu en clair de l'objet _safe_ en base et pour cela il lui faut conjointement,
  - avoir accès à la base en lecture ce qui requiert, soit une complicité auprès du fournisseur de la base de donnée, soit **la complicité de l'administrateur technique**.
  - avoir la clé de décryptage des contenus de celle-ci inscrite dans la configuration de déploiement du service. Ceci suppose la **complicité de l'administrateur technique** effectuant ces déploiements.
- (2) ait obtenu le challenge `cx` stocké dans la base IDB _safe_ du terminal ce qui suppose,
  - d'avoir une session ouverte sur le terminal (mot de passe du login sur un PC, sur un mobile avoir le mobile _déverrouillé_).
  - d'ouvrir une application pour pouvoir lire en _debug_ la base de données IDB _safe_.

Ayant obtenu le challenge `cx`, il faut ensuite écrire une application dédiée pour craquer par force brute le code PIN en tentant la vérification de signature `sign` par la clé `Va` du challenge `SH(PIN, cx)`.

> Ce double _piratage / complicité_ donne accès à la clé `K` du _safe_ de `Bob`, donc au contenu du _safe_. Toutefois `p0 p1 r0 r1` restent inviolées et non modifiables par le hacker, puisque ne résidant que dans la mémoire de l'utilisateur.

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

# Questions ouvertes

### Comment éviter une inflation incontrôlable de création de _safes_ fantômes
Un utilisateur ne pourrait créer un _safe_ qu'après avoir obtenu un ticket d'invitation déposé par un autre _utilisateur_.
- le ticket a une durée de vie limitée.
- le code du ticket parvient par un moyen externe (mail ...).
- le nombre de tickets généré par un utilisateur est limité (N par mois / an ...).
