# Liste d'exemples de requêtes avec "searchRaw"

* Phrase - Un champ - Sans analyzer - La correspondance doit être de 100%

```php  
$maRequete = "une phrase au hasard";

$result = MyModel::searchRaw([
    'query' => [
        'match' => [
            'monChamp' => $maRequete,
        ],
    ],
]);  
```

* Phrase - Plusieurs champs - Avec analyzer - La correspondance peut ne pas être à 100%   

```php  
$maRequete = "une phrase au hasard";
$tabChamps = ['champ1', 'champ2', 'champ3'];
$typeTraitement = 'best_fields'; //Voir documentation 
$analyzer = 'french'; //Voir documentation

$result = MyModel::searchRaw([
    'query' => [
        'multi_match' => [
            'query' => $maRequete,
            'fields' => $tabFields,
            'type' => $typeTraitement,
            'analyzer' => $analyzer,
        ],
    ],
]);  
```

* Un terme - Un champ - La correspondance doit être de 100% - Pas analysable   
```php  
$monTerme = "test";

$result = MyModel::searchRaw([
    'query' => [
        'term' => [
            'monChamp' => $monTerme,
        ],
    ],
]);  

```
`test`
* Plusieurs termes - Un champ - La correspondance doit être de 100%  - Pas analysable   

```php  

$tabTermes = ["un", "deux", "trois", "quatre", "cinq"];

$result = MyModel::searchRaw([
    'query' => [
        'terms' => [
            'monChamp' => $tabTermes,
        ],
    ],
]);  
```