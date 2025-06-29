---
layout: page
title: Use-cases : circuitscourts, asocial
---

# Use Case _asocial_

### Compte
Un _compte_ enregistré dans une organisation de _asocial_:
- a une _carte de visite_ (photo et texte) facultative, de son choix, qui n'est lisible que par ses _contacts_.
- a des notes personnelles.
- a des _chat un à un_ avec d'autres comptes.
- peut sponsoriser la création de nouveaux comptes.
- est **membre** d'un groupe _principal_ et peut être membre de plusieurs autres groupes _hébergés_.

Un _contact_ est soit l'interlocuteur d'un _chat un à un_ du compte, soit un membre des groupes dont le compte est membre.

Un compte a un identifiant aléatoirement fixé à sa création et des clés de cryptages:
- une clé personnelle qui crypte ses notes personnelles et n'est communiquée à aucun autre compte (et non lisible par le serveur).
- une clé d'accès à sa sa _carte de visite_ transmissible à ses _contacts (mais non lisible par le serveur).
- une clé _asymétrique_ technique qui sert à échanger les clés des _carte de visite_ et de _groupe_.

### Groupe
Un _groupe_ enregistré dans une organisation de _asocial_:
- a un _chat_ pour les discussions entre les membres du groupe. 
- des membres qui sont des comptes, dont certains ayant le _privilège d'animation_ du groupe.
- des notes réservées aux membres du groupe.

Un compte a un identifiant aléatoirement fixé à sa création et une clé de cryptage transmise aux membres du groupe (mais non lisible par le serveur) :
- elle donne accès aux membres aux informations du groupe et à sa _carte de visite_.
- elle crypte les notes du groupe.

Un groupe **_principal_** rassemble des comptes, de 1 à quelques dizaines (une famille, une équipe, une petite association ...), dont les coûts d'abonnement et de consommation de calcul sont décomptés globalement et le cas échéant facturés globalement.

Un groupe **NON principal** est _hébergé_ par **UN** groupe principal qui en gère les limites maximales d'espace qui peuvent être utilisées et sont effectivement utilisées.

### Chat
Un chat est une suite limitée d'échanges textuels courts entre ses participants,
- _chat un à un_:
  - il n'y a que deux interlocuteurs.
  - les textes sont cryptés par une clé spécifiquement générée pour le chat et transmise aux deux interlocuteurs (la clé n'est pas lisible par le serveur).
- _chat de groupe_:
  - tous les membres du groupe sont des interlocuteurs.
  - les textes sont cryptés par la clé du groupe.

### Note
Une note,
- contient un texte de quelques milliers de signes au plus,
- peut contenir des _fichiers_ (d'une centaine de Mo au plus) ayant un type MIME.

Le texte et les fichiers sont cryptés,
- soit par la clé du compte pour une note personnelle,
- soit par la clé du groupe pour une note de groupe.

### Maîtrise des ressources d'hébergement
Deux classes de ressources sont gérées:
- **les ressources _d'espace_**: elles ont un coût récurent que l'application soit utilisée ou non. Elles correspondent à un _abonnement_ dont le niveau dépend des limites maximales allouées.
  - nombre maximal de `na: nn+nc+ng` (`nn`: nombre de notes personnelles, `nc`: nombre de chats personnels, `ng`: nombre de participations aux groupes).
  - volume maximal `va` de fichiers des notes personnelles.
  - nombre maximal de notes du groupe et des groupes hébergés.
  - volume maximal de fichiers attachés à ces notes.
- **les ressources _de calcul_**: elles sont décomptées lors des opérations demandées par un compte. Le coût de consommation est forfaitaire en de leur seuil et décompté exactement au delà de ce seuil.
  - nombre de lectures et d'écritures effectuées au cours des actions exécutées sur demande d'un compte membre du groupe,
  - volume des fichiers _lus_ (download) et écrits (upload) par les membres du groupe.

#### Groupe _principal_
Il gère la comptabilité de ses membres. Chaque compte ne peut être membre que d'un seul groupe principal et peut en changer, à savoir en une seule opération être résilié de son groupe principal actuel et devenir membre d'un autre groupe principal: 
- peut être _bloqué_ par l'organisation (voir les alertes ci-après).
- enregistre pour chacun de ses membres,
  - les limites maximales d'espace attribuées : `na` et `va`.
  - les espaces effectivement utilisés `nu` et `vu`.
  - la limite de consommation journalière moyenne (sur M / M-1),
  - la consommation effective sur M et M-1.
- enregistre pour chacun de ses groupes hébergés,
  - les limites maximales d'espace attribuées : `na` (nombre de notes) et `va` (volume des fichiers attachés aux notes).
  - les espaces effectivement utilisés `nu` et `vu`.

**Groupe _gratuit_**: 
- ses limites globales d'espace attribuables aux membres et aux groupes hébergés sont fixées sous privilège _comptable_.

**Groupe _payant_**:
- ses limites globales d'espace attribuables aux membres et aux groupes hébergés sont fixées sous privilège de _membre animateur_ du groupe.
- il dispose d'un **solde monétaire** :
  - le solde est crédité à l'occasion de **paiements** dont l'historique est conservé.
  - le solde est débité,
    - virtuellement chaque milliseconde pour la partie _abonnement_
    - à chaque opération demandée par un membre du groupe. 
- il est bloqué quand le solde est négatif.

> Un groupe principal subit a minima une action par mois afin que son solde soit significatif dans l'enregistrement historique mensuel.

#### Un groupe _hébergé_ ...
- est _rattaché_ à **son** groupe principal qui en gère les limites maximales d'espace attribuées et les espaces effectivement utilisés.
- peut changer de groupe principal hébergeur.

#### Abonnement / consommation par _compte_
Un compte a des coûts d'abonnement qui lui sont propres mais de plus partage l'espace des groupes desquels il est membre et qui ne peut pas être facilement réparti. Son décompte _d'abonnement_ inclut arbitrairement une quote part de l'espace utilisé par les groupes hébergés par le groupe principal. 

> En revanche aucune quote part ne lui est affectée concernant sa présence comme membre de groupes hébergés par un autre compte principal que le sien. S'il en _abuse_, l'animation de ces groupes peut aussi lui interdire l'accès en écriture et la capacité à faire dériver le volume utilisé. 

**La consommation d'un compte** est une donnée plus objective correspondant aux lectures, écritures, volumes montant et descendant, effectivement décomptées sur les actions exécutées au nom du compte.

### Contrôle de l'espace global alloué à une organisation
Ce contrôle est effectué à travers les compteurs suivants enregistrés dans un document `Espace` singleton par organisation:
- nombre maximal `nd` (de notes / chats / participations aux groupes) distribuable à l'ensemble des groupes principaux.
- nombre `na` (de notes / chats / participations aux groupes) effectivement attribué aux groupes principaux.
- volume maximal `vd` (des fichiers des notes) distribuable à l'ensemble des groupes principaux.
- volume `va` (des fichiers des notes) effectivement attribué aux groupes principaux.

### Alertes
Une _alerte_ est un signal fort visant un compte ou un ensemble de comptes. 
- une alerte porte un message et peut être _sans frais_ ou _porteuse d'une restriction_ faible ou forte.
- une alerte provoque une notification poussée aux applications qui en ont souscrit l'écoute.

##### Alerte globale de l'administrateur technique.
Ce document singleton par organisation provoque une _notification_ à chaque mise à jour (dont l'effacement _OK_) par l'administrateur technique. Toutes les applications à l'écoute de l'organisation reçoivent une notification poussée:
- _annonce d'une indisponibilité future, restriction aux seules actions de lecture seulement, blocage général de toutes les actions_ ...

#### Autres alertes
- alerte d'un groupe _principal_ à tous les membres du groupe
  - explicitement levée par un animateur du groupe ou une action sous _privilège comptable_.
  - solde monétaire faible ou négatif (blocage).
- alerte d'un groupe _principal_ explicitement levée par un animateur du groupe ou une action sous _privilège comptable_ à un membres précis du groupe.

## Types de _credentials_

### AT : privilège d'administration technique (singleton)
- donnée du PBKDF X d'une phrase longue. Le SHA de X est enregistré en configuration du serveur.
- autorise les actions de gestion des compteurs de `Espace`.
- autorise la déclaration / suspension des privilèges _comptables_ des organisations.
- autorise la création de certains reports comptables.

### PC `org` : privilège _comptable_ d'une organisation
- donnée du PBKDF X d'une phrase longue. Le SHA de X est enregistré en table des _phrases de credential_.
- autorise la création d'un groupe principal et de son premier membre animateur.

### AC `org ac` : accès au compte `ac` de l'organisation `org`.

### Table des phrases

## Documents

### Compte
L'identifiant co d'un compte est tiré aléatoirement à 


# Use Case _circuit court_

## Vision générale: les _documents_

Un **point-de-livraison regroupe des consommateurs** et est identifié par son code `gc`. 
- **Document FGC** : fiche de renseignement donnant des informations de contact, son ou ses mots de passe, liste des groupements de producteurs auxquels il peut commander. 

Un **consommateur** est identifié par son code dans son point-de-livraison: `gc co`. 
- **Document FCO** : fiche de renseignement donnant des informations de contact, son mot de passe, un statut de blocage.

Un **groupement de producteurs** est identifié par son code `gp`. 
- **Document FGP** : fiche de renseignement donnant des informations de contact, son ou ses mots de passe, liste des groupes de consommateurs qui peuvent lui émettre des commandes.

Un **producteur** est identifié par son code dans son groupement: `gp pr`. 
- **Document FPR** : fiche de renseignement donnant des informations de contact et son mot de passe.

Une **référence de produit** est identifiée par son code dans son producteur: `gp pr rp`.

Le **calendrier des livraisons d'un groupement de producteurs** est identifié par le code du groupement `gp`.
- **Document CALG** : il donne la liste des livraisons déclarées avec pour chacune:
  - son de code `livr`, date de livraison (théorique et immuable).
  - les dates-heures d'ouverture, de clôture des commandes, d'expédition et d'archivage. Des règles fixent comment et quand ces dates peuvent être changées, en particulier les unes par rapports aux autres.
  - la liste des points-de-livraison (groupes de consommateurs) livrés.

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
  - généré / mis à jour à chaque mise à jour d'un `BCC` du groupe pour une ligne concernant un produit de ce producteur. 
  - il donne par produit du producteur la somme des quantités commandées dans le groupe. Ce document est une redondance générée / mise à jour à chaque mise à jour.

_Remarque 1_: quand une commande est réceptionnée, pour chaque produit (en général commandé mais pas forcément), figure une quantité livrée ou un poids livré.
- pour certains produits, des poulets par exemple, la quantité livrée est le nombre N de poulets et le poids livré est une liste de N poids individuels des poulets. Quand il y a un poids total c'est, soit temporaire en estimation avant obtention des poids individuels, soit la somme des poids individuels.

_Remarque 2_: le rapprochement entre les informations de commande et celles de livraison (réception) permet de détecter des problèmes à résoudre:
- des quantités excédentaires ou insuffisantes: il faudra pour chaque consommateur ajuster la quantité distribué.
- des paquets individualisés (des poulets) au déchargement non attribués et des consommateurs n'ayant pas reçu leurs paquets.
- des produits livrés mais pas commandés: il faudra les répartir sur des consommateurs, le cas échéant un consommateur virtuel représentant le point-de-livraison lui-même.

Le **catalogue général des produits** d'un groupement est identifié par `gp`.
- **Document CATG**. Il donne pour chaque produit:
  - un _descriptif permanent_ ne pouvant pas changer après déclaration, sauf le libellé. Un produit _à l'unité_ ne peut pas devenir _au poids_ (il faut définir un autre produit).
    - son code et sa référence (_code-barre_).
    - un libellé descriptif,
    - si c'est un produit _sec_, _frais_ ou _surgelé_.
    - des indicateurs de label / qualité, taux de TVA applicable, 
    - si le produit est vendu à l'unité, au poids et / ou par _caisse_ / _demi-caisse_.
  - une _liste chronologique_ de dates à laquelle les conditions de ventes du produit ont changé:
    - sa disponibilité,
    - son prix unitaire,
    - ses poids _net_ et _brut_: le poids _net_ est celui sans l'emballage (ce que mange le consommateur), son poids _brut_ inclut l'emballage (ce que ça pèse approximativement dans le camion).

Le **catalogue d'une livraison** d'un groupement est identifié par `gp livr`.
- **Document CATL**. Il donne pour une livraison, la liste des produits avec pour chacun sa condition de vente.
  - la catalogue d'une livraison est _calculé_ avant ouverture de la livraison depuis le catalogue général.
  - il peut être amendé ponctuellement jusqu'à l'ouverture de la commande.
  - après ouverture de la commande, quand une condition de vente d'un produit change, les deux conditions existent: celle _actuelle_ et celle _à l'ouverture de la commande_.
  - les conditions de vente ne peuvent plus changer après la date-heure d'expédition (des paiements définitifs ont pu avoir lieu).

Le **répertoire général** des groupes et groupements est un singleton (pas d'identifiant):
- **Document RG**:
  - pour chaque groupe, une _carte de visite_ du groupe.
  - pour chaque groupement, une _carte de visite_ du groupement.

Le **répertoire des consommateurs** d'un groupe est identifié par le code du groupe `gc`.
- **Document RCO** : pour le groupe lui-même et pour chaque consommateur il donne une fiche de contact.

Le **répertoire des producteurs** d'un groupement est identifié par le code du groupement `gp`.
- **Document RPR** : pour le groupement lui-même et pour chaque producteur il donne une fiche de contact.

Le **chat d'une livraison** est identifié par le groupement livrant et la date de livraison `gp livr`.
- **Document CHL**: c'est une suite chronologique de news alimentée par le groupement. Tous les groupes de consommateurs sont concernés. Un point-de-livraison (groupe de consommateurs) peut y déposer aussi des news (avec modération).

Le **chat d'une distribution** est identifié par le groupement livrant, la date de livraison et le point-de-livraison distributeur `gp livr gc`.
- **Document CHD**: c'est une suite chronologique de news alimentée par les consommateurs.

Le **chat d'un point-de-livraison (groupe de consommateurs)** est écrit par les consommateurs indépendamment de toute livraison et est identifié par `gc`.
- **Document CHCO**: c'est une suite chronologique de news alimentée par les consommateurs. Les groupements de producteurs peuvent émettre, avec modération, des news.

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
- chat de son groupe.
- chats des livraisons ouvertes de son groupe.
- ses commandes sur les livraisons ouvertes. Ce dernier fil peut être utile pour un _consommateur_ pour lequel il y a plusieurs utilisateurs susceptibles de commander: famille, proches, voisins... Il permet de voir apparaître des notifications quand un de ces utilisateurs a modifié une commande.

> Il ne suit pas par fils de news les évolutions tarifaires, les évolutions des dates, etc. C'est l'animateur du groupe qui en fera les informations de synthèses sur le chat du groupe.

### Point de vue d'un groupe (un de ses animateurs) (A SUIVRE).

### _Fils de synchronisation_ des documents
Chaque document est accessible par son identifiant.

Les documents devant être _synchronisés_ sont rattachés à un ou des _fils de synchronisation_ selon leurs propriétés identifiantes. Chaque application terminale déclare à quels _fils_ elle est abonnée de manière à recevoir une notification circonstanciée quand un document rattaché à ce fil a changé.

#### Liste des documents, définition de leurs index
- RG: cartes de visite des groupes et groupements
- RC: cartes de visites des consommateurs d'un groupe: `gc`
- RP: cartes de visite des producteurs d'un groupe: `gp`

- FGC: fiche d'un groupe. `gc`
- FCO: fiche d'un consommateur: `gc co`
  - index 1 : `gc`
  - index 2 : `co`
- FGP: fiche d'un groupement: `gp`
- FPR: fiche d'un producteur: `gp pr`
  - index 1 : `gp`
  - filter : `pr`
- CATG: catalogue des produits d'un groupement: `gp`
- CATL: catalogue d'une livraison: `gp livr`

- CALG: calendrier d'un groupement: `gp`
- LIVRG: livraison d'un groupement: `gp livr`
  - index 1 : `gp`

- BCC: bon de commande d'un consommateur: `gc co gp livr`
  - index 1 : `gc gp livr`
  - index 2 : `gc co livr`
- BCG: bon de commande d'un groupement: `gc gp livr`
  - index 1 : `gc gp`
  - index 2 : `gp livr`
- CART: carton d'un producteur pour la livraison à un groupe: `gp pr livr gc`
  - index 1 : `gp livr gc`
  - index 2 : `gp pr`
  - index 3 : `gp livr`

- CHL: chat d'une livraison: `gp livr`
  - index 1 : `gp`
- CHD: chat d'une distribution: `gp livr gc`
  - index 1 : `gp livr`
  - index 2 : `gp gc`
- CHCO: chat d'un groupe de consommateurs: `gc`
- CHPR: chat d'un groupement de producteur: `gp`

#### Liste des _bags_
La description d'un type de bag donne:
- la **liste de ses propriétés identifiantes**.
- pour chaque document pouvant faire partie du bag:
  - son type,
  - le numéro de l'index (0, 1, 2) définissant ses propriétés d'appartenance, par convention 0 pour l'id complète. Chaque index doit référencer **toutes** les propriétés identifiantes du bag.

Bag `#CMDGC` : `gc gp livr` - commande d'un groupe gc à un groupement gp pour une livraison livr
- `BCG` : 0 - singleton dans le fil
- `CART` : 1
- `BCC` : 1
- Credentials: `CO gc, GC gc`

Bag `#BCC` : `gc co livr` - commandes d'un consommateur gc co pour une livraison livr (tous groupements confondus)
- `BCC` : 2
- Credentials: `CO gc, GC gc`

Bag `#CMDOV` : `gc gp` - commandes d'un groupe gc à un groupement gp
- `BCG` : 1
- Credentials: `GP gp, GC gc`

Bag `#CALGP` : `gp` - calendrier des livraisons d'un groupement gp
- `CALG` : 0 - singleton pour le fil
- `LIVRG` : 1
- `CHL` : 1
- Credentials: `GP, GC, CO`

Bag `#CMDGP` : `gp livr` - commandes à un groupement gp pour une livraison livr
- `CHD` : 1
- `BCG` : 2
- `CART` : 3
- Credentials: `GP, GC, CO`

Bag `#RG` : singleton `RG` - répertoire général des groupes et groupements
- `RG`: 0
- Credentials: `GP, GC, CO`

Bag `#RGC` : `gc` - fiche d'un groupe gc, ses consommateurs, son chat 
- `RC` : 0 - singleton pour le fil
- `CHCO`: 0 - singleton pour le fil
- `FGC` : 0 - singleton pour le fil
- `FCO` : 1
- Credentials: `GC gc, CO gc`

Bag `#FCO` : `gc co` - fiche du consommateur gc co
- `FCO` : 0
- Credentials: `GC gc, CO gc co`

Bag `#CHD` : `gp gc` - chats des distributions d'un groupement gp à un groupe gc
- `CHD` : 2
- Credentials: `GC gc, CO gc`

Fil `#CMDPR` : `gp pr` : commandes à un producteur gp pr
- `CART` : 2

Fil `#RGP` : `gp` - fiche d'un groupement gp, ses producteurs, son chat
- `RP` : 0 - singleton pour le fil
- `CHPR` : 0 - singleton pour le fil
- `FGP` : 0 - singleton pour le fil
- `FPR` : 1

Fil `#FPR` : `gp pr` - fiche du producteur gp pr
- `FPR` : 0

Fil `#CHL` : `gp livr` - chat de la livraison livr d'un groupement gp
- `CHL` : 1
- `CHD` : 1


### Abonnement à un bag, total (tous documents) ou partiel (certains seulement)
Pour s'abonner à un bag il faut fixer:
- son type et son path exact: `#CMDGC/gc1/gp1/livr1`
- si l'abonnement est _partiel_ la liste des types de documents ciblés : `[BCG, CART]`

## Applications terminales: _activités_

Une _classe d'activité_ est décrite par son nom `CMDCO` _commandes d'un consommateur_:
- **des variables immuables de construction**: liste des variables _string_ qui sont fournies à la construction d'une instance et sont invariantes sue le cycle de vie.
  - `org` ; code de l'organisation.
  - `gc` : le code d'un point-de-livraison.
  - `co` : le code d'un consommateur récupérant ses produits auprès de ce point.
  - `initials` de l'utilisateur,
  - `pwd` : mot de passe de l'utilisateur pour accès au couple `gc.co` pour les initiales fournies.
- **la liste des types de credentials** à instancier depuis les arguments de construction: `[CREDCO]`
- **les variables**. Ces valeurs déterminent quels _bags_ sont dynamiquement instanciés. Ce sont:
  - soit des valeurs scalaires: `vmin`
  - soit des listes de valeurs scalaires: `*gp`
  - soit des listes de tuples: `*gpx,livrx`
- la liste des _fils_ gérés par l'activité avec leurs identifiants et la correspondance avec les variables qui les définissent.
  - `#RG` - répertoire des groupes et groupements
    - singleton
    - génère `*gp` par la méthode `lgp()`.
  - `#RGC : {gc} - [RC, CHCO, FGC]` - fiche partielle du groupe gc, ses consommateurs, son chat.
    - singleton
    - `RC CHCO FGC` : sont des singletons de par leur id fixée.
  - `#FCO : {gc, co}` - fiche du consommateur gc co
    - singleton
  - `#CALGP : {*gp}` - calendrier des livraisons du groupement gp
    - N _fils_ : `*gp` est une liste.
  - `#CHD : {*gp, gc}` - chats des distributions d'un groupement gp au groupe gc
    - N _fils_ : `*gp` est une liste.
  - `#CMDGC : {lx.gc, lx.gp, lx.livr}` - commande du groupe gc à un groupement gp pour une livraison livr
    - N _fils_ puisque `*lx` est une liste.

`*gp` : liste des groupements susceptibles d'organiser une livraison.
- **Calculé** par la méthode `lgp(#RG)`
  - la méthode a accès à l'activité et donc en particulier ses _fils_ et les variables.
  - la liste des arguments indique quelles variables et fils elle utilise dans son calcul. Si l'une d'elle change, la liste est recalculée.
  - Si `RG` n'avait pas été un singleton, `lgp` aurait été construite en itérant sur la collection des documents.

`*lx` : liste de tuples, chacun désignant une livraison à afficher.
- **Saisie**: l'utilisateur _désigne_ 0 à N couples `gc, gp, livr` dans une liste.

Cette activité montre un processus complexe: 
- les fils #RG #RGC #FCO peuvent être instanciés en état _à charger_ à l'ouverture de l'activité. Les variables `*gp` et `*lx` sont des listes vides.
- la fin du chargement de `RG` déclenche le (re)calcul de `*gp`.
- il en découle l'instanciation de N fils `#CALGP` et N fils `#CHD` en état _à charger_.

**Remarque:** SI la méthode `lgp()` avait été dépendante d'une variable `vmin` indiquant un _seuil_ quelconque sur les groupements listés dans `RG`, la liste `*gp` aurait été calculée en tenant compte de la valeur de `vmin`.
- Si `vmin` est une valeur saisie, il résulterait de son changement le recalcul de `*gp` par la méthode `lgp()`, avec en conséquence des fils en plus ou en moins.
- des fils `CALGP` apparaîtraient en état _ok_ alors que d'autres seraient en état _à charger_: à l'écran c'est visible, dans le cas ou des totalisations seraient effectuées si tous les _fils_ ne sont pas _ok_, ils sont clairement faux / provisoires / voire même masqués.

Le calcul `lgp()` n'est pas forcément en lui-même complexe: toutefois à la fin d'un recalcul, non seulement le nouvel état (la liste des gp) est important mais aussi:
- les gp ajoutés par rapport à l'état précédent: il faudra _ajouter_ des fils.
- les gp supprimés de l'état précédent: il faudra supprimer **les abonnements** à ces fils.

### Fils _partiels_ d'une activité
Un _fil_ est partiel quand l'abonnement ne porte que sur certains types de ses documents: pour un type _partiel_ l'activité ne va pas être notifié des évolutions des documents non cités.

Mais un _fil_ qui était _partiel_ vis à vis d'un type peut ultérieurement ne plus l'être: l'abonnement devra être mis à jour et le _fil_ sera marqué _obsolète_ (avec 0 comme version détenu en session) afin d'obtenir tous les documents de ce fil.

Pour chaque document susceptible d'être ou non _partiel_, il est indiqué la variable associée éventuelle gouvernant sa qualité _partiel / total_.

### Un fil a plusieurs états
- **_en chargement, en mise à jour, ok_**
  - **en chargement / à charger**: il n'a jamais été demandé, il est _vide_ par méconnaissance de son contenu.
  - **en mise à jour / obsolète**: le fil a été chargé mais une notification a indiqué des évolutions de documents du fil sur le central. La mise à jour a été demandée et est en attente de retour (qui peut être long), le fil est _obsolète_.
  - **ok** : le fil a été chargé, aucune notification n'a été reçue le concernant, il est a priori à jour.
- **_actif, passif_**
  - un fil _actif_ a un abonnement qui peut notifier des évolutions.
  - un fil _passif_ a un contenu mais n'a plus d'abonnement actuellement demandant à être notifié de ses évolutions.

### Chargement / la mise à jour d'un fil
Il faut mettre à jour la base locale: c'est donc une opération _longue_.

Une file des actions en attente sur les fils est gérée:
- la mise en file est immédiate ainsi que le marquage éventuel d'un statut du fil, 
- les actions _longues_ associées sont effectuées en asynchrone.

### Présence de fils _inactifs_ en session
Un fil peut avoir été actif et des documents chargés parce qu'à un moment donné l'utilisateur a eu besoin d'en voir le contenu. Si plus tard ce fil n'a plus d'intérêt faut-il le supprimer ?
- si oui :
  - s'il redevient utile il faudra le recharger intégralement ce qui sera long.
  - il ne sera pas visible en mode avion.
- si non : il encombre la base locale avec des données qui peut-être ont été demandée il y longtemps puis plus jamais.

Une gestion opportuniste est pertinente:
- quand un fil n'est plus actif, sa _date_ de fin d'activité est notée en base locale.
- quand il redevient _actif_, y compris en mode _avion_, la date de fin d'activité est effacée.
- en fin de session, les fils _inactifs_ depuis longtemps sont purgés.
