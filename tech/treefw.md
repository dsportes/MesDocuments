---
layout: page
title: Réflexions à propos d'un framework "TreeFw"
---

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

# Objectifs du framework
Cette couche logicielle a plusieurs objectifs:
- gérer la persistance des arbres de documents et fichiers dans la base de données et le storage.
- gérer la _synchronisation cohérente_ de copies distantes des arbres.
- notifier par _push_ des applications distantes des évolutions des arbres auxquels elles ont souscrites.

Le framework a deux niveaux d'action:
- **dans une Cloud Function _centrale_** ayant accès à la base de données et au Storage. Ce peut être un serveur HTTPS standard, ou une Cloud Function sous différentes variantes technique.
- **dans des applications de différentes natures techniques** faisant appel aux services des Cloud Functions centrales, typiquement une application Web, une application mobile, etc.

## Gestion de la persistance _centrale_ dans une Cloud Function
Le framework gère le stockage des documents et fichiers dans le couple base de données qui Storage détient _l'original_ des **documents et fichiers regroupés en arbres** comme expliqué ci-avant.
- cette gestion est sécurisée par des transactions ayant la propriété ACID du SGBD.
- les contenus des fichiers créés / modifiés par une transaction doivent avoir fait l'objet d'un `preLoad` en Storage AVANT mise à jour par une transaction. Le framework assure le nettoyage des contenus du Storage non référencés dans un arbre (suite à un incident technique).

### Création / mise à jour / suppression
Une couche logicielle d'API permet à la couche applicative de Cloud Functions d'accéder aux documents en mémoire sous forme d'objet de classes applicatives et de déclarer des créations / suppressions / mises à jour des documents en mémoire.

Les _arbres_ sont créés explicitement et supprimés explicitement: un arbre vide peut exister.

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

#### Cohérence _forte_ ans un arbre, _faible_ entre arbres
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

# Applications clientes
## PWA : _progressive web app_
Une telle application s'exécute dans le contexte d'un browser mais a sa propre fenêtre:
- l'application (son logiciel) est identifiée par son URL.
- une instance d'application correspond à une installation dans un browser sur un device donné (par exemple une sous Chrome et une sur Edge).

Pour simplifier il n'y a à un instant donné au plus une instance d'application donnée sur un device donné.

> Rien n'interdit de mettre en ligne le même logiciel (au détail près de son `manifest`) sous 3 URLs: dans ce cas on peut ouvrir une fois ... 3 applications identiques (au nom près).

## Application Mobile (androïd)
Il n'y a au plus qu'une instance d'une application donnée sur un mobile donné. Elle a pu être lancée au démarrage du mobile et s'exécuter en _background_.

> Remarque: si une application s'exécute sous l'habilitation d'un _user authentifié_ elle doit prévoir:
- soit de connecter à un _user_ puis se déconnecter avant de se connecter à un autre.
- soit de gérer autant de contextes qu'il y a de users connectés en même temps dans l'application.

## Token
Une **instance d'application** distante (Web ou mobile) est identifiée par un `token` qui est une adresse à laquelle des messages pourront être poussés depuis un serveur (Cloud Function, ...) afin d'alerter l'instance de l'application sur l'évolution des documents de son, ou **ses** _périmètres_ d'intérêt.

Une instance d'application peut être en état,
- _foreground_: elle est visible par l'utilisateur et a le focus. Pour une application PWA sa fenêtre est sur le dessus et a le focus. Dans cet état l'application reçoit immédiatement les messages poussés et peuvent mettre à jour leurs copies locales de documents et leur UI.
- _background_: l'application n'est pas visible, cachée, voire même pas lancée. Dans cet état l'arrivée d'un message poussé provoque quand l'utilisateur clique sur la notification qui s'affiche,
  - si l'application est en exécution, son pop au premier plan,
  - si elle ne l'est pas son lancement.

## Authentification
Après avoir été lancée une application (PWA par exemple) peut soumettre des requêtes à son serveur (ou un de _ses_ serveurs) mais ne pourra pas accéder à beaucoup de données, la quasi totalité d'entre elles requérant un `Account` authentifié.

S'authentifier auprès d'un serveur consiste à lui soumettre des données de _credential_ (login / password, phrase secrète, etc.) celui-ci retournant à l'application en cas de succès un document `Account` portant a minima les propriétés:
- `id` : l'identifiant permanent du compte,
- `perimeter` : le périmètre du compte, une liste d'identifiants d'arbres `treetype itd` donnant pour chacun la liste des types de documents qui l'intéressent.

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

### Problème
Une application cliente pourrait se fabriquer un périmètre hors de ses limites.

Il faut faire calculer le périmètres sur le serveur et le retourner scellés / cryptés par la clé du serveur.

