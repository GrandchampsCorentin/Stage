# Liste des commandes sur console 

Toutes les commandes "artisan" sont listées ci-dessous :  

| Command	| Arguments	| Description |  
| :-----------: | :-----------: | :---------: |  
| `make:index-configurator`	| `name` - Le nom de la classe	| Créé un nouvel Index Configurator. |  
| `make:searchable-model`	| `name` - Le nom de la classe	| Créé un nouveau modèle lié à Elasticsearch. |  
| `make:search-rule`	| `name` - Le nom de la classe	| Créé une nouvelle règle de recherche. |  
| `elastic:create-index`	| `index-configurator` - La classe de l'IndexConfigurator	| Créé un Index Elasticsearch. |  
| `elastic:update-index`	| `index-configurator` - La classe de l'IndexConfigurator	| Met à jour les paramètres de configuration et le mapping d'un index Elasticsearch. |  
| `elastic:drop-index`	| `index-configurator` - La classe de l'IndexConfigurator	| Supprime un Index Elasticsearch.|  
| `elastic:update-mapping`	| `model` - La classe modèle	| Met à jour le mapping d'un modèle. |  
| `elastic:migrate`	| `model` - La classe modèle, `target-index` - Le nom de l'index à migrer	|  Migre un modèle vers un autre index. |  
| `scout:import` | `model` - La classe modèle | Indexe les données de la table SQL dont le nom est identique au nom du modèle utilisé. |
