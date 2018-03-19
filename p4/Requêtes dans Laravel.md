# Utilitaire

Une fois créés :
* un Index configurator
* un Index Elasticsearch
* un modèle particulier  

Tout est prêt.  
Maintenant on peut indexer et faire des recherches de données.

#### Requêtes de recherche basiques :  
```php
// Créer la requête
App\MyModel::search('phone')
    // Filtrer par champ
    ->where('color', 'red')
    // Organiser les données
    ->orderBy('price', 'asc')
    // Définit à partir de quel résultat on commence à afficher
    ->from(0)
    // Définit combien de résultats sont affichés (ici 10)
    ->take(10)
    // Obtenir les résultats
    ->get();
    // Passage au JSON
    ->JSON();
```  
Si il est nécessaire de préciser l'existence de relations, c'est possible avec `with` :  

```php
App\MyModel::search('phone') 
    ->with('makers')
    ->get();
```
En plus des fonctionnalités standart, le package offre la possibilité de filtrer les données dans Elasticsearch sans spécifier de requête :  
```php
App\MyModel::search('*')
    ->where('id', 1)
    ->get();
```

Il est aussi possible d'override les règles de recherche du modèle :  
```php
App\MyModel::search('Brazil')
    ->rule(App\MySearchRule::class)
    ->get();
```
Et d'utiliser une variété de conditions `where` :  
```php
App\MyModel::search('*')
    ->whereRegexp('name.raw', 'A.+')
    ->where('age', '>=', 30)
    ->whereExists('unemployed')
    ->get();
```
Enfin, il est toujours possible d'envoyer une requête personnalisée :  
```php
App\MyModel::searchRaw([
    'query' => [
        'bool' => [
            'must' => [
                'match' => [
                    '_all' => 'Brazil'
                ]
            ]
        ]
    ]
]);
```
Cette requête va renvoyer une réponse brute (raw response).

# Règles de recherche / SearchRule

Une règle de recherche est une classe qui décrit comment une requête de recherche va être exécutée.  
La commande ci-dessous permet de créer une telle classe : 

> `php artisan make:search-rule MySearchRule`

Dans le fichier `app/MySearchRule.php` on trouvera cette définition de la classe :  
```php
<?php

namespace App;

use ScoutElastic\SearchRule;

class MySearch extends SearchRule
{
    // Cette méthode retourne un array qui représente un contenu de requête booléenne.
    public function buildQueryPayload()
    {
        return [
            'must' => [
                'match' => [
                    'name' => $this->builder->query
                ]
            ]
        ];
    }
}
```

Vous pouvez vous renseigner sur les requête booléennes [ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html).

La règle de recherche par défaut renvoie le résultat suivant :  
```php
return [
   'must' => [
       'query_string' => [
           'query' => $this->builder->query
       ]
   ]
];
```

Ce qui signifie que par défaut, lorsque on appelle la méthode `search` sur un modèle, il essaie de trouver un résultat sur tous les champs du modèle.

Pour déterminer les règles par défaut de recherche propres à un modèle il faut ajouter une propriété :  
```php
<?php

namespace App;

use ScoutElastic\Searchable;
use Illuminate\Database\Eloquent\Model;

class MyModel extends Model
{
    use Searchable;
    
    // Il est possible d'ajouter plusieurs règles de recherche pour un modèle. Dans ce cas, le premier résultat qui ne sera pas vide sera retourné.
    protected $searchRules = [
        MySearchRule::class
    ];
}
```

Il est aussi possible de rajouter des règles de recherche dans le constructeur de requête (query builder) : 

```php
// Il est possible de rajouter une classe SearchRule directement
App\MyModel::search('Brazil')
    ->rule(App\MySearchRule::class)
    ->get();
    
// ou en appelant une règle particulière
App\MyModel::search('Brazil')
    ->rule(function($builder) {
        return [
            'must' => [
                'match' => [
                    'Country' => $builder->query
                ]
            ]
        ];
    })
    ->get();
```

# Les filtres disponibles

Il existe un petit nombre de filtres variés :

|Method	| Example |	Description |
| :-: | :-: | :-: |
|where($field, $value)	|where('id', 1)|	Vérifie l'égalité ed'une simple valeur.|
|where($field, $operator, $value)|	where('id', '>=', 1)|	Filtre les enregistrements avec la contrainte d'une règle donnée. Les opérateurs utilisables sont : =, <, >, <=, >=, <>.|
|whereIn($field, $value)	|where('id', [1, 2, 3])	|Vérifie si une valeur appartient à un champ spécifique.|
|whereNotIn($field, $value)	|whereNotIn('id', [1, 2, 3])|	Vérifie si une valeur n'appartient pas à un champ spécifique.|
|whereBetween($field, $value)	|whereBetween('price', [100, 200])	|Vérifie si une valeur est dans un intervalle donné.|
|whereNotBetween($field, $value)|	whereNotBetween('price', [100, 200])|	Vérifie si une valeur n'est pas dans un intervalle donné.|
|whereExists($field)	|whereExists('unemployed')	|Vérifie si une valeur est définie.|
|whereNotExists($field)	|whereNotExists('unemployed')	|Vérifie si une valeur n'est pas définie.|
|whereRegexp($field, $value, $flags = 'ALL')	|whereRegexp('name.raw', 'A.+')	|Filtre les enregistrements en fonction d'une expression régulière choisie. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/query-dsl-regexp-query.html#regexp-syntax) vous trouverez des renseignements sur la syntaxe.|
|whereGeoDistance($field, $value, $distance)|	whereGeoDistance('location', [-70, 40], '1000m')	|Filtre les enregistrements en fonction d'un point donné et d'un écart donné entre le point et la localisation. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-distance-query.html) des renseignements supplémentaies syntaxiques.|
|whereGeoBoundingBox($field, array $value)|	whereGeoBoundingBox('location', ['top_left' => [-74.1, 40.73], 'bottom_right' => [-71.12, 40.01]])|	Filtre les enregistrements avec des spécifications données. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-bounding-box-query.html) des renseignements supplémentaies syntaxiques.|
|whereGeoPolygon($field, array $points)	|whereGeoPolygon('location', [[-70, 40],[-80, 30],[-90, 20]])	|Filtre les enregistrements dans un polygone. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-polygon-query.html) des renseignements supplémentaies syntaxiques.|


# Migrations sans ralentissement

On ne peut pas changer le type des champs déjà créés dans Elasticsearch. La seule façon de faire est de créer un nouvel index avec le mapping correct et d'importer le modèle dans ce nouvel index.  
Une migration peu prendre quelques temps à ce faire, donc pour éviter les ralentissements durant le procédé de migration, le driver lit le vieil index et le copie dans le nouveau. Une fois la migration terminée, il relit le nouvel index et supprime ce qui appartient à l'ancien index et qui n'est pas commun. C'est la manière de procéder de la commande :  
> `php artisan elastic:migrate`.

Avant de lancer la commande, il est impératif que l'Index configurator utilises l'élément `ScoutElastic\Migratable`.   
Si il n'y est pas, ajoutez le, et lancez la commande  
> `php artisan elastic:update-index App\MyIndexConfigurator`.

Quand tout est prêt, et une fois les changements réalisés dans le mapping du modèle, lancez la commande `elastic:migrate` en utilisant la classe du modèle en premier argument et l'index ciblé en second argument :  
> `php artisan elastic:migrate App\MyModel my_index_v2`

**Remarque** : Si il vous faut juste ajouter des champs dans le mapping, utilisez la commande `elastic:update-mapping`.

# Debug

Mise à part le `->get()` qui ne retourne que les documents concernés par la requête, il existe deux méthodes qui peuvent aider lors de l'analyse de résultats suite à des recherches :

`explain`  
```php
App\MyModel::search('Brazil')
    ->explain();
```

Cette méthode permet d'avoir des informations supplémentaires telles que le **score** des documents.

`profile`  
```php
App\MyModel::search('Brazil')
    ->profile();
```  

Cette méthode permet d'avoir, en plus des informations données par le `->explain()`, des informations inhérentes aux *shards* touchés par la recherche.

Les deux méthodes renvoient des données brutes venant d'Elasticsearch (format JSON).