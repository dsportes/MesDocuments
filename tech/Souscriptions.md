---
layout: page
title: Souscriptions des sessions
---

## Souscriptions
Une session est identifiée de manière unique par son jeton de web-push `wpToken`: dans un serveur `sessionId` est un entier JS `hashInt(wpToken)`.

### Souscription élémentaire: _définition_
Une souscription élémentaire est:
- soit la souscription à la collection des documents de la classe: `Article:`.
- soit une souscription à UN document de la classe par sa clé primaire_. Par exemple `Article.pk:FR/3246`, _L'Article_ d'identifiant FR/3246.
- soit une souscription à une sous-collection de documents synchronisable : _classe du document . sous-collection : valeur de la clé_. Par exemple `Article.auteurs:a7689`, _les_ Articles ayant pour un des auteurs a7689.

La _définition_ d'une souscription élémentaire est un code de forme générale `Classe.clé:val`.
- `ids` d'une définition: shaInt de sa définition.
- la définition d'une souscription pouvant être un texte long quand le serveur renvoie par _notification_ une liste de souscriptions, il renvoie une liste d'entiers beaucoup plus compacte.

Les valeurs des clés sont des strings pouvant contenir des /.

### Objet souscription
Il comporte les éléments suivants:
- `org` : code l'organisation.
- `wpToken` : le token de la session.
- `sessionId` : son hashInt.
- `version` : date-heure de déclaration.
- `defs` : une map `{ ids: [def, msg] }` donnant en `ids` le shaInt du `def` représentant le code d'une souscription élémentaire:
  - `def` est le codage de sa définition.
  - `msg` est un message facultatif.

Une session _s'abonne_ à des _synchronisations / notifications_ en soumettant une opération `Subscribe` ayant en paramètre l'objet de souscription. Si `defs` est absent, la session est désabonnée.

#### Mise à jour de la base de données
Un document `Subs` mémorise la souscription globale:
  - `sessionId` est la clé primaire.
  - `data` est la sérialisation de l'objet souscription.

Une table annexe `Defs` : insertion / suppression pour chaque élément de `defs` d'un row `{ (org), sessionId, def }`.
- après lecture de la version déjà enregistrée et comparaison avec la nouvelle version, les souscriptions élémentaires déjà présentes sont inchangées, les nouvelles ajoutées et les autres supprimées.

## Notification des sessions en fin d'opération
Une opération peut mettre à jour des documents, par exemple des `Articles` et après _commit_ va émettre des notifications à toutes les sessions abonnées.

Pour chaque document de clé primaire pk mis à jour, il faut notifier les sessions:
- de la mise à jour du document,
- du changement des sous-collections.

Exemple 1:
- mise à jour d'un document Article : `{ pk: FR/3246, auteurs: [a7689],` data: ... }
  - la propriété `auteur` avait la valeur `a7689` et la conserve.
- les souscriptions élémentaires à notifier sont :
  - `Article:FR/3246`
  - `Article.auteur:a7689` : la liste des Articles est inchangée mais la Article FR/3246 a changé de contenu.

Exemple 2:
- mise à jour d'un document Article : `{ pk: FR/3246, auteurs: [a7689, a8887], data: ... }`
  - la propriété `auteurs` avait la valeur `a7689` et prend la valeur `a8887`.
- les souscriptions élémentaires à notifier sont :
  - `Article:FR/3246`
  - `Article.auteurs:a7689` : la liste des Articles est changée, déjà a priori réduite par l'évolution de la Article FR/3246 (mais peut-être d'autres)
  - `Article.auteurs:a8887 `: la liste des Articles est changée, déjà a priori augmentée par l'évolution de la Article FR/3246 (mais peut-être d'autres)

### Process
#### Étape 1
Une map des sessions à notifier est créée avec:
- pour clé: le sessionId d'une session à notifier,
- pour valeur: le Set des ids des définitions à notifier.

Depuis la liste des documents changés on calcule la liste des définitions des souscriptions élémentaires impactées. Pour chacune `def` de celles-ci,
- on obtient de la table `Esubs` la liste des `sessionId` qui référence `def`. Pour chaque sessionId,
  - on obtient le document de souscription.
  - on obtient l'élément la map des sessions à notifier de clé sessionId,
  - on ajoute l'ids shaInt de `def`.

#### Étape 2
Pour chaque session identifiée par sessionId à notifier on dispose du Set des `ids` à _synchroniser_.

Pour pousser une notification à la session il faut obtenir son `wpToken` (dans son document de souscription) depuis son `sessionId`.

Chaque session concernée recevra en conséquence une liste [ids] qui chacun est le shaInt d'une définition: la session retrouve la définition correspondante et pourra en obtenir si souhaité les synchronisations incrémentales ou intégrales depuis la version connue en session (s'il y en a une).

## Notifications avec _messages_
Les souscriptions élémentaires avec pour unique objectif de _synchroniser_ des documents n'ont pas de message: la session réceptrice ne fait pas apparaître de pop-up qui, du fait de leur nombre potentiellement important, seraient inopportunes.

Certaines souscriptions ont pour but de provoquer l'affichage d'une pop-up pour alerter l'utilisateur: ce peut être le cas quand l'application n'est PAS lancée. 
- la donnée de _synchronisation_ (la liste des `ids`) est inutilisée, non transmise à l'application.
- en revanche la notification peut porter un _message_, un texte généré sur le serveur.

### Message associé à UNE souscription élémentaire
La souscription peut renseigner pour chaque souscription élémentaire le texte du message qui sera émis. Par exemple pour `Article.auteur/Hugo` le message `"Maj d'un article de Hugo"`.

Une notification peut avoir plusieurs messages: ils sont concaténés sur des lignes séparées.

### Messages génériques 
Un tel message est déclaré pour s'appliquer à TOUTES les souscriptions élémentaires d'une même _classe_, c'est à dire de la partie avant le '/' dans sa description.

Dans l'objet souscription une map `msgGen` comporte des éléments ayant:
- pour _clé_ la classe: par exemple `Article.auteur`.
- pour _valeur_ le message, par exemple `"Maj d'un article"`.

Si une notification élémentaire `Article.aut/a7689` est à produire,
- si `Article.auteur/Hugo` a un _message_ spécifique, il est concaténé dans le message à afficher,
- sinon, 
  - si `Article.auteur` a une entrée dans `msgGen` c'est ce message qui est concaténé : `"Maj d'un article"`.
  - sinon, rien n'est concaténé.

_Remarque_: un même texte n'est jamais concaténé plus d'une fois.

## Documents en base de données: _historique_ des versions
Exemple:
La classe de document `Article` a les propriétés suivantes:
- `pk` : numéro identifiant de la Article.
- `auteur` : identifiant de son auteur (ici on suppose qu'un).
- `sujet` : identifiant du sujet dans la liste des sujets répertoriés.
- `data` : texte, fichiers attachés, etc.

On peut s'abonner à :
- la collection de tous les articles: `Article:`
- un article spécifié : `Article.pk:FR/3246`
- la collection des Articles d'un auteur `Hugo` donné: `Article.auteur/Hugo`
- la collection des Articles d'un sujet `écologie` donné: `Article.sujet/écologie`

### Cas simple
Les propriétés `auteur` et `sujet` sont _immuables_, ne peuvent pas changer pour un article donnée après sa création.

#### Problème de la _suppression_ de l'article `FR/3246`
L'article n'est pas _purgé_ mais _zombifié_:
- sa version `v` est celle de l'opération.
- son indicateur `z` est mis à la date du jour (les 5 premiers chiffres de la version), il indique que l'article n'existe plus _logiquement_.

La session a conservé des versions:
- `v[Article.pk:FR/3246]` : v1 la version du dernier article `FR/3246` obtenu du serveur.
- `v[Article.auteur/Hugo]` : v2 la plus haute des versions des articles retournés après sélection des Articles `where aut == 'Hugo'`.

La synchronisation par `Article.pk:FR/3246`: `where pk == FR/3246 and v > v1`:
- retourne la version vx marquée _zombi_.

La synchronisation par `Article.auteur/a7689`: `where aut == 'Hugo' and v > v2`:
- retourne la version vx marquée _zombi_. L'article FR/3246 est enlevé de la liste en mémoire `Article.auteur/Hugo`.

### Cas général
Les propriétés `auteur` et `sujet` NE sont PAS _immuables_ et peuvent changer pour un article donné après sa création.

> En plus du problème de la suppression, se pose celui du changement d'auteur et / ou de sujet.

Quand l'une au moins des propriétés `auteur` et `sujet` change de valeur, il est inséré un second row avec:
- la même version,
- l'indicateur `del` à true,
- les valeurs _avant_ de `auteur` et `sujet` quand elles ont changé,
- un `data` restreint au propriétés, `v`, `del` et de la clé primaire comme pour un _zombi_.

Exemple: l'article `Article:FR/3246` change d'auteur Hugo -> Victor mais pas de sujet.

    (1) v:...17            pk:FR/3246 auteur:Victor sujet:écologie 
    (2) v:...17 del z:..32 pk:FR/3246 auteur:Hugo        

La synchronisation par `Article:FR/3246`: `where pk == FR/3246 and v > v1`:
- retourne (1) : la ligne (2 del) est ignorée pour un accès primaire à un article.

La synchronisation par `Article.auteur/Hugo`: `where aut == Hugo and v > v2`:
- NE retourne pas la ligne (1), l'auteur n'est pas Hugo.
- retourne la ligne (2) avec `del` ce qui indique qu'il faut retirer FR/3246 de la sous-collection.

La synchronisation par `Article.sujet/écologie`: `where sujet == écologie and v > v2`:
- retourne la ligne (1), l'article est mise à jour dans la liste.
- NE retourne PAS la ligne (2) qui n'indique pas un sujet _écologie_.

La session terminale peut ainsi maintenir à jour:
- Article FR/3246
- ses deux sous-collections `Article.auteur/Hugo` et `Article.sujet/écologie`.

#### Purge des lignes `del`
Le fait de marquer un jour de _zombification_ sur une ligne permet quelques mois après ce jour de purger par une tâche d'administration celles-ci supposées ne plus avoir d'utilité, toutes les sessions ayant été mises à niveau incrémentalement (ou ayant subi un rechargement intégral).

## Propriétés de type _liste_
Dans l'exemple `Article`, la propriété `auteurs` peut être une liste d'auteurs `auteurs` de dimension modeste.

La synchronisation par `Article.auteurs/Hugo` s'écrit avec une clause différente: `where auteurs array-contains Hugo and v > v2`:
- elle retourne toutes les Articles dont l'un des auteurs cités dans `auteurs` vaut Hugo.
- si cette liste contient `[a1, a2, a3]` le process de notification doit rechercher l'existence de souscriptions élémentaires `Article.auteurs/a1` `Article.auteurs/a2` `Article.auteurs/a3`, bref tous les _abonnements_ à un des auteurs cités: c'est pour cela que cette liste doit rester modeste.
- quand cette liste évolue et que des auteurs par exemple `a2` et `a3` sont retirés les lignes `del` sont à inscrire **pour chacune des valeurs** `a2` et `a3`.

Remarque : `del` doit il être systématique -> 0:courant, 1:historique ?
- on ne peut pas on utiliser `z` sinon l'index de `z` va être énorme.
- on pourrait utiliser un index composite `pk / v / del`.
- `orderBy v` et `orderBy del` avc `limit 1` pour accès par `pk`.
