# Prérequis

PHP version >= 7.1.3
Laravel Framework version >= 5.6
Elasticsearch version >= 6

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
|indexer	|Configuré `single` pour avoir une indexation document par document ou `bulk` pour indexer un wagon de documents avec la méthode *bulk*. Par défaut, c'est configuré sur `single`.|

# Index configurator

Grâce à la classe Index configurator, il sera possible de gérer le paramétrage des Index.

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

    // Variable optionnelle qui reconfigure le nom par défaut de l'Index configurator.
    protected $name = 'produits_nacel';

    // Il est possible de paramétrer un analyzer pour les recherches. 
    // Il est obligatoire qu'il soit ici et non ailleurs.
    protected $settings = [
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

**Enfin**, la création d'un Index qui suit le paramétrage que l'on vient de configurer ce fait ainsi :  
> `php artisan elastic:create-index App\MyIndexConfigurator`  

# Modèles liés à Elasticsearch

Afin de réaliser des requêtes de recherche à travers Elasticsearch, il faut créer un/des modèles permettant de le faire :  
> `php artisan make:searchable-model MyModel --index-configurator=MyIndexConfigurator`

Le fichier `MyModel.php` va se créer dans le dossier `app/` de votre projet Laravel et ressemble à ça :  
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

Chaque modèle représente un type Elasticsearch (même si les types sont voués à disparaitre, pour le moment cela fonctionne comme cela). Par défaut, un type correspond à une table. 
D'autres options sont proposées dans la [documentation de Scout](https://github.com/babenkoivan/scout-elasticsearch-driver/blob/master/README.md#search-rules).

La dernière option importante est la propriété `$searchRules`. Elle permet la mise en place d'algorithmes de recherche pour un modèle. Les détails seront donnés dans la section [Règles de recherche]().

Après avoir configuré un mapping pour un modèle, on peut le mettre à jour :  
> `php artisan elastic:update-mapping App\MyModel`

# Liste des commandes sur console 

Toutes les commandes "artisan" sont listées ci-dessous :  
| Command	| Arguments	| Description |
| :-----------: | :-----------: | :---------: |
|make:index-configurator	|name - The name of the class	|Creates a new Elasticsearch index configurator.|  
|make:searchable-model	|name - The name of the class	|Creates a new searchable model.|  
|make:search-rule	|name - The name of the class	|Creates a new search rule.|  
|elastic:create-index	|index-configurator - The index configurator class	|Creates an Elasticsearch index.|  
|elastic:update-index	|index-configurator - The index configurator class	|Updates settings and mappings of an Elasticsearch index.|  
|elastic:drop-index	|index-configurator - The index configurator class	|Drops an Elasticsearch index.|  
|elastic:update-mapping	|model - The model class	|Updates a model mapping.|  
|elastic:migrate	|model - The model class, target-index - The index name to migrate	|Migrates model to another index.|  


For detailed description and all available options run php artisan help [command] in the command line.
