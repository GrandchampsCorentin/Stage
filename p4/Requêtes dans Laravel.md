# Utilitaire

Une fois créés :
* un Index configurator
* un Index Elasticsearch
* un modèle particulier  

Tout est prêt.  
Maintenant on peut faire des recherches sur les index.

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
**Remarque** : Je n'ai pas bien saisi l'utilité d'une telle méthode.  
```php
App\MyModel::search('phone') 
    ->with('makers')
    ->get();
```
En plus des fonctionnalités standart, le package offre la possibilité de filtrer les données dans Elasticsearch sans spécifier de requête.  
`search('*')` fera son travail de comparaison sur l'ensemble de l'index cible :  
```php
App\MyModel::search('*')
    ->where('id', 1)
    ->get();
```

Utiliser une variété de conditions `where` est possible (voir la liste dans le tableau en fin de document) :  
```php
App\MyModel::search('*')
    ->whereRegexp('name.raw', 'A.+')
    ->where('age', '>=', 30)
    ->whereExists('unemployed')
    ->get();
```
Enfin, il est toujours possible d'envoyer une requête personnalisée grâce au `searchRaw()` qui est une méthode très intéressante lorsqu'il est nécessaire de sortir des sentiers battus par le driver.  
Un exemple :  
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
La construction d'une requête searchRaw est presque identique à celle d'un document JSON à quelques variantes près :
* Les `{ }` deviennent des `[ ]`
* Les `:` deviennent des `=>`

 Le résultat d'une requête `searchRaw()` est déjà au format JSON et ne nécessite pas l'application d'un `explain()` ou d'un `profile()` pour proposer des informations supplémentaires.   
 
**Remarque** : Pour plus d'informations sur le `explain()` ou le `profile()`, allez voir la section **Debug** en bas de ce document.

**Documentation supplémentaire** : Pour plus de renseignements sur les méthodes de Laravel Scout c'est [ici qu'il faut cliquer](https://laravel.com/docs/5.6/scout#searching).

# Règles de recherche / SearchRule

## Créer une SearchRule

Une règle de recherche est une classe qui décrit comment une requête de recherche va être exécutée.  
La commande ci-dessous permet de créer une telle classe : 

> `php artisan make:search-rule MySearchRule`

Avec cette ligne de commande le fichier `MySearchRule` sera créé dans le dossier `App\`.

**Note** : Il est possible de déterminer un emplacement spécifique pour ces classe là tel que :   
> `php artisan make:search-rule App\Models\ES\SearchRules\LangueSearchRule`

Dans le fichier `MySearchRule.php` on trouvera cette définition de la classe :  
```php
<?php

namespace App\Models\ES\SearchRules;

use ScoutElastic\SearchRule;

class LangueSearchRule extends SearchRule
{
    //Cette méthode retourne un array, qui sera traité comme du JSON
    //Il est possible de créer n'importe quel traitement de requête à renvoyer
    //Par exemple, ci-dessous, un bout de requête booléenne (must[]) sera renvoyé
    public function buildQueryPayload() //Ne pas toucher au nom de la fonction
    {
        return [
            'must' => [
                'match' => [
                    //$this->builder->query va associer votre requête textuelle au champ libelle 
                    //Afin de faire des comparaisons de tokens et déterminer si ils matchent ensemble
                    'libelle' => $this->builder->query 
                ]
            ]
        ];
    }
}
```

## Utilisation d'une SearchRule

Pour utiliser une règle de recherche, il "suffit" de l'appeler dans votre traitement Elasticscout.

Exemple :  
```php
//Entête
use App\Models\ES\SearchRules\LangueSearchRule;
use App\Models\ProduitSlES

/*----
/...
----*/

//Requête
$requêteLangue = ProduitSlES::search('*')
                        ->whereIn('id', $tableauIDlangues)
                        ->rule(LangueSearchRule::class) //On utilise bien ici la règle de recherche stipulée au dessus
                        ->get()
                        ->JSON();
```

La règle de recherche renvoie tout ce qu'elle contient dans son `return`.  

## Les SearchRules dans les Modèles

Il est possible d'assigner des règles de recherche en prévision dans les modèles :  
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
**Note** : Les règles de recherche dans les modèles seront appliquées pour chaque recherche effectuée à travers ce modèle. Autrement dis, il faut les voir comme des routines. Vous utilisez le modèle ? Alors le traitement va se faire et par votre traitement ET par les règles de recherche assignées au modèle. 

## Improvisation dans un traitement du QueryBuilder

Il est aussi possible de rajouter des règles de recherche dans le constructeur de requête (query builder) : 

```php
//Il est possible de rajouter une classe SearchRule directement
//Ou d'override une règle existante dans le modèle
App\MyModel::search('Brazil')
    ->rule(App\MySearchRule::class)
    ->get();
    
//Ou en créant une règle particulière directement dans l'enchainement du traitement
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
|whereGeoPolygon($field, array $points)	| whereGeoPolygon('location',[[-70, 40], [-80, 30], [-90, 20]])	| Filtre les enregistrements dans un polygone. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-polygon-query.html) des renseignements supplémentaies syntaxiques.|


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

Mise à part le `->get()` qui ne retourne que les documents concernés par la requête, il existe deux méthodes qui peuvent aider lors de l'analyse de résultats suite à des recherches avce un `search()` :

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