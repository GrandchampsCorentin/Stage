# La recherche

* **Recherche "propre"**  

> `$résultat = MonModele::search('*')->get();`

* **Pour les recherches complexes**  

> `$résultat = MonModele::searchRaw([  
>    traitement sous forme de tableaux PHP traduit en JSON  
>    ]);`

# Les filtres "where" disponibles

Il existe un petit nombre de filtres `where` variés :

|Method	| Example |	Description |
| :-: | :-: | :-: |
|`where($field, $value)`	|where('id', 1)|	Vérifie l'égalité ed'une simple valeur.|
|`where($field, $operator, $value)`|	where('id', '>=', 1)|	Filtre les enregistrements avec la contrainte d'une règle donnée. Les opérateurs utilisables sont : =, <, >, <=, >=, <>.|
|`whereIn($field, $value)`	|where('id', [1, 2, 3])	|Vérifie si une valeur appartient à un champ spécifique.|
|`whereNotIn($field, $value)`	|whereNotIn('id', [1, 2, 3])|	Vérifie si une valeur n'appartient pas à un champ spécifique.|
|`whereBetween($field, $value)`	|whereBetween('price', [100, 200])	|Vérifie si une valeur est dans un intervalle donné.|
|`whereNotBetween($field, $value)`|	whereNotBetween('price', [100, 200])|	Vérifie si une valeur n'est pas dans un intervalle donné.|
|`whereExists($field)`	|whereExists('unemployed')	|Vérifie si une valeur est définie.|
|`whereNotExists($field)`	|whereNotExists('unemployed')	|Vérifie si une valeur n'est pas définie.|
|`whereRegexp($field, $value, $flags = 'ALL')`	|whereRegexp('name.raw', 'A.+')	|Filtre les enregistrements en fonction d'une expression régulière choisie. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/query-dsl-regexp-query.html#regexp-syntax) vous trouverez des renseignements sur la syntaxe.|
|`whereGeoDistance($field, $value, $distance)`|	whereGeoDistance('location', [-70, 40], '1000m')	|Filtre les enregistrements en fonction d'un point donné et d'un écart donné entre le point et la localisation. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-distance-query.html) des renseignements supplémentaies syntaxiques.|
|`whereGeoBoundingBox($field, array $value)`|	whereGeoBoundingBox('location', ['top_left' => [-74.1, 40.73], 'bottom_right' => [-71.12, 40.01]])|	Filtre les enregistrements avec des spécifications données. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-bounding-box-query.html) des renseignements supplémentaies syntaxiques.|
|`whereGeoPolygon($field, array $points)`	| whereGeoPolygon('location',[[-70, 40], [-80, 30], [-90, 20]])	| Filtre les enregistrements dans un polygone. [Ici](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-polygon-query.html) des renseignements supplémentaies syntaxiques.|

# Les autres filtres disponibles

|Method	| Example |	Description |
| :-: | :-: | :-: |
| `orderBy('champ','type d'ordre')` | orderBy('prix','desc')  |  Permet de classer les résultats en fonction d'un champ particulier |
| `from(int)` | from(15)  | Combien de résultats on saute avant de les prendre en compte ? Par défaut 0|
| `take(int)` | take(1000) | Combien de résultats on récupère en tout ? Par défaut 10 |
| `rule(SearchRule::class)` | rule(MySearchRule::class) | Indique dans le traitement global d'une requête qu'il faudra aussi utiliser le traitement de la classe SearchRule indiquée |
| `JSON()` | JSON() | Traduit l'ensemble du retour de la requête en JSON |
| `explain()` ou `profile()` | explain() ou profile() | Permettent d'avoir des informations supplémentaires dans le retour de la requête envoyée |

Il n'y a pas de conflits entre ces methodes et celles du query Builder. 