## [Recherche](#user-content-recherche)
*  [Recherche Globale et par ID](#user-content-globale-et-par-id)
*  [Recherche par Requête](#user-content-par-requête)

# Le scoring

Avant de partir à la croisade de votre multitude de documents indexés, il faut savoir comment Elasticsearch établi son système de pertinence. 

Lorsqu'une requête est envoyée, il y a un traitement de la requête qui est fait. Qu'il soit simple, ou complexe, Elasticsearch proposera une liste de résultats triés par `score`.

Il faut savoir que par défaut, le score de tout document est de `1.0`. 

Le score est modifié par une multitudes de propriétés, et chaque traitement supplémentaire permettra à Elasticsearch de répondre plus pertinemment à la recherche qui sera faite. 

La propriété la plus simple pour modifier le score est le `Boost`.

# Boost

Le Boost permet d'appliquer une modification du score rapide et visuelle.   
Par défaut le Boost est de 1.
```json
POST _search
{
    "query": { //On_indique_que_l'on_fait_une_requête
        "match" : { //Cette_requête_doit_matcher
            "libelle" : { //Le(s)_champ(s)_libelle_de_tout_le_cluster_(voir_l'url)
                "query" : "séjour linguistique", //La_comparaison_se_fait_avec_cette_requête_textuelle
                "boost" : 2 //On_booste_la_valeur_du_score_lorsque_cette_requête_matche
            }
        }
    }
}
```
Résultat : Lors de certaines recherches comme une recherche globale, les documents aux libellés matchant la requête auront plus de poids que les autres. 

Il est possible d'affecter un Boost à un champ :
```json
POST _search
{
    "query": {
        "multi_match" : { //Cette_propriété_permet_de_faire_la_requête_sur_plusieurs_champs
                "fields" : {"libelle^2", "contenu", "introduction^5", "accroche^10", "intitule"},
                "query" : "séjour linguistique",
            }
        }
    }
}
```
Dans l'exemple ci-dessus, nous avons une requête qui va essayer de matcher avec au moins un des champs listés dans "fields".  
Dans la propriété "fields", on nomme les champs concernés, et, si on souhaite leur appliquer un Boost, on leur ajoute une puissance x : `^2` par exemple. 

**Note** : Si pour le moment on ne parle que des Boosts "positifs", il existe la propriété `negative_boost` qui permet de rentrer des Boosts inférieurs à 1, et qui diminuent donc la valeur du score qu'ils impactent.

# L'Analyzer

L'Analyzer est un outil très puissant permettant de passer outre les règles verbeuses telles que : le genre, le nombre, la conjugaisons et les caractères spéciaux de la langue. 

## L'Analyzer dans l'Index

Paramétrer un analyzer dans l'index permettra de créer un index inversé, d'analyser tous les documents de l'index et de transformer les textes en tokens, utilisables pour la recherche.

Le mapping d'un index avec analyzer ressemble à cela : 
```php
<?php

namespace App\Models\ES\Configurator;

use ScoutElastic\IndexConfigurator;
use ScoutElastic\Migratable;

class SlProduitESConfigurator extends IndexConfigurator
{
    use Migratable;

    protected $name = 'sl_produit_es';

    protected $settings = [
        'analysis' => [ //On instancie les divers filtres pour l'analyzer
            'filter' => [
                'french_elision' => [ //Ce filtre supprime les articles avec apostrophe
                    'type' => 'elision',
                    'articles_case' => true,
                    'articles' => [
                        'l', 'm', 't', 'qu', 'n', 's',
                        'j', 'd', 'c', 'jusqu', 'quoiqu',
                        'lorsqu', 'puisqu',
                    ],
                ],
                'french_stop' => [ //Ce filtre nerécupère pas sous forme de token les mots communs de la langue française (Déterminants, etc...)
                    'type' => 'stop',
                    'stopwords' => '_french_',
                ],
                'french_stemmer' => [ //Ce filtre supprime genre, nombre et conjugaison en ne gardant que la racine des mots
                    'type' => 'stemmer',
                    'language' => 'light_french',
                ],
            ],
            'analyzer' => [ //On construit l'analyzer
                'french' => [ //Le nom de l'analyzer
                    'tokenizer' => 'standard', //Le type du tokenizer à adopter
                    'filter' => [ //Les filtres précédemments instanciés qui seront utilisés
                        'french_elision',
                        'lowercase', //Réduit tous les termes à être en minuscule
                        'french_stop',
                        'french_stemmer',
                    ],
                ],
            ],
        ],
    ];
}
```

## L'Analyzer dans le Modèle

Une fois l'Analyzer construit, il faut l'appliquer aux divers champs de l'index qui sont textuels. De ce fait, un index inversé sera rempli des tokens issus de l'analyse des données indexées.

Exemple : 
```php
<?php
namespace App\Models;

use ScoutElastic\Searchable;
use Illuminate\Database\Eloquent\Model;
use App\Models\ES\Configurator\SlProduitESConfigurator;

class SlProduitES extends Model
{
    use Searchable;

    protected $indexConfigurator = SlProduitESConfigurator::class;

    protected $searchRules = [
        //
    ];

    protected $mapping = [
        'properties' => [
            'langue' => [
                'type' => 'text',
                'analyzer' => 'french', //On détermine quel analyzer est utilisé pour indexer les valeurs du champ langue
            ],
            'formule' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
            'categorie' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
            'hebergements' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
            'villes' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
            'pays' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
            'modes_transports' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
            'accroche' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
            'introduction' => [
                'type' => 'text',
                'analyzer' => 'french',
            ],
        ],   
    ];
}
```

## L'Analyzer dans la Requête 

Enfin, il est possible d'appliquer un analyzer sur les requêtes textuelles avant qu'elles ne soient traitées par Elasticsearch. 

Il suffit souvent d'ajouter la phrase suivante : 
> `"analyzer" : "nomDeLanalyzer"`

Par exemple : 
```json
POST sl_produit_es/sl_produit_e_ss/_search
{
    "query" : {
        "multi_match" : {
           "query" : "sejour parent enfant",
           "fields" : ["cms.accroche","cms.introduction"],
           "analyzer" : "french"
        }
    }
}
```

Le plus performant sera une recherche analysée, faite sur un index ayant lui aussi été analysé, le tout par le même analyzer. 

# Recherche par ID

Afin de récupérer les informations indexées dans un document spécifique, on utilise le code suivant : `GET /index/type/id`

Ainsi les champs et leurs valeurs seront retournés. 

# Recherche par Requête - Termes

Pour faire des recherches par requête, quelques subtilités s'imposent. 

En RESTful, le GET ne peut pas avoir de body, le body étant le JSON que l'on envoit en dessous des liens. Pour contourner la chose, ElasticSearch à développé une API, Search, qui permet de faire de la lecture de données à travers le terme POST, qui accepte les bodys. 

**Documentation supplémentaire** : Tous les détails et subtilités de la propriété term(s) et consors [ici dans la documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html).

## Récupération par équivalence

On recherche tous les documents dont un ou plusieurs champs possèdent exactement telles valeurs.

![TermEqui](../images/TermEqui.png)

*  1 - On spécifie le terme `_search` obligatoire pour de la recherche en POST
*  2 - On fait une recherche donc on ajoute `query`
*  3 - `term` permet de faire une recherche par équivalence. La/les valeurs(s) et le/les champs entrés doivent être identiques pour renvoyer un résultat. 

# Recherche par Requête - Chaines de caractères

## Match :

`Match` est la méthode standart permettant de faire de la recherche textuelle. 

Exemple simple :  
```json
POST /_search
{
    "query" : {
        "match" : {
            "sejour" : "formation professionnelle linguistique"
        }
    }
}
```

Plusieurs outils sont inclus dans le match dont le principal est `fuzzi`. Une des manières de tolérer les fautes de frappes. 


Ses paramètres :
* `fuzziness` : Détermine combien de caractères peuvent ne pas être dans le mot pour le valider comme résultat possible. Par défaut la valeur est `"AUTO"`.
* `prefix_lenght` : Le nombres de caractères initiaux non concernés par le `fuzzy`. Cela permet de réduire le nombre de termes à examiner. Par défaut la valeur est `0`.
* `max_expansions` : Le nombre maximal de combinaisons que tentera la méthode `fuzzy` afin d'obtenir un résultat. Par défaut la valeur est de `50`.
* `fuzzy_transpositions` : Autorise les transpositions de caractères (ab -> ba). Par défaut la valeur est true.

Exemple :  
```json
POST sl_produit_es/sl_produit_e_ss/_search
{
    "query" : {
        "match" : {
            "cms.formule" : {
                "query" : "sejour parent enfant",
                "fuzziness" : "AUTO",
                "max_expansions" : 10,
                "prefix_length": 1,
                "fuzzy_transpositions" : false,
            }
        }
    }
}
```  
Résultat : 
```json
    "formule": "Séjour parents-enfants",
```

Les autres outils utilent avec le `match` sont [par là](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html).

**Récupération de toutes les données du cluster :**

![AllGET](../images/AllGET.png)

*  1 - Afin de faire des recherches par requête, il est obligatoire d'être en POST **et** d'utiliser "_search" de l'API Search
*  2 - On indique que l'on souhaite faire une requête.
*  3 - "match_all" permet de récupérer tout les documents et leurs champs/valeurs du cluster


**Le multi-match**

Le `multi_match` permet de faire une recherche textuelle sur plusieurs champs. 

Exemple : 
```JSON
POST sl_produit_es/sl_produit_e_ss/_search
{
    "query" : {
        "multi_match" : {
           "query" : "sejour parent enfant",
           "fields" : ["cms.accroche","cms.introduction"]
        }
    }
}


```
Lors de l'utilisation du `multi_match` il est possible d'ajouter une option de recherche appellée `type` que l'on retrouve  [ici dans la documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html).  
Ses paramètres :
* `best_fields` : Par défaut. Cherche les documents qui matchent avec n'importe quel champ, mais utilises le score de clui qui retourne le meilleur score.
* `most_fields` : Cherche les documents qui matchent avec n'importe quel champ et combine les scores de chaque champ matchant.
* `cross_fields` : Traite les champs avec le même `analyzer` comme si ils n'étaient qu'un seul et unique champ. Cherche chaque mot dans n'importe quel champ.
* `phrase` : Lance une requête `match_phrase` sur chaque champ et combine le score de chaque champ. [Renseignements](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-phrase).
* `phrase_prefix` : Lance une requête `match_phrase_prefix` sur chaque champs et combine les scores de chaque champ. [Renseignements](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#type-phrase).

**Documentation supplémentaire** : Tous les détails et subtilités de la propriété match [ici dans la documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html). 

## Query string :

`Query_string` est la méthode avancée de recherche textuelle, agrémentée de la syntaxe Lucene.

fuziness : Accepte les fautes de frappe
wildcard : Accepte l'oubli d'une ou de plusieurs lettres
recherche de proximité : Accepte que l'ordre des mots de la requête textuelle soit différent de celui proposé
range : Accepte des intervalles de dates / nombres / chaines de caractères
caractères réservés : `+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /`

**Documentation supplémentaire** : Tous les détails et subtilités de la propriété query_string [ici dans la documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html).


## Requête booléenne :

Une requête booléenne se présente comme suit : 
```json
POST _search
{
    "query" : {
        "bool" : {
            "must" : {
                //Traitement
            },
            "filter" : {
                //Traitement
            },
            "should" : {
                //Traitement
            },
            "must_not" : {
                //Traitement
            },
        }
    }
}
```

* **MUST** : Elasticsearch DOIT traiter la recherche et proposer les documents ayant matché avec l'ajout d'un score par document.
* **FILTER** : Elasticsearch DOIT traiter la recherche et proposer les documents ayant matché, sans interférer avec le score. 
* **SHOULD** :  Elasticsearch PEUT traiter la recherche.
    * Si le SHOULD est seul, il fonctionnera comme un MUST en moins puissant en terme de scoring. 
    * Si le SHOULD est couplé à un MUST :
        * 1er cas : Le SHOULD matche seul. Il propose les documents ayant matché avec un score par document. Degrès de pertinence `+`.
        * 2ème cas : Le MUST matche seul. Il propose les documents ayant matché avec un score par document.  Degrès de pertinence `++`.
        * 3ème cas : Les deux matchent. Le MUST propose les documents ayant matché avec un score par document. Le SHOULD appuie le score des documents ayant matchés avec lui aussi.  Degrès de pertinence `+++`.
* **MUST_NOT** : Elasticsearch NE DOIT PAS proposer les documents ayant matché avec le traitement du MUST_NOT. Ceci n'interfère pas avec le scoring des documents résultant d'une autre propriété.
   
Exemple concret : 

```json
POST sl_produit_es/sl_produit_e_ss/_search
{
    "query" : {
        "bool" : {
            "must" : {
                    "match" : {"categorie" : "sejour formation"},
                    "term" : {"langue" : "anglais"},
                },
            "should" : { 
                    "query_string" : { 
                    "query" : "grand sejour en angleterre avec apprentissage de la langue", 
                    "fields" : {"cms.introduction","cms.accroche^2"}, 
                    "type" : "cross_fields",
                },
            }
        }
    }
}
```

# Aggregations
