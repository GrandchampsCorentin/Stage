# L'outil qui facilite la vie

*  Comme vous êtes des utilisateurs de Google Chrome, il existe une extension qui propose une interface graphique de gestion et de traitement des Clusters, des Nodes, des Index et des Documents existant, un peu comme un phpMyAdmin. 

Vous pourrez vous la procurer ici : [Elasticsearch Head](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm)

L'extension n'est pas design mais permet d'avoir un visuel sur ses instances ES. 

# Astuces

**Globale**  
* La relation des fichiers avec Laravel : 1 IndexConfigurator <=> 1 Modèle <=> 1 Index. 

**Modele**
* Un modèle peut à la fois être affilié à la recherche avec Elasticsearch et associé à une BDD avec Eloquent ou QueryBuilder

**Type**
* Un index a un unique type. Elasticsearch préconise d'appeler ce type "_doc". Nous n'avons aucune main mise sur le nom du type d'un index créé par Laravel, mais ce n'est pas important.   
*Les types seront dépréciés au passage de la version 7 d'Elasticsearch.*

**Alias**
* Les Alias permettent de faire des modifications d'Index sans interruption de service. 

**Mapping**  
* Il est possible d'ajouter des champs à un mapping existant et de mettre à jour l'index concerné. 
* Il n'est pas possible de modifier/supprimer des champs d'un mapping sans supprimer et recréer l'index concerné. 
* Il est possible de ne pas spécifier de mapping dans un modèle.  
Elasticsearch peut se débrouiller seul. Mais ce n'est pas conseillé. 
* Il est possible de spécifier des boosts dans le mapping et dans la recherche.  
**Il est préférable de spécifier les boosts au moment de la recherche.**  
* Faites attention aux types que vous utilisez pour vos champs d'index dans le mapping.   
En effet le type "keyword" n'est pas analysable et toute recherche sur un champ "keyword" doit avoir une correspondance à 100% avec la chaine de caractère enregistrée.  

**Analyzer**  
* Les analyzers se construisent dans les IndexConfigurators.
* Il est préférable de paramétrer et d'utiliser le même analyzer lors de la création de l'index (et donc de l'indexation des données) et de la recherche de données sur ce même index. 
* Il est tout à fait possible de construire et de définir un ou plusieurs analyzer et de leur ajouter autant de filtres que souhaité parmi ceux existant.

