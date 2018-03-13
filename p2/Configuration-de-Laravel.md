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
`php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
php artisan vendor:publish --provider="ScoutElastic\ScoutElasticServiceProvider"`