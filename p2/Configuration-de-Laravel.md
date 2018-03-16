# Prérequis

* PHP version >= 7.1.3
* Laravel Framework version >= 5.6
* Elasticsearch version >= 6

# Installation

Dans le dossier de Laravel, ouvrir la console qui permet de traiter avec `composer`

Dans la console entrer : `composer require babenkoivan/scout-elasticsearch-driver`

L'installation du package va se faire.

# Configuration du package

Afin de configurer le package, il est nécessaire de publier des paramètres avec les commandes suivantes : 
>  `php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"`  
>  `php artisan vendor:publish --provider="ScoutElastic\ScoutElasticServiceProvider"`

Ensuite, dans le fichier `config/scout.php`, modifier le code suivant :  
> `'driver' => env('SCOUT_DRIVER', 'algolia'),` en `'driver' => env('SCOUT_DRIVER', 'elastic'),`

Il est possible de configurer le driver dans le fichier `config/scout_elastic.php` 

Les options disponibles sont :

| Option	| Description |
| :-----------: | :---------: |
|client	| Une des configurations pour paramétrer Elasticsarch. Plus d'informations [ici](https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/_configuration.html#_building_the_client_from_a_configuration_hash). Par défaut l'hôte est déterminé comme suit : localhost:9200. |
|update_mapping	|Une option qui permet de définir si on automatise la mise à jour des mappings. Par défaut c'est configuré sur `true`|
|indexer	|Configuré `single` pour avoir une indexation document par document ou `bulk` pour indexer un wagon de documents avec la méthode *bulk*. Par défaut, c'est configuré sur `single`. **Note** : la commande d'indexation des données présentes en BD `scout:import` fonctionne très bien en `single`.|

# Index configurator

Grâce à la classe Index configurator, il sera possible de gérer le paramétrage des Index.

**Remarque** : Il va être nécessaire de créer autant d'IndexConfigurator que d'Index. Un IndexConfigurator est lié à un **unique** Index et inversement.

La ligne suivante permet de créer un nouvel Index configurator :  
> `php artisan make:index-configurator MyIndexConfigurator`

Cette ligne de commande va créer le fichier `MyIndexConfigurator` dans le dossier `app/` du projet Laravel, qui ressemble à ceci :  
```php
<?php

namespace App;

use ScoutElastic\IndexConfigurator;
use ScoutElastic\Migratable;

class MyIndexConfigurator extends IndexConfigurator
{
    use Migratable;

    // Variable optionnelle qui reconfigure le nom par défaut de l'Index créé à partir de cette configuration.
    // De base, le nom de l'Index sera le même que celui de la classe sans "IndexConfigurator"
    protected $name = 'produits_nacel';

    // Il est possible de paramétrer un analyzer pour les recherches. 
    // Il est obligatoire que l'analyzer soit ici et non ailleurs.
    protected $settings = [
        'index' => [
            'number_of_shards' => 5, //Paramètre par défaut si rien n'est renseigné
            'number_of_replicas' => 1 //Paramètre par défaut si rien n'est renseigné
        ]
        'analysis' => [
            'analyzer' => 'french_light',
            'fields' => [
                'stemmed' => [
                    'type' => 'text',
                    'analyzer' => 'french_heavy',
                ],
            ],
        ],
    ];
}
```
Plus d'informations sur le paramétrage d'un index dans la [documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/guide/current/index-management.html).

# Créer un Index

**Enfin**, la création d'un Index ce fait ainsi :  
> `php artisan elastic:create-index App\MyIndexConfigurator`  

L'index prendra soit le nom de sa classe (en enlevant "IndexConfigurator") soit le nom spécifié dans le corps de la classe de l'IndexConfigurator utilisé.

# Modèles liés à Elasticsearch

Afin de réaliser des requêtes de recherche à travers Elasticsearch, il faut créer un/des modèles permettant de le faire :  
> `php artisan make:searchable-model MyModel --index-configurator=MyIndexConfigurator`

**Remarque** : Un modèle est un équivalent du type. Sauf qu'ici particulièrement, on ne peut l'appeler "_doc" car le nom du modèle est important pour plusieurs traitements décris plus loin. Lors d'un appel URL, on utilisera donc `index/modèle/id` tout comme on le faisait avec  `index/type/id` avec type = "_doc". 

Le fichier `MyModel.php` va se créer dans le dossier `app/` de votre projet Laravel.  

Un modèle ressemble à ça :  
```php
<?php

namespace App;

use ScoutElastic\Searchable;
use Illuminate\Database\Eloquent\Model;

class MyModel extends Model
{
    use Searchable;

    protected $indexConfigurator = MyIndexConfigurator::class;

    protected $searchRules = [
        //
    ];

    // Il est possible ici de définir au préalable un mapping pour les champs du modèle.
    // Ceci n'est qu'un exemple de mapping et il est nécessaire de l'adapter à vos besoins.
    protected $mapping = [
        'properties' => [
            'text' => [
                'type' => 'text',
                'fields' => [
                    'raw' => [
                        'type' => 'text',
                        'index' => 'not_analyzed',
                    ]
                ]
            ],
        ]
    ];
}
```

Le mapping est important, mieux est définie la structure, meilleure sont indexation et recherche. On peut notamment spécifier quel analyzer sera utilisé pour tel ou tels champs lors de la recherche. 

**Remarque** : Il est obligatoire de rentrer quelque chose dans la variable "$mapping" auquel cas, aucune importation (voir en dessous) ne pourra se faire, avec un message d'erreur typique : 
> `Nothing to update: the mapping is not specified.`

Chaque modèle représente un type Elasticsearch (même si les types sont voués à disparaitre, pour le moment cela fonctionne comme cela). Par défaut, un type correspond à une table. 
D'autres options sont proposées dans la [documentation de Scout](https://laravel.com/docs/5.5/scout#configuration).

La dernière option importante est la propriété `$searchRules`. Elle permet la mise en place d'algorithmes de recherche pour un modèle. Les détails seront donnés dans la section [Règles de recherche/SearchRule]().

Après avoir configuré un mapping pour un modèle, on peut le mettre à jour :  
> `php artisan elastic:update-mapping App\MyModel`

# Indexer des données

Une fois les Index et les Modèles créés, intervient l'indexation des données. 

Une méthode simple grâce à la commande : 
> `php artisan scout:import "App\MyModel`  

Permet d'aller chercher dans votre base de donnée la table "MyModel" et, si elle la trouve, d'indexer tout son contenu dans l'Index issu du même IndexConfigurator que le modèle. 

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


Pour une description détaillées, entrer cette ligne de commande avec la commande qui vous intéresse : `php artisan help [command]` **EXISTE MAIS NE FONCTIONNE PAS**



