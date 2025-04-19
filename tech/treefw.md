---
layout: page
title: Réflexions à propos d'un framework "TreeFw"
---

# Arbres de documents
## Documents, classes et classes majeures
Une instance de document contient un certain nombre de données structurées selon sa _classe_, qui peut être soit _classe majeure_, soit une sous classe de sa class majeure.

> Par exemple la classe majeure `Account` peut avoir des sous-classes `AccountAdmin` `AccountStandard`. Le **type d'un document** est le nom de sa classe majeure.

Toutes les classes majeures sont des sous-classes des classes abstraites `Document` ou `Singleton` (elle-même sous-classe de `Document`).

### Sérialisation / désérialisation
Les instances de document sont _sérialisables_ : leurs données peuvent être _packées_ en un binaire par convention appelé _data_ du document.

## Arbres
Un arbre est une instance d'une classe Tree avec les propriétés suivantes:
- `type`: type d'arbre, un code identifiant la nature de l'arbre. La liste fermée des types est déclarée dans l'application.
- `tid` : ID de l'arbre (un string court libre).

Un arbre est une collection de documents:
- des singletons: chaque document est identifié dans son arbre par son **type de document**.
- des collections typées de documents _NON_ singleton. Chaque document est identifié dans son arbre par,
  - `td` : son type de document (sa _classe majeure_, pas sa classe effective détaillée).
  - `did` : son ID de document dans l'arbre (un string court). Cet ID peut être par exemple de la forme `A1.B1.C1` définissant de facto une arborescence fonctionnelle. 

## Persistance en base de données
Un arbre est stocké sous la forme d'un _record_ accessible par sa clé / path `type/tid`

Un document est toujours rattaché à un arbre: il est accessible par sa clé / path :
- `type/tid/td/1` : pour un type de document singleton.
- `type/tid/td/did` : pour un type de document NON singleton.

## Fichiers
Un fichier est persisté en deux parties:
- un descriptif attaché à un arbre (donc en base de données),
- son contenu effectif stocké dans un Storage (de type S3).

Son descriptif est accessible depuis son arbre par sa clé / path `type/tid/file/did`

Son contenu effectif est identifié par une clé unique `fid` et accessible en Storage sous le path `type/tid/fid`.

Le descriptif en base possède a minima les propriétés:
- `v` : son numéro de version dans son arbre,
- `fid` : son `fid` en Storage.

La mise à jour d'un fichier `type tid "file" id` dans son arbre est une opération double:
- la création d'un nouveau contenu et la génération d'un nouvel identifiant `fid2`: c'est une opération `preLoad` qui charge dans le storage ce contenu et lui attribue son identifiant de Storage.
- la validation de ce changement dans le cadre d'une transaction de la base de données remplaçant dans son descriptif le fid par le nouveau `fid2`. 

# Objectifs du framework
## Gestion de la persistance _centrale_
Le framework gère le stockage des documents et fichiers dans le couple base de données qui Storage détient _l'original_ des **documents et fichiers regroupés en arbres** comme expliqué ci-avant.
- cette gestion est sécurisée par des transactions ayant la propriété ACID du SGBD.
- les contenus des fichiers créés / modifiés par une transaction doivent avoir fait l'objet d'un preLoad en Storage AVANT mise à jour par une transaction. Le framework assure le nettoyage des contenus ayant fait l'objet d'un preLoad sans avoir finalement été utilisé par une transaction de mise à jour.

## Gestion de la synchronisation de copies distantes des arbres
Des applications peuvent détenir des _copies partielles_ des arbres stockés en central:
- seulement certains arbres cités par leurs identifiants;
- dans ces arbres, seulement les types de documents souhaités.
