---
layout: post
title: EDEN and ECHO in D4M
---

# 2018 EDEN and ECHO development environment


## Preparation

* Install Docker for Mac
* Give more than 4GB ram to it after installation
* Set the following environment variables in your local shell which are used by git and flightplan deployments. This can be done in `~/.bash_profile` or equivalent.

 
```
export GIT_AUTHOR_NAME='Ming Sung'
export GIT_AUTHOR_EMAIL='msung@sourceintelligence.com'
export GIT_COMMITTER_NAME='Ming Sung'
export GIT_COMMITTER_EMAIL='msung@sourceintelligence.com'
export SI_DEPLOY_USER='msung'

```

* From your local command prompt, do 

```
docker volume create mysql-data
docker volume create mongodb-data
```
* After EDEN installation, MySQL and MongoDB will be local. So you can not have other local DB installation


## EDEN installation Guide

* Install Docker Sync `localhost$ gem install docker-sync`
* Download the latest development container config

```
localhost$ cd eden
localhost\eden$ docker pull docker.sourceintelligence.net/svcdev
localhost\eden$ docker run docker.sourceintelligence.net/svcdev > ../docker-compose.yml
```
* `localhost\eden$ docker-sync start`
* `localhost\eden$ docker-compose run --rm --service-ports eden7`
* Now you are inside the container
* `container$ docker-compose up -d`
* Now you restore all the EDEN MySQL Data, make sure you have `docker_eden_ls2` DB created and restore all Limysurvey data needed
* Connect to MongoDB and restore needed data
* `container$ cd sf2`
* `container\eden\sf2$ composer install`
* `container\eden\sf2$ php app/console assets:install web`
* `container\eden\sf2$ php app/console server:run 0.0.0.0:3000 -vvv`
* `localhost$ sudo ifconfig lo0 alias 10.254.254.254/24` for intelliJ debug purpose
* Your eden intelliJ can listen to port 3000 to debug


## ECHO installation Guide

* turn off echo in EDEN container
* `container$ cd echo`
* Update `.env` file to points to `eden`
* `container\echo$ cp ../docker-compose.yml .`
* Make sure the java8 environment has this line: `SERVICE_8080_NAME: echo`
* `container\echo$ docker-compose run --rm --service-ports java8`
* Now you are in the ECHO container
* `./gradlew bootRun` to run ECHO server
* Your echo intelliJ can listen to port 8000 and debug

## Misc

* `php -r \@phpinfo\(\)\; | grep php.ini` to find where is php.ini
*`docker ps -a` to list all containers
* `docker stop` and `docker rm` to remove containers
* `docker images` to list all the images.
* `docker rm mongodb-container` then `docker image rm -f e28cba33aad0` to remove the mongo image for example, so next docker-compose up will pull down a fresh new image to restart.
* `docker run -it --rm --mount source=mongodb-data,destination=/aaa krmcbride/debian bash` put yourself in the mongo data valume, then `cd /aaa` to change directory in order to remove the mongo lock file and fix the mongo lock issue.
* or you can try:
* `docker-compose run --rm mongodb rm /data/db/mongod.lock` then `docker-compose up -d mongodb`
* `docker run -it --rm --mount source=mysql-data,destination=/aaa krmcbride/debian bash`
* `docker exec -it dev2_eden_1 /bin/bash` put yourself in the dev-2 docker container
* you can test the batch command inside the container for example `php /var/www/project/app/console --env=prod --no-debug source44:cmrt:qc:flags:create 30`
* or `php /var/www/project/app/console --env=prod --no-debug source44:cmrt:smelter:unmatched:email:send --startDate "2017-12-01" --endDate "2017-12-28"`
* some commands

```
docker-sync stop
docker-sync clean
docker-sync start
```

### test cmrt bulk upload on dev-2

* `ssh dev-2`
* `docker exec -it dev2_eden_1 /bin/bash`
* `mkdir tmp`
* copy all the needed xlsx and scv to the /tmp directory
* from my local eden/tmp directory, do `scp cmrt_data.csv dev@dev-2:~/svcdev/` copy file into dev-2 first
* then `ssh dev-2`
* then `docker cp cmrt_data.csv dev2_eden_1:/var/www/project/tmp/` 
* then `docker exec -it dev2_eden_1 /bin/bash`
* then `cd tmp`
* then `php ../app/console source44:cmrt:bulk-upload 89667 cmrt_data.csv --verificationOnly` to test
 
## Test ECHO extraction, export
* turn on local echo server
* then use rabbit mq page at http://localhost:15672/#/queues/%2F/echo%2Freport
* login rabbit mq admin page as quest
* choose either extraction or report generation channel
![Dev Setting](https://mingyuansung.github.io/graphic/extraction_values.png)

* modify extraction related db record value 1st
![Dev Setting](https://mingyuansung.github.io/graphic/echo_extraction_test_1.png)

* fill in property and payload value for extraction, then post message
![Dev Setting](https://mingyuansung.github.io/graphic/extraction_values.png)

* modify report related db record value 1st
![Dev Setting](https://mingyuansung.github.io/graphic/echo_extraction_test_2.png)

* or fill in property and payload value for roll up export generation, then post message
![Dev Setting](https://mingyuansung.github.io/graphic/report_values.png)



## Screen Shots related

![Dev Setting](https://mingyuansung.github.io/graphic/echo_remote_debug_setting.png)