# Etape 1 : La configuration d'un Index

Grâce à la classe `Index configurator`, il sera possible de gérer la configuration de chaque Index.

**Remarque** : Il va être nécessaire de créer autant d'IndexConfigurator que d'Index. Un IndexConfigurator est lié à un **unique** Index et inversement.

La ligne suivante permet de créer un nouveau fichier IndexConfigurator :  
> `php artisan make:index-configurator MyIndexConfigurator`


Cette ligne de commande va créer le fichier `MyIndexConfigurator` dans le dossier `app/` du projet Laravel.  

**Note** : il est tout à fait possible de spécifier un répertoire différent :
> `php artisan make:index-configurator App\Models\ES\Configurator\LangueIndexConfigurator`  
Il ne faut cependant pas oublier de spécifier le **namespace** dans le fichier !

Le fichier IndexConfigurator ressemble à ceci :  
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
        'analysis' => [ //Configuration de l'analyzer qui sera utilisé lors de l'indexation des données
            'filter' => [ //Mise en place de filtres particuliers
                'french_elision' => [ //Ce filtre supprime les mots composés d'apostrophe
                    'type' => 'elision',
                    'articles_case' => true,
                    'articles' => [
                        'l', 'm', 't', 'qu', 'n', 's',
                        'j', 'd', 'c', 'jusqu', 'quoiqu',
                        'lorsqu', 'puisqu',
                    ],
                ],
                'french_stop' => [ //Ce filtre supprime les mots communs de la langue française (sujets, déterminants, etc...)
                    'type' => 'stop',
                    'stopwords' => '_french_',
                ],
                'french_stemmer' => [ //Ce filtre découpe les termes afin de ne retenir que la racine du mot
                //Idéal pour chercher un mot sans se soucier du genre, du nombre ou de la conjugaison
                    'type' => 'stemmer',
                    'language' => 'light_french',
                ],
            ],
            'analyzer' => [ //Construction de l'analyzer avec les filtres
                'french' => [ //Nom de l'analyzer
                    'tokenizer' => 'standard', //Tokenizer permet de découper les phrases en token (Un token comportant un terme)
                    'filter' => [
                        'french_elision',
                        'lowercase',
                        'french_stop',
                        'french_stemmer',
                    ],
                ],
            ],
        ],
    ];
}
```
Plus d'informations sur le paramétrage d'un index dans la [documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/guide/current/index-management.html).

# Etape 2 : La création d'un Index

La création d'un Index permet d'instancier un nouvel Index dans votre Elasticsearch, avec les propriétés définies précédemment dans l'IndexConfigator. 

La création d'un Index ce fait basiquement ainsi :  
> `php artisan elastic:create-index App\MyIndexConfigurator`  

**Note** : si vous avez rangé votre IndexConfigurator dans un autre répertoire, il suffit de le lui indiquer comme suit :  
> `php artisan elastic:create-index App\Models\ES\Configurator\LangueIndexConfigurator`

L'index prendra soit le nom de sa classe (en enlevant "IndexConfigurator") soit le nom spécifié dans le corps de la classe de l'IndexConfigurator utilisé.

Ici, j'ai choisi de lui attribuer le nom qui précède IndexConfigurator.

# Etape 3 : Lier un Modèle de Laravel à un Index d'Elasticsearch

## Créer un nouveau modèle

Afin de réaliser des requêtes de recherche à travers Elasticsearch, il faut créer un/des modèles permettant de le faire :  
> `php artisan make:searchable-model MyModel --index-configurator=MyIndexConfigurator`

**Remarque** : Un modèle est un équivalent du type. Sauf qu'ici particulièrement, on ne peut l'appeler "_doc" car le plugin ne le permet pas. Lors d'un appel URL, on utilisera donc `index/modèle/id` tout comme on le faisait avec  `index/type/id` avec type = "_doc". 

Le fichier `MyModel.php` va se créer dans le dossier `app/` de votre projet Laravel.  

**Note** : Il est encore une fois possible de spécifier un autre répertoire dans lequel le modèle se créera. Il ne faudra pas oublier de modifier le namespace. Un exemple ci-dessous :  
> `php artisan make:searchable-model App\Models\Langue --index-configurator=App\Models\Es\Configurator\LangueIndexConfigurator`

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
        'properties' => [ // Définition des champs de l'index
            'text' => [ // Le champ 'text' est défini comme :
                'type' => 'text', // de type 'text'
                'analyzer' => 'french', // et analisé par l'analyzer 'french'
                ]
            ],
        ]
    ];
}
```

## Utiliser un modèle existant

Votre modèle est déjà créé : 

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Model;

class Langue extends Model
{
    use SoftDeletes;

    public function slProduit()
    {
        return $this->hasMany('App\Models\Web\SlProduit');
    }
}
```

Il faudra alors rajouter plusieurs éléments :
* L'appel des classes Searchable et IndexConfigurator  
> `use ScoutElastic\Searchable;`    
> `use App\Models\ES\Configurator\LangueIndexConfigurator;`  
* L'utilisation de la classe Searchable  
> `use Searchable;`  
* La connexion à la base de données ciblée  
> ` protected $connection  = 'web'; `    
> `protected $table 		    = 'langues';`    
> ` protected $guarded 		  = [];`    
* L'utilisation de l'IndexConfigurator  
> `protected $indexConfigurator = LangueIndexConfigurator::class;`  
* Les règles de recherches   
> `    protected $searchRules = [
        //
    ];`
* Le mapping  
> `    protected $mapping = [
        'properties' => [
            'id' => [
                'type' => 'integer',
                'analyzer' => 'french'
            ],
        ],
    ];
`

Le modèle devrait ressembler à ça une fois toutes les modifications faites :  
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Model;
//L'appel des classes Searchable et IndexConfigurator
    use ScoutElastic\Searchable;
    use App\Models\ES\Configurator\LangueIndexConfigurator;
//---
class Langue extends Model
{
    use SoftDeletes;

    //L'utilisation de la classe Searchable
        use Searchable;
    //---

    //La connexion à la base de données ciblée
        protected $connection   = 'web';
        protected $table        = 'langues';
        protected $guarded 	    = [];
    //---

    //L'utilisation de l'IndexConfigurator
        protected $indexConfigurator = LangueIndexConfigurator::class;
    //---

    //Les règles de recherches
        protected $searchRules = [
            //
        ];
    //--- 

    //Le mapping
        protected $mapping = [
            'properties' => [
                'id' => [
                    'type' => 'integer',
                ],
            ],
        ];
    //---

    public function slProduit()
    {
        return $this->hasMany('App\Models\Web\SlProduit');
    }
}
```

## L'importance du mapping

Le mapping est important car mieux est définie la structure, meilleure sont l'indexation et la recherche. On peut notamment spécifier quel analyzer sera utilisé pour tel ou tels champs lors de la recherche. 

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
> `php artisan scout:import App\Models\Langue`  

Permet d'aller chercher dans votre base de donnée la table spécifiée dans le modèle et, si elle la trouve, d'indexer tout son contenu dans l'Index issu du même IndexConfigurator que le modèle. 

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