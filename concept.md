# Introduction

Elasticsearch est un moteur de recherche open-source, basé sur le software open source d'Apache *Lucene*, largement distribuable, facilement évolutif et de qualité professionnelle. Accessible grâce à une API étendue et élaborée, Elasticsearch peut alimenter des recherches extrêmement rapides.

# Du plus grand au plus petit : les termes importants

### Le Cluster

C'est l'entité qui regroupe tout et qui s'occupe de l'indexation et de la recheche.
Un *Cluster* est une collection composée de 1 à X *Nœuds* (Node) qui se partagent les données.

Chaque *Cluster* est identifié par un identifiant unique qui par défaut est "elasticsearch".
L'identifiant du *Cluster* permettra de l'associer avec des *Nœuds*. 

Il faut faire attention à ne pas réutiliser les mêmes noms de *Cluster* dans différents environnements, cela pourrait provoquer des erreurs de jointure entre *Nœuds* et *Cluster* ! /!\
Par exemple, vous pouvez utiliser logging-dev, logging-stage et logging-prod pour les clusters de développement, de transfert et de production.

*Ordre d'idée* : Parc informatique

*** Les Nœuds ou Nodes

Les *Nœuds* stockent vos données et soutiennent le travail d’indexation et de recherche du *Cluster* auquel ils sont liés (grâce à l'identifiant de celui-ci). 

Chaque *Nœud* possède son propre identifiant unique UUID (Universally Unique IDentifier) qui lui est assigné dès son démarrage. Ceci permet d'identifier quel serveur sur le réseau correspond à quel *Nœud* dans les *Clusters* existant. 

Par défaut, les *Nœuds* sont joins avec le *Cluster* identifié par "elasticsearch". 

*Ordre d'idée* : Serveur

### L'Index

Ici, un *Index* est une collection de documents qui ont quelque chose de similaire. 
Par exemple, il est possible d'avoir un index pour les données client, un autre pour un catalogue de produits, ou encore un autre pour les données concernant les commandes, etc...

Un *Index* est identifié par un nom, écris en minuscules, et est typé. 
Le type peut avoir n'importe quel appellation, mais pour des questions de maintenabilité, il sera préférable de donner le type *_doc* aux Index. 
Lors d'opérations sur des *Documents* telles que l'indexation, la recherche, la mise à jour, ou la suppression, on utilise ce nom afin de se référer à l'*Index* touché.

Dans un *Cluster*, on peut définir une infinité d'*Index*.

### Shards & Replicas

#### Shards 

Un *Index* peut contenir une masse très importante d'informations et de données, ce qui pourrait surcharger le hardware du *Nœud*. 

De ce fait il est possible de découper en plusieurs éclats (*Shards*) un *Index* afin qu'il soit contenu sur plusieurs *Nœuds*. 

L'idée sera donc de diviser le poids de l'*Index* et d'augmenter les performances de recherche en proposant plusieurs accès sur ce même Index.

#### Replicas

Ceci correspond à un système de sauvegarde dédié aux *Shards*, permettant de faire de la restauration de *Shards/Nœuds* si besoin est. 

On en tire deux gros avantages : 
* Une récupération et une disponibilité immédiate des copies lors de problèmes avec l'original
* La possibilité de faire des recherches en parallèle du premier *Shard* sur les *Shards* copiées et ainsi optimiser la recherche

Par défaut, chaque Index est découpé en 5 *Shards* et possède 1 *Replica*, soit, sur une configuration avec deux *Nœuds*, 10 *Shards* par Index (dont 5 *Shards* répliqués = 1 *Relica*).


*Ordre d'idée* : Disque Dur

### Les Documents

Écris en JSON, les *Documents* détiennent la matière première, l'information, et sont indexables.

Dans un même *Index*, il est possible de créer des relations parent/enfant entre deux Documents.

*Ordre d'idée* : Fichier

