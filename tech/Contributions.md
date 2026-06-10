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

# Mode _veille_
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
  - si dev1 est volé / perdu, toute personne qui pourra s'y connecter accédera à tout ce qu'elle veut.
- Cas 2: 2FA demande un code PIN court.
  - il faut que la seconde tentative infructueuse de saisie du code PIN réinitialise 2FA en lui faisant perdre tous les logins qu'elle a enregistrés.

### Cas à envisager
Soit devX l'appareil _externe_ d'un hacker.
- dev1 et dev2 sont les mêmes.
- dev2 est perdu / volé: bref l'utilisateur N'A PAS dev2 sous la main pour agir: une demande issue de devX peut-elle être acceptée par dev2 ?
- dev1 et dev2 ont été volés par un hacker: peut-il se connecter ?
- le vol de dev1 ou dev2 donne-t-il la possibilité au voleur d'avoir accès aux _passphrases_ des users, le cas échéant par des moyens logiciels évolués ?
- la possession d'un numéro de téléphone est-elle requise ou une facilité éventuelle ?

## Questions ouvertes

### Comment éviter une inflation incontrôlable de création de _safes_ fantômes
Un utilisateur ne pourrait créer un _safe_ qu'après avoir obtenu un ticket d'invitation déposé par un autre _utilisateur_.
- le ticket a une durée de vie limitée.
- le code du ticket parvient par un moyen externe (mail ...).
- le nombre de tickets généré par un utilisateur est limité (N par mois / an ...).

Un _safe_ n'ayant ni credentials ni invitations en cours pourrait avoir une vie brève: ça ne dissuade pas d'en créer mais au moins ils s'auto-détruisent.
- bref un safe qui vient d'être créé est _en sursis_ tant qu'il a déposé une demande d'invitation et n'a pas pu valider une acceptation. 
- après, il reste soumis à la règle du dernier mois d'accès + N.

### Garbage Collector des documents
Solution générique à rechercher.
- en partie liée, peut-être, avec le décompte de la consommation des ressources ?
