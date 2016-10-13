---
layout: post
title: ECHO Installation Guide after EDEN Installation
---  
 
##0. Maven
* `brew install maven`

##1. RabbitMQ

* `brew install rabbitmq`
* `locate rabbitmq-server` to find where is your rabbitmq installation, then start it for example on my own computer `sudo /usr/local/bin/rabbitmq-server -detached && rabbitmq_log`

##2 ElasticSearch

* we need to use veriosn 0.19.12 right now
* create `elasticsearch-0.19.12.rb` and copy this [content](https://github.com/Source-Intelligence/misc/blob/master/homebrew/elasticsearch-0.19.12.rb) to your `/usr/local/Library/Formula` directory
* then run `brew install elasticsearch-0.19.12`
* to manually start it you do:
* `cd /usr/local/cellar/elasticsearch-0.19.12/0.19.12/bin`
* `elasticsearch -f`
* to make it automatically load when computer starts, you do:
* `ln -sfv /usr/local/opt/elasticsearch-0.19.12/*.plist ~/Library/LaunchAgents`
* `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.elasticsearch-0.19.12.plist`
* use this command to check elasticsearch process `ps ax | grep elastic | grep Cellar`

##3. Get ECHO source from Github


* copy `server/src/main/resources/config/application.dist.yml` to be `application.yml` and update the content depend on your own environment
* Here is mine:
    
```
    echo:
    sql:
        username: s44admin
        password: test
        primary_db: test
        ls2_db: limesurvey_3
        batch_db: test
    mongodb:
        primary_db: local
    es:
        index: s44_echo_server
        cluster: elasticsearch_mingsung
    elasticsearch:
    # autoCreateIndex: false
    properties:
        "path.logs": /usr/local/var/log/elasticsearch/client
```

## 8. Build and Test
* `mvn clean install`
* to compile and boot the server `mvn clean install -pl server -am -DskipTests && java -jar server/target/echo-server-<version>.jar`
* to boot the server without compiling `java -jar server/target/echo-server-<version>.jar`
* if want to put your applicaiotn yml at a different location and boot `java -jar -Dspring.config.location=/foo/bar/baz.yml server/target/echo-server-<version>.jar`
* use this command to check the health `curl localhost:8080/manage/health`
* `locate rabbitmq-server` to find where is your rabbitmq installation, then start it for example on my own computer `sudo /usr/local/bin/rabbitmq-server -detached && rabbitmq_log`

## 9. Useful alias
* alias rabbitmq_start='sudo /usr/local/bin/rabbitmq-server -detached && rabbitmq_log'
* alias rabbitmq_stop='sudo /usr/local/bin/rabbitmqctl stop'
* alias rabbitmq_restart='sudo /usr/local/bin/rabbitmqctl stop && /usr/local/bin/rabbitmq-server -detached && rabbitmq_log'




