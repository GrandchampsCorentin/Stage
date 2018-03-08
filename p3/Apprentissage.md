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

**Le code le plus basique pour Copier/Coller des documents d'un Index à un autre est le suivant :**


![IndexREINDEX](/uploads/bb9952c216627fca5d275cb60823000f/IndexREINDEX.png)

Vous trouverez d'autres méthodes plus pointues dans la [documentation officielle](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html).

**Le code ci-dessous permet de supprimer un Index**

![IndexDELETE](/uploads/74b82cae725c5dddaa29e36ed88d5f72/IndexDELETE.png)

# Documents

## Remplir un index de documents 

*Notions abordées : 
-> Insertion par ID
-> Insertion de masse API bulk*

### Insertion par ID

L'insertion par ID est simple, en voici un exemple : 

![DocPUT](/uploads/fbc66b921385e81979f288a1257d0026/DocPUT.png)

*  1 - On indique le chemin sous la forme /index/type/id en proposant un ID valide qui sera celui du nouveau document
*  2 - On rentre les valeurs associées à leur champ

Il est possible de faire une insertion sans préciser l'id (/index/type/), un id sera automatiquement associé au document, mais il sera long, peu significatif, et compliqué à utiliser (ex : "6a8ca01c-7896-48e9-81cc-9f70661fcb32")

### Insertion de masse

Rentrer un à un les documents dans les index peut s'avérer long. Pour éviter de devoir tout faire par l'ID il existe une API dénommée Bulk dans ElasticSearch qui permet d'envoyer un groupement de documents d'un seul coup. 

Voici un exemple :

![DocBULK](/uploads/76af3435d7fb0cb8f0f00e25dff695af/DocBULK.png)

*  1 - On appelle l'API Bulk avec "_bulk"
*  2 - En une ligne on définit 
   1.  l'action que doit faire la requête, ici `"create"` pour la création,
   2.  l'index où elle doit avoir lieu, ici `"_index" : "sandwich"`,
   3.  le type de l'index, ici `"_type" : "_doc"`,
   4.  l'id du document que cette ligne traite, ici `"_id" : "1"`.
*  3 - Après un saut de ligne, on indique les valeurs des champs à indexer tels que `"name", "ingrédients", "origine"` etc...

Il est **impératif** de :

*  renseigner l'action sur une ligne, sauter la ligne, renseigner les valeurs comme dans l'exemple
*  Tout mettre en ligne. Le saut de ligne étant utilisé spécialement.
*  Ne pas oublier le dernier saut de ligne à la fin de la dernière ligne.

Le terme "create" peut être remplacé par "index". 

Le premier reporte une erreur dans le cas où le document qu'il tente de créer existe déjà dans l'index.

Le second s'occupe d'ajouter ou de remplacer le document si nécessaire.
 

Il est possible d'appeler un fichier plutôt que de tout rentrer à la main dans l'interpréteur de requêtes. Mais n'ayant pas réussi à utiliser cette fonctionnalité, je ne la décrirai pas ici.

Pour plus de renseignements sur l'API Bulk, jetez un oeil sur la [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html).

## Modifier les documents d'un index 

### Modification sur un document

Pour modifier un document, rien de plus simple ! 

[DocPOST](/uploads/7d91bdc56cd7b70ed9fd19936e3652f7/DocPOST.png)

*  1 - L'url doit être au format `POST /index/type/id/_update` le "_update" appelant l'API de mise à jour
*  2 - Il faut ajouter l'objet "doc" pour préciser quel(s) champ(s) modifier
*  3 - La liste des champs à modifier et leur nouvelle valeur.

Il existe un autre terme utilisable à la place de "doc" qui est "script" et qui permet d'implémenter des petits scripts tel qu'un compteur ou autre, les détails sont dans la [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html).

### Modification sur une masse de documents

L'API Bulk propose d'autres actions que le "create" ou l'"index" telles que l'"update" et le "delete". 

Pour faire une modification de masse, la sémantique est très proche d'un "create" : 

`{ "update" { "_index" : "mon_index, "_type" : "_doc", "_id" : "mon_id" } }`*Ne pas oublier le saut de ligne ici*

`{ "doc" : { "champ1" : "valeur1", "champ2" : "valeur2" } }`*Ne pas oublier le saut de ligne ici*

Le terme "doc" est rajouté ainsi que les accolades.

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