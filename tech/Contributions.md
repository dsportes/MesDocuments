---
layout: page
title: Contributions en attente
---

# Modules génériques utilisables dans les logiciels serveurs
Ces modules forment une couche logicielle offrant un certain nombre de services de bases raccourcissant et sécurisant l'effort de développement.

## Gestion des _opérations_
**Une opération est initiée par la réception d'un requête HTTP**. La couche de base:
- identifie l'opération demandée en fonction de l'URL.
- fixe son _time_ une date-heure unique (pour le serveur) et croissante qui sera attribuée aux versions des documents créés / modifiés / supprimés par l'opération.
- récupère les paramètres d'entrée et les met à disposition de l'opération dans une structure.
- **gère l'opération comme une succession de phases:**
  - **phase 1:** vérification que les paramètres d'entrée sont bien formés, présents s'ils étaient obligatoires, etc. _Normalement_ l'application terminale s'en est assuré mais l'opération dans le serveur fait cette vérification pour éviter de tomber sur des exceptions dues à des valeurs qui n'auraient pas dues être trouvées en entrée. Cette phase s'effectue sans accès à la base de données.
  - **phase 2:** c'est une transaction au sens de la base de données. Elle lit, met à jour créé et supprime des documents et élabore un résultat. En cas d'exception détectée comme liée à une saturation technique de la base de données, cette phase est annulée et relancée (du moins un certain nombre de fois).
  - **phase 3:** c'est une phase de nettoyage qui peut effectuer quelques mises à jour dans la base de données, typiquement dans la table `FTP` pour la gestion des fichiers.
  - **phase 4:** la phase 2 a produit une liste de documents mis à jour. Cette phase identifie les abonnements associés et génèrent les _notifications_ aux applications abonnées.
- **retourne le résultat à l'application appelante,**
  - construit en phase 2,
  - ainsi que la liste des abonnements calculés en phase 4 auxquels l'application appelante était abonné. Ceci évite à l'appelant d'attendre une _notification_ (qui peut être un peu plus tardive) venant par le browser

## Gestion des documents
Un module gère une _mémoire cache_ des documents les plus récemment demandés et mis à jour. 

Ainsi la demande par une opération en phase 2 d'un document qui se trouve en _cache_,
- soit délivre le document s'il n'est pas exigé à être absolument dans sa version la plus récente mais tolère une _certaine ancienneté_,
- soit, et c'est le cas général, s'assure que c'est bien la dernière version: si c'est le cas seul un index a été lu, sinon le document est lu et conservé en _cache_.

Le module gère également une _mémoire cache de documents_ **pour chaque opération en cours**.
- les documents lus y sont stockés dans leur forme _objet compilé_ (et non le format sérialisé de stockage en base).
- les documents _zombifiés_ et créés y sont aussi stockés.

A la fin de l'opération le module gère les _abonnements_ impactés par les documents modifiés / créés / zombifiés / détruits, et les met à jour dans la base : ils seront utilisés en phase 4 de l'opération pour notifier les applications abonnées.

## Requête de collecte des documents sur un _abonnement élémentaire_
Quand une application terminale souhaite disposer des documents mis à jour pour un _abonnement_ donné, ce module détermine:
- si la mise à jour peut être _incrémentale_ ou si elle doit être _intégrale_,
- quels documents sont à retourner en fonction de la dernière version détenue par l'application terminale.

## Modules _providers_ d'accès à la base de données
Un module _provider_ présente un interface indépendant de la base de données gérée. Pour chacun des services de cet interface, il implémente l'accès effectif à la base qu'il gère:
- mise en forme / sérialisation des documents,
- cryptages / hachages éventuels des _data_ et propriétés indexées,
- gestion des transactions commit / rollback.

Pour un serveur donné, seul le _provider_ requis est importé.

> Typiquement en _test_ l'usage du provider _SQLIte_ simplifie le développement plutôt que ceux qui seront utilisés effectivement en production (_Postgresql_, _Firebase_...).

## Module _providers_ d'accès au _storage_
Les quelques services généraux d'accès sont développés pour chaque type de storage souhaité (AWS-S3, GCP, File-system ...).

## Principe de _déclaration_ statique
La structure des documents (leurs propriétés) se fait par _déclaration statique_ dans des modules de configuration.

## Modules utilitaires
Un module de cryptographie évite de gérer les subtilités du paramétrages des algorithmes.

> Vis à vis des applications terminales, les serveurs ne connaissent **QUE** les concepts de documents / abonnements et mises à jour incrémentales. Le reste de la logique applicative passe par l'usage des opérations.

# L'application _Safe_ 
Cette application a pour objet de gérer des _coffres forts_ pour des utilisateurs.

En lançant _Safe_ un utilisateur donne les éléments d'identification de son _coffre fort_ de manière à ce que les applications lancées ultérieurement sur cet appareil puissent y trouver les diverses données _sensibles_ de l'utilisateur dont ses _droits d'accès_ aux documents des applications.

L'application _Safe_ une fois initialisée sur un appareil est en charge:
- de gérer ses appareils _de confiance_.
- pour chaque application:
  - de stocker `ses droits` et les mettre à jour.
  - de stocker _divers objets_:
    - des objets transmis par un autre utilisateur disposant lui aussi d'un _safe_. 
    - des objets interprétés comme des _options de lancement_.
    - des objets interprétés par l'application comme des _préférences_.
  - de lancer l'application, le cas échéant selon _l'option de lancement_ que l'utilisateur a sélectionnée.

> Le lancement d'applications par le _Safe_ évite le risque de lancement d'une application _piratée_ et permet de choisir le cas échéant des _options_ au lancement ouvrant la session dans un contexte déjà préfixé.

> L'URL de lancement de _Safe_ est un élément de sécurité: le code de cette application est lisible dans le browser et il est possible de vérifier auprès de sites certificateurs que l'application est _fair_. Ceci évite d'avoir à le faire pour chaque application gérée par _Safe_, quoi que ce soit toujours possible. 

# Un utilisateur, ses appareils et ses applications

## Appareils _de confiance_ d'un utilisateur
Un utilisateur qui veut utiliser une application depuis un _device_ est placé devant deux cas de figure:
- **soit il juge l'appareil _de confiance_**,
  - il l'utilise régulièrement, que se soit le sien ou celui d'un proche,
  - il peut y laisser quelques informations cryptées et espérer raisonnablement les retrouver plus tard.
- **soit il n'a pas confiance dans cet appareil** partagé par des utilisateurs _inconnus_, comme au cyber-café ou celui d'une connaissance qui le lui a prêté temporairement:
  - il ne doit pas y laisser quelque information que ce soit, aucune trace de son utilisation de l'application,
  - il ne peut pas compter sur le fait qu'il ait déjà utilisé ce même appareil antérieurement pour y retrouver des données.

Lancer une application depuis un appareil _de confiance_ a plusieurs autres avantages:
- **démarrage plus rapide, moins de réseau et moins d'accès dans le serveur** en utilisant une petite base de données locale (cryptée) pour chaque application comme _cache_ de ses documents: ceux qui y figurent et à jour n'auront pas besoin d'être demandés au serveur de l'application.
- **disponibilité des droits** de l'utilisateur pour chaque application le dispensant de s'en souvenir ou de les copier / coller d'un support externe.
- **possibilité d'accéder à l'application en mode _avion_** sans accès au réseau en utilisant les documents et les droits en _cache_.

> Même _de confiance_ un appareil _peut_ être utilisé par d'autres que soi-même: même dans un cadre familial ou de couple, l'appareil n'est pas strictement _personnel_.

## Lancement d'une application
Depuis son application _Safe_ qui gère son _coffre fort_, un utilisateur peut lancer ses applications:
- **soit depuis un appareil _anonyme_:**
  - soit normalement en laissant _Safe_ obtenir son contenu depuis le serveur _Safe_.
  - soit en fournissant un _file / clé USB_ disposant de ce contenu (crypté).
  - Les applications opèrent en mode **_incognito_**.
- **soit depuis un appareil _de confiance_:**
  - **soit en mode _avion_:** le contenu du _Safe_ est obtenu depuis le cache local crypté du _Safe_. Les applications opèrent en mode **_avion_**.
  - **soit en mode _normal_:** le contenu du _Safe_ est obtenu depuis le serveur _Safe_ (mettant à jour un _cache_ local crypté). Les applications opèrent en mode **_synchronisé_**.

Au lancement d'une application une page d'accueil est présentée:
- elle peut être spécifique de l'option de lancement choisie;
- elle peut être _pré-initialisée_ selon cette même option, voire tout _objet_ stocké dans son _safe_ par l'utilisateur.

Au cours de l'exécution de l'application des droits peuvent être ajoutés / modifiés si l'application est en mode _synchronisé ou incognito_: ils sont répercutés sur le serveur _Safe_.

**En mode _synchronisé_** en fermant l'application, l'utilisateur peut choisir:
- de laisser ses _abonnements de news_ activés: des notifications apparaîtront, même quand l'application sera fermée et même si ce n'est plus le même utilisateur qui dispose de l'appareil. Les _notifications_ ne font qu'annoncer des changements sans en donner les détails et évitent de délivrer des informations confidentielles. 
- de fermer ses _abonnements de news_: aucune notification ne parviendra plus sur l'appareil relativement à cette application. L'utilisateur peut le prêter à quelqu'un d'autre sans risque ... mais lui-même ne recevra plus de notifications (il faut choisir).

**En mode _avion_**, il n'y a pas de réseau, pas _d'abonnements actifs_: les dossiers peuvent être consultés mais pas mis à jour. 
- la base de données locale est cryptée par la clé `K` du _Safe_ de l'utilisateur. Elle contient pour chaque sous-collection associée à un abonnement, les documents qui en faisaient partie à un instant t: ceux-ci ne sont pas du tout dernier état mais a minima dans l'état t où ils ont été accédés la dernière fois sur ce appareil.
- l'utilisateur peut saisir des textes ou formulaires purement locaux et stocker des fichiers (comme des photos prises en mode avion): toutes ces informations pourront être utilisées pour mettre à jour des documents quand le réseau sera à nouveau disponible en sécurité.

**En mode _incognito_** la fermeture de l'application ne laisse pas le choix: les _abonnements de news_ sont tous supprimés, aucune notification ne parviendra plus sur cet appareil résultant de l'usage précédent de l'application.

### Mode _veille_
Les applications ouvertes ont un mode _veille_ optionnel: s'il est activé, l'application se met en veille en cas de non utilisation pendant quelques minutes (fixés selon le degré de paranoïa de l'utilisateur). Pour sortir de la veille un code est nécessaire et au second échec l'application se ferme.

# Glossaire technique

### Strong Hash: `PBKDF`
- **SH(s1, s2, SEP)** (Strong Hash): le SH s'applique à un couple de textes `s1 s2`, typiquement un login / mot de passe, mais aussi aux _passphrase_ en une ou deux parties. Il a une longueur de 32 bytes et est unique pour chaque couple de textes `s1 s2`. Il est _strong_ parce qu'incassable par force brute dès lors que le couple de textes ne fait pas partie des _dictionnaires_ des codes fréquemment utilisés. `SEP` est un caractère de séparation / remplissage qui allonge le couple `s1 + SEP + s2` à une taille minimale.

### Clés asymétriques C / D : cryptage / décryptage
Un couple de clés `Ca / Da` asymétriques généré par A:
- `Ca` est une clé publique de **cryptage**: elle est utilisée par B pour crypter par une clé X un objet qui ne pourra être décrypté que par A.
- `Da` est la clé privée de **décryptage**: elle est utilisée par A pour décrypter par la clé X un objet qui a été crypté par B en utilisant `Ca`.
- la clé X est la même qu'elle ait obtenue depuis Ca/Db ou Cb/Da.

### Clés asymétriques S / V : signature / vérification
Un couple de clés `Sa / Va` asymétriques généré par A:
- `Va` est une clé publique de **vérification**: elle est utilisée par B pour _vérifier que la signature S d'un texte T_ a bien été générée par A en utilisant `Sa`.
- `Sa` est la clé privée de **signature**: elle est utilisée par A pour _générer la signature S d'un texte challenge T_.

-----------------------------------------------------------------------

# Décompter les consommations

Une application peut être gratuite pour certains de ses utilisateurs (voire tous) et payante pour d'autres selon l'usage qu'ils en font. 

Mais même dans le cas où l'application est toujours gratuite pour tous, il peut être cependant nécessaire / intéressant de décompter des unités d'œuvre d'utilisation afin le cas échéant de les limiter.

La mise en œuvre de **contrats** permet,
- d'effectuer des décomptes d'usage par des _utilisateurs_, 
- d'en gérer des limites,
- si souhaité d'établir des _factures / paiements / régularisations_.

## Contrats et utilisateurs, leurs consommations

Un _utilisateur_ est identifié par l'application et tout utilisateur est toujours rattaché **à un instant donné** à un _contrat_. Au cours de sa vie un _utilisateur_ peut être détaché de son contrat précédent et être rattaché à un autre.

Un _contrat_ est un document d'identifiant généré à sa création par l'application visant à suivre et gérer les consommations des utilisateurs qui lui sont rattachés: les consommations sont décomptées pour le mois _courant_ et le mois précédent (immuable par principe).

> Dans le cas général un _contrat_ concerne un _petit collectif d'utilisateurs_ une famille, une équipe, une petite association ... mais éventuellement un seul utilisateur.

Le décompte des consommations sur un contrat peut entraîner des contraintes applicatives, par exemple:
- rejet d'opération pour dépassement de certains seuils,
- ralentissement en cas de dépassement,
- blocage éventuel de certaines opérations pour certains utilisateurs.

## Unités d'œuvre des contrats (UC)

Chaque application déclare ses _unités d'œuvre_ selon deux catégories: les unités de _stock_ et les unités de _calcul / travail_.

Chaque unité a ses compteurs spécifiques: elles ne se moyennent ni se s'additionnent pas entre elles. Définir si c'est nécessaire des unités génériques englobantes.

Les unités sont décomptées,
- pour chaque _utilisateur_ d'un contrat.
- globalement pour le _contrat_ (somme des valeurs pour chacun de ses utilisateurs).

### Unités de _stock / volume_
Leur existence a un coût par le seul effet du temps qui passe. Par exemple:
- S1: nombre de commandes en cours.
- S2: volume en Mo de fichiers stockés.

> Même sans utiliser l'application, le stockage a un coût dépendant du temps.

Le contrat conserve pour chaque unité:
- le valeur _courante_, c'est à dire la dernière valeur attribuée,
- la moyenne sur le mois en cours,
- la moyenne sur le mois précédent.

La moyenne est calculée au prorata du temps passé (à la milliseconde) à chaque valeur.

### Unités de _calcul / travail_
Elles _coûtent_ effectivement lorsqu'elles sont sollicitées. Par exemple:
- C3: **nombre** de lecture de documents D1 et D2.
- C4: **volume en Mo** des fichiers de type F1 téléchargés.

> Tant qu'une application n'est pas utilisée, ces coûts sont nuls.

Le contrat conserve pour chaque unité:
- la valeur _courante_.
- la somme des consommations pour le mois en cours,
- la somme des consommations pour le mois précédent,

Le nombre _courant_ est une **valeur moyenne journalière récente** lissée sur au moins 20 jours:
- après le 20 du mois c'est le total du mois en cours ramené à 24h.
- avant le 20 du mois, par exemple le 5, c'est la somme,
  - de 5/20 fois le total du mois en cours ramené à 24h.
  - de 15/20 fois le total du mois précédent ramené à 24h,

#### _Quotas_
Un _quota_ est codifié par deux chiffres `en`, sa valeur étant `(10**e) * n`.

      3 => 3
     14 => 40
     25 => 500
     37 => 7000 - Kilo
     67 => 7,000,000 - Mega
     99 => 9,000,000,000 - Giga
    125 => 5,000,000,000,000 - Tera
    155 => 5,000,000,000,000,000 - Peta

Un _quota_ permet d'exprimer sur un entier très court (un byte),
- soit un _seuil maximum_,
- soit un ordre de grandeur _forfaitaire_ d'une quantité. Par exemple la quantité 5342 correspond au _quota_ 36 (6000, le plus petit supérieur à sa valeur).

## Quotas applicables à un contrat

Pour chaque UC déclarée, le contrat définit un _quota_ à rapprocher de la valeur courante correspondante. Par exemple:
- S1-nombre de commandes en cours :  un quota de `23` correspond à 300 documents.
- C4-volume en Mo des fichiers de type F1 téléchargés : un quota de `65` correspond à 5000000, soit 5Mo par jour (lissé sur 20 jours).

**Pour les unités de _stock_, le _quota_ est un maximum qui ne peut pas être dépassé**, l'opération demanderesse tombe en exception.
- _sauf_ si l'opération est marquée _privilégiée_,
- _sauf_ si le maximum est certes dépassé mais en baisse.
- le **niveau d'alerte** est le pourcentage de dépassement (au plus 999%) au delà de 80% (0 en deçà).

**Pour les unités de _calcul_ le dépassement du quota peut provoquer un ralentissement de l'opération**, un temps d'attente fonction du taux de dépassement afin de _dissuader l'usage d'opérations coûteuses_, voire pour les grands dépassements de faire tomber l'opération en exception (si elle n'est pas _privilégiée_).

### Pour chaque _utilisateur_
Des _quotas_ par UC sont également fixés par _utilisateur_: 
- l'action de blocage / ralentissement et le niveau d'alerte sont les plus restrictifs des deux _utilisateur / contrat_.

## Décompte continu des unités sur un contrat: arrêté mensuel
Le décompte est établi en nombre pour toutes les unités effectivement consommées dans le mois courant et le mois précédent: il est disponible,
- par utilisateur,
- par sommation, globalement pour le contrat.
- pour les _unités de stock_ le nombre n'est pas entier puisqu'il s'agit d'une moyenne au temps passé effectivement (en milliseconde).

> A la fin de chaque mois un arrêté mensuel est calculé et mémorisé comme _mois précédent_ désormais invariant: l'application peut archiver les factures sur la profondeur d'historique de son choix avec un format adapté au traitement statistique.

## Historique des _quotas_ sur N mois
On conserve après l'arrêté mensuel, pour chaque compteur, sa valeur **forfaitisé** ramené au quota immédiatement supérieur.

### Tarif: prix unitaire pour chaque UC
Un _tarif_ donne un _prix unitaire_ pour chaque UC.

L'application du tarif à tous les compteurs courants _forfaitisés_ donne un _montant monétaire_:
- _provisoire_ pour le mois courant,
- _définitif_ pour le mois précédent.

> La _monnaie_ est virtuelle: c'est à l'application de prciser le _cors_ de sa monnaie en monnaies réelles (euro, dollar ...) pour convertir en monnaie virtuelle de l'application les versements reçus (en monnaie réelle).

> Quand un _contrat_ est utilisé d'une manière assez stable dans le temps, ces montants ont des chances d'être identiques d'un mois sur l'autre.

## Débits et crédits, solde
La _facture mensuelle d'un contrat_ comporte une liste de _débits / crédits_: chacun a,
- un _code_ qui indique sa nature: par exemple _paiement reçu_,
- une _référence_ ou _commentaire_ permettant d'en savoir plus dans l'application.
- un montant positif ou négatif.

La première ligne est un _débit_ correspondant à la consommation du _contrat_ dans le mois.

### Ajustement global de l'application
Une fois la consommation ainsi calculée, l'application peut introduire une ligne finale d'ajustement, au débit ou au crédit:
- remise sur volume,
- remise / pénalité selon le type de contrat,
- le cas échéant **la remise peut être égale au montant de la consommation** ce qui revient à de la gratuité. 

La _balance_ en fin de mois correspond à la balance en fin du mois précédent, plus les crédits, moins les débits.

En cours de mois une _balance provisoire_ est calculable:
- depuis la balance au mois précédent,
- la consommation en cours _forfaitisée_,
- la ligne d'ajustement spécifique de l'application.
- la balance des débits / crédits du mois.

### Politique vis à vis d'une balance négative
L'application fixe sa politique vis à vis d'un _contrat_ présentant une balance _provisoire_ négative. Par exemple:
- tolérance si la balance en début de mois était positive,
- tolérance pour les opérations _privilégiées_,
- alerte selon le nombre de jours estimés avec _balance positive_: simple calcul du nombre de jours pendant lequel la balance restera positive si la consommation se maintient au même niveau que la moyenne du mois courant et du mois précédent.
- rejet de l'opération.

## Authentification d'utilisateurs via leur contrat

Tout utilisateur rattaché à un contrat _peut_ y être associé à une ou plusieurs **passphrases**: 
- une _passphrase_ donnée ne peut être associée qu'à un seul contrat à un instant donné et à un seul _utilisateur_ dans le contrat.
- dans son contrat, une map associe chaque utilisateur à son _objet d'authentification_.

#### Exemple 1: authentification par clés de _signature / vérification_
- un _objet d'authentification_ d'un utilisateur `id` est formé d'un couple `[hkp, kv]`
  - `kv` est la _clé de vérification_ de signature.
  - `hkp` est le hash court de la _clé de signature_ et est listé comme _passphrase du contrat_.
- une opération dispose dans ses _jetons d'accès_ d'un couple `[hkp, sign]`. La méthode d'authentification,
  - recherche un _contrat_ ayant la passphrase `hhp`,
  - dans ce contrat récupère `id kv` de l'utilisateur et vérifie la signature reçue par la `kv` enregistrée.

#### Exemple 2: authentification par _phrase secrète_ de l'utilisateur
- un _objet d'authentification_ d'un utilisateur `id` est formé d'un couple `[lhps, shps]`
  - `lhps` est un hash _long et fort_ comme le PBKDF d'une phrase secrète connue du seul utilisateur.
  - `shps` est un hash _court_ de la phrase secrète et est listé comme _passphrase du contrat_.
- une opération dispose dans ses _jetons d'accès_d'un couple `[lhps, shps]`. La méthode d'authentification,
  - recherche un _contrat_ ayant la passphrase `shps`,
  - dans ce contrat récupère `id lhps` de l'utilisateur et vérifie que le `lhps` reçu correspond au `lhps` enregistré.

> Dans ces deux exemples une seule action localise un contrat, identifie et authentifie un _utilisateur_.

> Un contrat est aussi accessible par son identifiant et contient tous ses utilisateurs enregistrés par leur identifiant.

### Suivi des consommations dans les opérations

Si l'application enregistre les _consommations_, sauf exception toute opération du serveur commence par récupérer un _contrat par défaut de l'opération_ à qui imputer ses consommations d'UC. 

Au cours de l'opération, d'autres _contrats_ peuvent être cités à qui imputer certaines consommations spécifiques selon la logique de l'application. Par exemple le stockage et l'utilisation de documents partagés par un **groupe** d'utilisateurs peut être imputé au _contrat du groupe_ plutôt qu'aux _contrats_ des utilisateurs eux-mêmes.

### Début et fin d'opération
Le début d'une opération va, en général, 
- lire le _contrat_ depuis la base et authentifier l'utilisateur demandeur de l'opération,
- en cas de basculement sur le mois suivant, calculer la facturation et repositionner les compteurs.

En cours d'opération, les compteurs du mois courant sont mis à jour (si nécessaire), la _balance provisoire_ peut être réévaluée ainsi que le nombre de jours en _balance positive_.

La fin d'une opération va, en général, enregistrer en base la mise à jour du _contrat_.

Un _contrat_ a un mois de dernier calcul d'arrêté comptable: une tâche périodique à partir du N d'un mois effectue a minima une opération sur chaque contrat dont le dernier mois d'arrêté n'est pas le mois courant afin d'éviter de perdre des facturations sur les _contrats_ peu utilisés.

Un traitement périodique peut aussi collecter un historique sous forme de fichier CSV pour analyse externe.

## Détail du document `Contrat`
Propriétés:
- `statCode` (indexé) : code statistique: pour déclenchement sélectif d'actions / reports.
- `lcm` (indexé) : dernier mois calculé / facturé: pour déclencher le calcul de la facture.
- `balance` : balance temporaire en cours de mois.
- `negBal` (indexé) : niveau d'alerte de balance négative.
- `nbdPos` : nombre de jours estimés restant avec balance positive si la consommation continue de suivre la tendance de M et M-1.
- `minLrdu` (indexé) : `lrdu` le plus ancien des _users_.
- `time` : date-heure du dernier calcul.
- `users` : map avec une entrée par `userId`: `{ ldr, quotas, counts }`
  - `lrdu` : dernier jour de référence, utilisation comme entrée d'un contrat. 
  - `quotas` : array de N+1 éléments pour un historique de N mois.
    - le premier élément donne les quotas courants (_maximum_),
    - les éléments suivants donnent les consommations _forfaitaires_ constatées sur le mois.
    - chaque éléments a C compteurs (courts).
  - `counts` : Un array de C éléments chacun étant `[ vc, cm, pm ]` 
    valeur courante, mois courant, mois précédent.
- `total` : `{ quotas, counts }`. somme des compteurs des _users_.
- `invoices` : deux éléments, mois en cours, mois précédent `[dbcr, balance]`:
  - `dbcr` : liste de _débits / crédits_: chacun a `[ code, ref, m ]`:
    - `code` : indique sa nature: par exemple _paiement reçu_,
    - `ref` : une _référence_ ou _commentaire_ permettant d'en savoir plus dans l'application.
    - `m` : un montant positif ou négatif.
  - `balance` : balance au début du mois
- `authKeys` (indexé) : liste des clés d'accès externes aux _users_.
- `authUsers` : map avec une entrée par userId. 
  - _val_ : objet contenant les informations d'authentification. 

**Remarques**
- La liste `authKeys` est calculable par l'application depuis la map `authUsers`.
- les _compteurs_ sont des _number_, les _compteurs forfaitaires_ sont des bytes.
- L'application déclare une _configuration_:
  - le nombre de mois historisés.
  - une liste avec un terme [code, stock] par UC:
    - _code_: ce code court sert aux traces / éditions.
    - _stock_: true si c'est une unité de _stock_ (sinon c'est une unité de calcul).

### _Users_ virtuels
Il est possible de décompter des consommations et définir des quotas maximum pour des _usages_ identifiés. Par exemple:
- des _groupes_ sont rattachés à un contrat.
- chaque _groupe_ peut avoir des quotas pour certains compteurs, voire disposer d'un calcul de consommation propre.

Certains users virtuels peuvent être gérés comme des entrées de _comptabilité analytique_: chaque consommation _réelle_ est _dédoublée_ dans un _user_ ET dans un _user virtuel analytique_.

On peut déclarer ainsi un _user_ identifié par l'application, mais ce _user_ étant virtuel n'a pas d'entrée dans `authUsers`.
- l'application donne dans sa configuration une liste des identifiants des users ayant un comportement _analytique_.
- la ligne **total** du contrat ne totalise pas les lignes des _users analytiques_ mais les lignes correspondant à des consommations réelles.

### Contrats / _users_ disparus
L'application peut considérer comme _disparu_ des _users_ qui n'ont pas _utilisé_ leur contrat depuis un certain nombre de jours.

Il est possible de filtrer par une tâche tous les contrats ayant un _user_ présumé disparu et le cas échéant de gérer:
- leur suppression dans le contrat,
- voire la suppression du contrat lui-même si aucun _user_ n'est plus vivant.

--------------

# Contributions diverses en attente

### Lecture d'un fichier
Elle peut s'effectuer de deux manières:
- en retournant le contenu binaire du fichier dans la couche applicative,
- en retournant une URL d'accès sécurisé valable un certain temps, typiquement à transmettre à une application externe.

### Cohérence _forte_ / _faible_ entre collections de documents
Toutes les collections de documents et documents dont la synchronisation a été demandée par la même requête sont en cohérence _forte_, calculés exactement sur le même état de la base noté par la date-heure t de l'opération.

En revanche, quand des documents sont synchronisés par deux requêtes (r1 à t1) et (r2 à t2) différentes, leur cohérence est _faible_, certains étant connus dans l'état t1 de la bse et d'autres dans l'état t2 de la base: toutefois pour ces documents rien ne dit que les état à t1 et t2 sont différents ou identiques.


> On pourrait certes grouper dans une même requête toutes les _synchronisations de tous les abonnements_ mais la requête put être longueet le volume (trop) important. Il y a applicativement un compromis à choisir entre _force de la cohérence entre documents_ et lourdeur technique.

### Authentification _double_
(questions, pertinence)

Faut-il prévoir d'obliger à une authentification depuis un appareil #1 exigeant une confirmation sur un appareil #2 (favori ou non).
- que se passe-t-il quand l'appareil #1 n'est déclaré _favori_ ?
- et si l'utilisateur N'A PAS d'appareil #2 (au moins sous la main) ?
- si #2 n'est PAS favori, pour valider le login de #1 il lui faut un couple (s1, s2) qu'il vient a priori déjà de donner sur #1 ?

### Attacher des _droits d'accès_ aux _abonnements_
Il peut être requis par une application qu'un _droit d'accès_ soit fourni pour chaque abonnement élémentaire.
- ceci évite qu'une session soit _notifiée_ de mises à jour de documents auxquels elle n'a pas droit d'accès.
- la _synchronisation effective_ (récupérer le contenu des documents) est sauf exception soumise à des _droits d'accès_.

## OBSOLÈTE ??? : _activités_ définies dans une application

Dans une application terminale une _activité_ désigne un ensemble de tâches cohérentes qu'un utilisateur peut effectuer. Par exemple dans l'exemple _circuitscourts_:
- l'activité _commande d'un consommateur_ où un consommateur peut déclarer les quantités qu'il souhaite recevoir pour les livraisons en cours.
- l'activité _contrôle et réception des livraisons_ pour les animateurs d'un point-de-livraison visant à vérifier les commandes, réceptionner les camions et noter les quantités reçues.
- l'activité _préparation d'une livraison_ pour un groupement de producteurs, gérant le calendrier de la livraison, rassemblant les cartons préparés par les producteurs pour effectuer une tournée auprès des points-de-livraison associés.

Une _activité_ décrit à la fois:
- sur quelles données elle doit opérer, quels documents doivent être rendus visibles aux utilisateurs ayant opté pour cette activité.
- quels processus _suites d'actions élémentaires concourant à un objectif plus global_, un utilisateur peut engager dans le cadre de cette activité.
- quels _credentials_ un utilisateur doit présenter pour avoir le droit de voir les données et d'exécuter les processus.

### Une session d'une application pour un utilisateur peut avoir plusieurs activités ouvertes

#### L'activité d'amorce
Il est toujours possible de lancer une application sans avoir d'activité favorite enregistrée.
La session doit présenter un minimum d'information à l'utilisateur:
- pour qu'il puisse décider quelle activité il va choisir d'exercer.
- saisir les _credentials_ requis.

Pour ce faire l'utilisateur va souvent avoir besoin de voir quelques données: elles sont définies dans une activité bien identifiée _amorce_ qui peut présenter des informations non soumises à un _credential_.

Le _desktop de l'application_ est affiché pour présenter ces choix.

Ultérieurement plusieurs activités peuvent être ouvertes:
- plusieurs du même type: assurer _le contrôle et réception des livraisons_ pour deux points-de-livraisons.
- plusieurs de types différents: assurer _la commande d'un consommateur_ (pour lui-même) et contribuer à _la préparation d'une livraison_ à titre d'aide d'un groupement de producteurs ami.

### Un _type d'activité_ a des paramètres et est associé à un ou plusieurs _types de credentials_
Par exemple l'activité _commande d'un consommateur_ a pour paramètres,
- `gc` : le code d'un point-de-livraison.
- `co` : le code d'un consommateur récupérant ses produits auprès de ce point.
- `initials` d'un utilisateur,
- `pwd` : mot de passe déclaré pour le couple `gc co` pour les `initiales` fournies.

Ce type d'activité est associé à un ou plusieurs types de _credential_, ici par exemple `CREDCO` qui peut se construire à partir des paramètres de l'activité:
- `gc co` : sont les identifiants d'un _credential_ `CREDCO` dont les autres propriétés sont `initals` et `pwd`.

Le ou les _types de credentials_ déclarés associés à une activité, doivent avoir tous leurs paramètres dans les paramètres du _type d'activité_: un _credential_ peut ainsi être généré depuis ceux-ci et sera délivré aux opérations qui le requièrent.

### Choisir et exercer une activité dans une session d'une application terminale
Le _desktop_ de l'application présente à l'utilisateur les _types d'activité_ qu'il peut choisir. 
- Ce peut être une liste courte,
- Ce peut être pour une application complexe une liste longue, avec une possibilité de sélection par mot clé et / ou une présentation arborescente.
- Le paramétrage du _desktop_ déclare comment il apparaît et comment l'utilisateur peut sélectionner une activité.

Quand l'utilisateur a choisi un type d'activité il doit saisir tous les paramètres de l'activité: par exemple un code `gc` et un code `co`, des `initials` et un `pwd`.

Il a alors une _activité ouverte_, ce qui apparaît sur son _desktop_.

Il peut en ouvrir d'autres, du même type ou non, chacune figurée par exemple par un onglet et / ou une icône et / ou un libellé.

Depuis le _desktop_ l'utilisateur peut _basculer d'une activité à une autre_ par exemple en cliquant sur un onglet ou une icône, ou si l'application le permet en voir plus d'une affichée (une en haut, une en bas).

### Enregistrement d'une _session favorite_ dans son _profil_
Si l'utilisateur a un profil enregistré, à n'importe quel moment de sa session de travail en cours il peut:
- sélectionner en les cochant certaines de ses activités en cours,
- enregistrer cette sélection en lui donnant un libelle clair pour lui, comme _commandes Bob à JP_.

# Authentifier les utilisateurs d'une application
Un utilisateur d'une application y est enregistré par un document _User_ portant un identifiant _userId_ permanent et immuable.

L'objectif du processus d'authentification est de s'assurer que la personne _physique_ qui tente d'accéder à son _user_ est bien habilité à le faire.

### Les authentifications _physiques_
Elles utilisent une caractéristique physique, typiquement une empreinte digitale, 
- côté _serveur_ elle est enregistré dans le document _user_, 
- côté application terminale elle est donnée par un lecteur d'empreinte.

Il est concevable d'enregistrer plusieurs empreintes pour un même _user_ autorisant ainsi plusieurs personnes physiques à être authentifiables.

**Limitations:**
- tous les appareils n'ont pas de lecteur d'empreinte.
- il faut avoir enregistré **physiquement** d'avance toutes les empreintes ayant droit d'accès: on ne peut pas ajouter à distance un droit d'accès à une personne dûment authentifiée dans la vraie vie.

### _userId / password_ versus _passphrase_
**Dans le mode _login / password_**, l'utilisateur connaît au moins un identifiant externe unique (_login_) du document _user_.
- la donnée du _login_ aboutit à **un** _user_ (ou aucun);
- la donnée par l'utilisateur d'un _password_ lui permet de se faire authentifier par comparaison avec celui ou ceux enregistrés dans le _user_.

**Limitations:**
- le **login** est souvent une donnée de la vie réelle: e-mail ou numéro de téléphone. Ceci compromet la confidentialité personnelle des personnes physiques. Mais ce peut être aussi un _pseudo_ libre (mais unique).
- par négligence le **password** est trop court et peut être cassé soit par force brute, soit par usage de dictionnaires des passwords usuels.

**Dans le mode _passphrase_** une phrase longue (unique sur le serveur) effectue à la fois l'identification du user et son authentification.
- la passphrase peut être changée (sous condition).
- sa longueur (32 signes au minimum par exemple) empêche les hackers de les retrouver par force brute et les _dictionnaires_ sont inopérants.

**Limitations:**
- **si la passphrase a été générée aléatoirement**, sa sécurité est maximale mais de facto reportée sur l'utilisateur qui sauf pour les hypermnésiques sont incapables de mémoriser une telle clé. L'utilisateur va écrire celle-ci sur un support quel qu'il soit comportant un risque de diffusion / vol involontaire.
- **si la passphrase a été fixée par l'utilisateur** elle peut être longue MAIS correspondre à quelque chose de **signifiant / mnémotechnique** pour lui: 

      les$feuilles$mortes$l_ours_et_le_singe_animaux_sages

Les références poétiques ne sont pas exceptionnlles et simples à se souvenir mais un algorithme ne peut les essayer toutes en variant les séparateurs.

Un doublon de _passphrase_ n'est pas à exclure quand elle est déclarée par un humain. Or tomber par hasard sue phrase déjà enregistrée ouvre automatiquement l'accès à un _user_ qui n'est pas le sien: il faut donc interdire les doublons sur _une partie de la phrase_, par exemple les 16 premiers catactères (ou une primère ligne d'une phrase à 2 lignes), ce qui incite à choisir une seconde moitié sans rapport avec la première.

> La limitation évidente est que saisir une telle phrase à chaque connexion est pénible.

### Les authentifications à double facteur (2FA)
On connaît son _user_ par un _login_: le concept 2FA vise à authentifier l'utilisateur qui a identifié son compte.

#### Distributeur de mots de passe à durée de vie limitée
L'utilisateur installe Une application 2FA et lui fait  enregistrer les couples _application / userId_ auxquels il est susceptible de se connecter.

Juste avant de se conecter, l'application 2FA est ouverte et lui affiche les _passwords_ utilisables pour chaque couple _application / userId_: ça laise moins d'une minute à l'utilisateur pour le saisir.

**Limitations**
- si on se fait voler son matériel, ou pire **UN des matériels** sur lesquels l'application 2FA à été installée (et configurée) et que le hacker peut accéder à tous les logins enregistrés.
- certaines applications 2FA limitent le risque en demandant de saisir un code PIN: ceci rallonge la procédure. Toutefois ce code PIN est stocké sur l'appareil, donc pas à l'abri d'un casse technique.

#### Envoi d'un code sur un autre appareil
Soit un mot de passe temporaire généré par le serveur, soit une URL à cliquer pour valider l'accès (par SMS ou e-mail).

La sécurité repose sur le fait que l'utilisateur est le **seul** à pouvoir lire ses SMS ou e-mails.

Dans ce cas le hacker ne pourra pas recevoir le code ou l'URL de validation.

**Limitations:**
- numéro de téléphone ou e-mail sont nécessaires: ceci compromet la confidentialité personnelle des personnes physiques, un _user_ étant toujours porteur d'une réfrence dans le vrai monde (ce qui exclut les applications volontairement anonymes).
- si le mobile de l'utilisateur est volé, le voleur voit passer les SMS ou a accès aux e-mails. Certes il doit se connecter (quoi que pour un SMS ?) mais il semble que ce n'est pas si complexe que cela (du moins pour les policiers).
- si le mobile est _perdu_, la procédure peut être complexe et a minima longue.

# Recherche d'une _moins mauvaise_ solution
### Solution de base
Le choix de la _passphrase_ longue à fixer par l'utilisateur coche les cases de la sécurité et de l'universalité: si les autres options de connexion _simplifiée_ échouent, il reste toujours à l'utilisateur la possibilité de citer sa _passphrase longue_.

### Options de login _rapides_
Elles font appel au principe d'une application 2FA qui s'exécute et délivre à l'application cherchant à se connecter la _passphrase_ complète. Cette application pouvant s'exécuter,
- sur le même appareil `dev1` que l'application demandant le login,
- sur un AUTRE appareil `dev2`.

Une application 2FA sur un appareil doit être configuré d'avance pour tous les logins pouvant lui être présentés afin de transmettre à l'application demanderesse la _passphrase_ souhaitée.

Pour que ce soit rapide l'application 2FA ne devrait pas demander elle-même un code PIN long:
- Cas 1: 2FA ne demande pas de code PIN.
  - si dev1 est volé / perdu, toute personne qui pouura s'y connecter accèdera à tout ce qu'elle veut.
- Cas 2: 2FA demande un code PIN court.
  - il faut que la seconde tentative infructueuse de saisie du code PIN réinitialise 2FA en lui faisant perdre tous les logins qu'elle a enregistrés.

### Cas à envisager
Soit devX l'appareil _externe_ d'un hacker.
- dev1 et dev2 sont les mêmes.
- dev2 est perdu / volé: bref l'utilisateur N'A PAS dev2 sous la main pour agir: une demande issue de devX peut-elle être acceptée par dev2 ?
- dev1 et dev2 ont été volés par un hacker: peut-il se connecter ?
- le vol de dev1 ou dev2 donne-t-il la possibilité au voleur d'avoir accès aux _passphrases_ des users, le cas échéant par des moyens logiciels évolués ?
- la possession d'un numéro de téléphone est-elle requise ou une facilité éventuelle ?
