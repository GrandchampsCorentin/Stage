source : https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html

# Installation de Java

Java est nécessaire au bon fonctionnement d'ES. Il est nécessaire de vérifier qu'il est bien installé sur la machine:
`java -version`

Si non installé, suivre : https://docs.oracle.com/javase/8/docs/technotes/guides/install/linux_jdk.html#BJFGGEFG

Ajouter par la suite la variable JAVA_HOME:
```bash
sudo vim /etc/profile
```

Dans le fichier , renseigner le chemin:
```bash
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64/"
export PATH=$JAVA_HOME/bin:$PATH
```

Sauvegarder, recharger et vérifier:
```bash
source /etc/profile
echo $JAVA_HOME
```

# Installation d'Elastic Search

Suivre: https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

# Tips
```bash
#init ou systemD?
ps -p 1

#Elasticsearch can be started and stopped as follows:
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service

#liens vers le fichier de config
sudo vi /etc/elasticsearch/elasticsearch.yml
```