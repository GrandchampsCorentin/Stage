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

The available options are:

Option	Description
client	A setting hash to build Elasticsearch client. More information you can find here. By default the host is set to localhost:9200.
update_mapping	The option that specifies whether to update a mapping automatically or not. By default it is set to true.
indexer	Set to single for the single document indexing and to bulk for the bulk document indexing. By default is set to single.
Note, that if you use the bulk document indexing you'll probably want to change the chunk size, you can do that in the config/scout.php file.
