---
layout: page
title: Souscriptions des sessions
---

## Souscriptions
Une session est identifiée de manière unique par son jeton de web-push `subsJSON`: `sessionId` est le shaS de ce jeton.

### Souscription élémentaire: _définition_
Une souscription élémentaire est:
- soit la souscription à la collection des documents de la classe: `Article`.
- soit une souscription à UN document de la classe par sa clé primaire_. Par exemple `Article/shaS(FR/3246)`, _L'Article_ d'identifiant FR/3246.
- soit une souscription à une sous-collection de documents synchronisable : _classe du document . sous-collection : valeur de la clé_. Par exemple `Article/auteurs/shaS(a7689)`, _les_ Articles ayant pour un des auteurs a7689.

### Objet souscription
Il comporte les éléments suivants:
- `sessionId` : son shaS.
- `subsJSON` : le token de la session.
- `org` : code l'organisation.
- `title` : (facultatif) titre des notifications textuelles.
- `url` : (facultatif) url d'ouverture de l'application sur click d'une notification textuelle.
- `v` : date-heure de déclaration.
- `defs` : une map `{ def1: msg1, def2: msg2 ... }` donnant pour chaque définition de souscription élémentaire son _message_ textuel à afficher dans la notification ou '' s'il n'y en a pas.

Une session _s'abonne_ à des _synchronisations / notifications_ en soumettant une opération `CreateSubscription` ayant en paramètre:
- l'objet de souscription, dont son organisation: il doit être émis une souscription par organisation.
- le booléen `longLife`: si true la souscription reste valable quelques jours en l'absence de session active, sinon quelques heures. En fin de session, l'utilisateur décide s'il souhaite ou non continuer à recevoir des notifications textuelles.

Les opérations `DeleteSubscription UpdateSubscription` gèrent les souscriptions en cours de session.

#### Mise à jour de la base de données
Un document `Subs` mémorise la souscription globale:
- `sessionId` est la clé primaire.
- `data` est la sérialisation de l'objet souscription.

Des documents annexes `SubsItem` sont gérés pour chaque élément de `defs` d'un row `{ (org), sessionId, def }`.
- après lecture de la version déjà enregistrée et comparaison avec la nouvelle version, les souscriptions élémentaires déjà présentes sont inchangées, les nouvelles ajoutées et les autres supprimées.
- un index sur `def` permet de récupérer toutes les sessions ayant cette définition et de les notifier.

## Notification des sessions en fin d'opération
Une opération peut mettre à jour des documents, par exemple des `Article` et après _commit_ va émettre des notifications à toutes les sessions abonnées.

Pour chaque document de clé primaire pk mis à jour, il faut notifier les sessions:
- de la mise à jour du document,
- du changement des sous-collections.

Exemple 1:
- mise à jour d'un document Article : `{ pk: FR/3246, auteurs: [a7689],` data: ... }
  - la propriété `auteurs` avait la valeur `a7689` et la conserve.
- les souscriptions élémentaires à notifier sont :
  - `Article/shaS(FR/3246)`
  - `Article/auteurs/shaS(a7689)` : la liste des Articles est inchangée mais la Article FR/3246 a changé de contenu.

Exemple 2:
- mise à jour d'un document Article : `{ pk: FR/3246, auteurs: [-a7689, +a8887], data: ... }`
  - la propriété `auteurs` avait la valeur `a7689` et prend la valeur `a8887`.
- les souscriptions élémentaires à notifier sont :
  - `Article/shaS(FR/3246)`
  - `Article/auteurs/shaS(a7689)` : la liste des Articles est changée, déjà a priori réduite par l'évolution de la Article FR/3246 (mais peut-être d'autres).
  - `Article/auteurs/shaS(a8887)`: la liste des Articles est changée, déjà a priori augmentée par l'évolution de la Article FR/3246 (mais peut-être d'autres)

### Process
#### Étape 1
Une map des sessions à notifier est créée avec:
- pour clé: le sessionId d'une session à notifier,
- pour valeur: le Set des définitions à notifier.

Depuis la liste des documents changés on calcule la liste des définitions des souscriptions élémentaires impactées. Pour chacune `def` de celles-ci,
- on obtient de `SubsItem` la liste des `sessionId` qui référence `def`. Pour chaque sessionId,
  - on obtient le document de souscription.
  - on ajoute le `def` et son _message_ éventuel.

#### Étape 2
Pour chaque session identifiée par sessionId à notifier on dispose du Set des `def` à _synchroniser_.

Pour pousser une notification à la session il faut obtenir son `shaJSON` (dans son document de souscription) depuis son `sessionId`.

Chaque session concernée recevra:
- une liste des [def] ayant changé,
- un _texte de message_ (éventuellement vide) concaténation de tous les messages des `def` notifiées.

## Notifications avec _messages_
Les souscriptions élémentaires avec pour unique objectif de _synchroniser_ des documents n'ont pas de _texte de message_: la session réceptrice ne fait pas apparaître de pop-up qui, du fait de leur nombre potentiellement important, seraient inopportunes.

Certaines souscriptions ont pour but de provoquer l'affichage d'une pop-up pour alerter l'utilisateur: ce peut être le cas quand l'application n'est PAS lancée. 
- la donnée de _synchronisation_ (la liste des `def`) est inutilisée, non transmise à l'application (qui n'est pas exécution).
- en revanche la notification peut porter un _texte de message_ généré sur le serveur.
- le _titre_ de ce message est le nom de l'application et de l'organisation concernée.
- _l'url_ de ce message permet d'ouvrir l'application sur un click de l'utilisateur sur la notification: cette url peut être munie d'un _query string_ qui détermine (éventuellement) les conditions d'ouverture de l'application (son affichage initial et ses documents à charger)
.
### Message associé à UNE souscription élémentaire
La souscription peut renseigner pour chaque souscription élémentaire le texte du message qui sera émis. Par exemple pour `Article.auteurs/shaS(Hugo)` le message déclaré par l'application terminale pourrait être `"Maj d'un article de Hugo"`.

Le message suit les règles d'I18N de la session et c'est l'application terminale qui dspose des _noms en clair_ et autres données et sait lesquelles il convient d'afficher dans un pop-up de notification.

Une notification peut avoir plusieurs messages: ils sont concaténés sur des lignes séparées.
