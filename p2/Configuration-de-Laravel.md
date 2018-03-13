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

`
test
`
