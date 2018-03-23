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
    "query": { //On indique que l'on fait une requête
        "match" : { //Cette requête doit matcher
            "libelle" : { //Le(s) champ(s) libelle de tout le cluster (voir l'url)
                "query" : "séjour linguistique", //La comparaison se fait avec cette requête textuelle
                "boost" : 2 //On booste la valeur du score lorsque cette requête matche
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
        "multi_match" : { //Cette propriété permet de faire la requête sur plusieurs champs
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

## Match et variantes :

`Match` attaque la chaine de caractère entière.

**Récupération de toutes les données du cluster :**

![AllGET](../images/AllGET.png)

*  1 - Afin de faire des recherches par requête, il est obligatoire d'être en POST **et** d'utiliser "_search" de l'API Search
*  2 - On indique que l'on souhaite faire une requête.
*  3 - "match_all" permet de récupérer tout les documents et leurs champs/valeurs du cluster

match_all
match
multi_match -> types de multi_match (most/best/cross/phrase/phrase_prefix)
match_phrase
match_phrase_prefix

**Documentation supplémentaire** : Tous les détails et subtilités de la propriété match [ici dans la documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html).
## Query string :

`Query_string` attaque la chaine de caractère en la segmentant en tokens. (1 token = 1 "mot").

fuziness
wildcard
recherche de proximité
range

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
POST sl_produit_e_ss/_doc/_search
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

Les 

# Analyzer

# Aggregations
