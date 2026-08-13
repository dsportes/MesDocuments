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
- elles disparaissent si la bse est réinitialisée.

