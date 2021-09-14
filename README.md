# Alfresco Community 7
Transfer data from a alfrescoA to a alfrescoB

## How do I get set up?

```
$ curl https://artifacts.alfresco.com/nexus/content/groups/public/org/alfresco/alfresco-ftr-distribution/7.0.0/alfresco-ftr-distribution-7.0.0.zip --output alfresco-ftr/alfresco-ftr-distribution-7.0.0.zip

$ unzip alfresco-ftr/alfresco-ftr-distribution-7.0.0.zip -d alfresco-ftr/alfresco-ftr-distribution-7.0.0

$ docker-compose up -d
```

## Option #1 - Importing and transferring files

**[https://docs.alfresco.com/content-services/community/admin/import-transfer/](https://docs.alfresco.com/content-services/community/admin/import-transfer/)**

1. Go to alfrescoA repository and create a myfolder directory with files via shareA
    * http://localhost:8081/share
2. According to the documentation it is necessary to create a "replica"
![](/screenshots/screenshot1.png)
3. Create job to copy documents from "myfolder" to transfer target
4. Once the transfer is complete, access the alfresco bulkimport and import the data
![](/screenshots/screenshot2.png)
5. Access shareB and check that the import is OK
    * http://localhost:8083/share

### Pros & Crons ###
    * Very fast process
    * Doesn't keep UUIDs, create user, etc

## Option #2 - Replication data

**[https://docs.alfresco.com/content-services/community/admin/replication/](https://docs.alfresco.com/content-services/community/admin/replication/)**

1. Go to alfrescoA repository and create a myfolder directory with files via shareA
    * http://localhost:8081/share
2. According to the documentation it is necessary to create a "replica"
![](/screenshots/screenshot3.png)
3. Create job to copy documents from "myfolder" to transfer target
4. Access shareB and check that the import is OK
    * http://localhost:8083/share

### Pros & Crons ###
    * Keeps UUIDs, creator user, etc
    * Not a fast process

## Option #3 - ACP Export & Import

1. Install js-console on alfresco and share
*[in another terminal]*
```
$ git clone https://github.com/larede/js-console
$ cd js-console
$ mvn package

$ docker cp javascript-console-share/target/javascript-console-share-0.7-SNAPSHOT.amp $(docker ps | grep shareA | awk '{print $1}'):/tmp
$ docker exec -it -u root $(docker ps | grep shareA | awk '{print $1}') bash
    $ java -jar /usr/local/tomcat/alfresco-mmt/alfresco-mmt*.jar install /tmp/javascript-console-share-0.7-SNAPSHOT.amp  /usr/local/tomcat/webapps/share -nobackup -verbose
    $ exit
$ docker restart $(docker ps | grep shareA | awk '{print $1}')

$ docker cp javascript-console-repo/target/javascript-console-repo-0.7-SNAPSHOT.amp $(docker ps | grep alfrescoA | awk '{print $1}'):/tmp
$ docker exec -it -u root $(docker ps | grep alfrescoA | awk '{print $1}') bash
    $ java -jar /usr/local/tomcat/alfresco-mmt/alfresco-mmt*.jar install /tmp/javascript-console-repo-0.7-SNAPSHOT.amp  /usr/local/tomcat/webapps/alfresco -nobackup -verbose
    $ exit
$ docker restart $(docker ps | grep alfrescoA | awk '{print $1}')

$ docker cp javascript-console-share/target/javascript-console-share-0.7-SNAPSHOT.amp $(docker ps | grep shareB | awk '{print $1}'):/tmp
$ docker exec -it -u root $(docker ps | grep shareB | awk '{print $1}') bash
    $ java -jar /usr/local/tomcat/alfresco-mmt/alfresco-mmt*.jar install /tmp/javascript-console-share-0.7-SNAPSHOT.amp  /usr/local/tomcat/webapps/share -nobackup -verbose
    $ exit
$ docker restart $(docker ps | grep shareB | awk '{print $1}')

$ docker cp javascript-console-repo/target/javascript-console-repo-0.7-SNAPSHOT.amp $(docker ps | grep alfrescoB | awk '{print $1}'):/tmp
$ docker exec -it -u root $(docker ps | grep alfrescoB | awk '{print $1}') bash
    $ java -jar /usr/local/tomcat/alfresco-mmt/alfresco-mmt*.jar install /tmp/javascript-console-repo-0.7-SNAPSHOT.amp  /usr/local/tomcat/webapps/alfresco -nobackup -verbose
    $ exit
$ docker restart $(docker ps | grep alfrescoB | awk '{print $1}')
```
https://gist.github.com/larede/7cecb42de7f3cfe0eafc188bd23b2fd7

2. Export backup ACP on alfrescoA via share and download it
![](/screenshots/screenshot4.png)
3. Import local file Myfolder.acp on alfrescoB via share
4. Import backup ACP on alfrescoB via share
![](/screenshots/screenshot5.png)

### Pros & Crons ###
    * Documentation is not official
    * Very slow process
    * Not keep UUIDs

## Option #4 - Import ACP backup with Bootstrap Content Extension Point

**[https://docs.alfresco.com/content-services/community/develop/repo-ext-points/bootstrap-content/](https://docs.alfresco.com/content-services/community/develop/repo-ext-points/bootstrap-content/)**


```
$ git clone git@github.com:larede/alfresco-sdk-samples.git
$ cd alfresco-sdk-samples
$ git checkout alfresco-51
$ cd all-in-one/bootstrap-content-repo
$ mvn package

$ docker cp target/bootstrap-content-repo-1.0-SNAPSHOT.jar $(docker ps | grep alfrescoB | awk '{print $1}'):/usr/local/tomcat/webapps/alfresco/WEB-INF/lib
$ docker restart $(docker ps | grep alfrescoB | awk '{print $1}')
```

### Pros & Crons ###
    * Very slow process
    * keeps UUIDs
    * Doesn't work with a large amount of data
