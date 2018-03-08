# Premiers pas

## Sommaire :
### [Index](#user-content-index)
*  [Création d'un Index](#user-content-créer-un-index)
*  [Modification/Suppression d'un Index](#user-content-modifiersupprimer-un-index)

### [Documents](#user-content-documents)
*  [Création d'un Document](#user-content-remplir-un-index-de-documents)
*  [Modification d'un Document](#user-content-modifier-les-documents-dun-index)
*  [Suppression d'un Document](#user-content-supprimer-les-documents-dun-index)

### [Recherche](#user-content-recherche)
*  [Recherche Globale et par ID](#user-content-globale-et-par-id)
*  [Recherche par Requête](#user-content-par-requête)

# Index 

## Créer un index

**Vocabulaire :** Mapping

Le Mapping c'est l'action de définir spécifiquement la structure d'un Index.

**Exemple :** 

![IndexPUT](/uploads/328406612f6e110fe17a6559780e6c74/IndexPUT.png)

*  1 - On indique que l'on souhaite créer l'Index "sandwich" 
*  2 - On indique que l'on souhaite façonner sa structure avec l'objet "mappings"
*  3 - On définit le type de l'Index, qui sera toujours "_doc"
*  4 - On indique que des propriétés vont exister
*  5,6 - On inscrit le nom des propriétés, et leur type. 

Ce procédé est une bonne pratique à avoir, car il est possible de créer des Index sans structure construite au préalable. Si vous avez du mal à comprendre, dites vous que c'est presque la même chose que déclarer une table SQL et ses champs.

## Modifier/Supprimer un index 

La modification d'un Index ne peut pas se faire directement.

Pour "modifier" un Index, il faut en créer un nouveau qui correspond à ce que l'on souhaite et :
*  Copier coller les documents de l'ancien Index vers le nouveau
*  Ou, implémenter les documents dans le nouvel Index

Enfin, supprimer l'ancien Index. 

Le code le plus basique pour Copier/Coller des documents d'un Index à un autre est le suivant :

![IndexREINDEX](/uploads/bb9952c216627fca5d275cb60823000f/IndexREINDEX.png)

Vous trouverez d'autres méthodes plus pointues dans la [documentation officielle]().

Le code ci-dessous permet de supprimer un Index 

![IndexDELETE](/uploads/74b82cae725c5dddaa29e36ed88d5f72/IndexDELETE.png)

# Documents

## Remplir un index de documents 

*Notions abordées : 
-> Insertion par ID
-> Insertion de masse API bulk*

## Modifier les documents d'un index 

*Notions abordées :
-> Modification par ID
-> Modification de masse*

## Supprimer les documents d'un index 

*Notions abordées :
-> Suppression par l'ID
-> Suppression de masse*

# Recherche 

## Globale et Par ID

Notions abordées : 
-> Recherche API Search de base

## Par Requête

Notions abordées : 
-> Fuzyness
-> Wildcard
-> Multi searching
-> Analyzer
-> Aggregation
etc etc

# Redémarrer ElasticSearch