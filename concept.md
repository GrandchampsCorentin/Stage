# Introduction

Elasticsearch est un moteur de recherche open-source, basé sur le software open source d'Apache *Lucene*, largement distribuable, facilement évolutif et de qualité professionnelle. Accessible grâce à une API étendue et élaborée, Elasticsearch peut alimenter des recherches extrêmement rapides.

–          L’utilisation d’une base de données NoSQL

–          Une forte scalabilité

–          L’utilisation d’une API REST/HTTP/JSON

–          Des temps de réponses compris entre 20ms et 300ms
Il est distribué (architecture de type cloud computing).
Il a une scalabitlité horizontale : possibilité de rajouter des nœuds au cluster pour augmenter sa capacité de traitement
C’est un moteur de recherche « noSPOF » (no Single Point Of Failure). Si un nœud tombe, le cluster fonctionne toujours.
Il utilise une base de données NoSQL horizontale.
Il utilise la méthode REST pour gérer ses index : L’indexation des données s’effectue à partir de requêtes HTTP PUT. La recherche des données s’effectue avec des requêtes HTTP GET. Le flux d’informations est codé selon le format JSON.
Il est associé à deux autres produits open source que sont Kibana et Logstash : ceux-ci sont respectivement un outil de visualisation de données et un ETL.