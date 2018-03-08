# Premiers pas

## Sommaire :
### [Index](#user-content-index)
*  [Création d'un Index](#user-content-créer-un-index)
*  [Modification d'un Index](#user-content-modifier-un-index)
*  [Suppression d'un Index](#user-content-supprimer-un-index)

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
*  4 - On définit les propriétés de l'Index
*  5,6 - On inscrit le nom des champs, et leur type. 

Ce procédé est une bonne pratique à avoir, car il est possible de créer des Index sans structure construite au préalable. Si vous avez du mal à comprendre, dites vous que c'est presque la même chose que déclarer une table SQL et ses champs.

## Modifier un index 

*Notion abordée : 
-> Les différentes façons de modifier un index*

## Supprimer un index

*Notion abordée : 
-> Suppression d'un index*

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