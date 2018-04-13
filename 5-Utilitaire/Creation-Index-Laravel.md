#Créer rapidement un index dans laravel

* 1 - **L'index configurator**  

> `php artisan make:index-configurator App\IndexConf\MyIndexConfigurator`

* 2 - **L'index**  

> `php artisan elastic:create-index App\IndexConf\MyIndexConfigurator`  

* 3 - **Le Modèle**  

> `php artisan make:searchable-model App\Models\MyModel --index-configurator=App\IndexConf\MyIndexConfigurator`

* 4 - **Indexer les données**  
  * *Par importation d'une BDD*   

  > `php artisan scout:import App\Models\Langue`  

  * *Par enregistrement d'une collection*   
  
  > `$maCollection->save(); OU $maCollection->searchable();`