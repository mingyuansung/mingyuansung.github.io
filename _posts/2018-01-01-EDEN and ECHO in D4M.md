---
layout: post
title: EDEN and ECHO in D4M
---

# 2018 EDEN and ECHO development environment

##Preparation

* Install D4M
* Set the following environment variables in your local shell which are used by git and flightplan deployments. This can be done in `~/.bash_profile` or equivalent.

 
```
export GIT_AUTHOR_NAME='Stephen Colbert'
export GIT_AUTHOR_EMAIL='scolbert@sourceintelligence.com'
export GIT_COMMITTER_NAME='Stephen Colbert'
export GIT_COMMITTER_EMAIL='scolbert@sourceintelligence.com'
export SI_DEPLOY_USER='scolbert'

```

* From your local command prompt, do 

```
docker volume create mysql-data
docker volume create mongodb-data
```
* After installation, MySQL and MongoDB will be local. So you can not have other local DB installation

##EDEN installation Guide

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
* `sudo ifconfig lo0 alias 10.254.254.254/24`
* Your eden intelliJ can listen to port 3000 to debug

## ECHO installation Guide

* turn off echo in EDEN container
* `cd echo`
* Update `.env` file to points to `eden`
* `cd echo`
* `cp ../docker-compose.yml .`
* Make sure the java8 environment has this line: `SERVICE_8080_NAME: echo`
* `container\echo$ docker-compose run --rm --service-ports java8`
* Now you are in the ECHO container
* `./gradlew bootRun` to run ECHO
* Your echo intelliJ can listen to port 8000 and debug

## Misc

* `docker ps -a` to list all containers
* `docker stop` and `docker rm` to remove containers
* some commands

```
docker-sync stop
docker-sync clean
docker-sync start
```

