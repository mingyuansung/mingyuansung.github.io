---
layout: post
title: DOCKER EDEN, ECHO Installation extra other than Kevin's note
---  
 
##0. Docker
* install [Docker Toolbox](https://www.docker.com/products/docker-toolbox) first

##1. Eden

* get the latest master branch
* `docker-machine start default` to start the container
* `php -S localhost:8000 -t EDEN/sf2/web/` to start PHP server
* `cd EDEN`
* Make sure you have Mike’s [fullstack setup](https://github.com/mnichols1313/EDEN/commit/82334eaa58b7d48b62b14e4fe5cb04712d0e5766) downloaded to your EDEN directory
* `docker-compose -f docker-fullstack.yml up` to bring down (update) needed images and start EDEN



##2 Echo

* get the latest master branch
* make sure you have Java 7 installed
* `cd ECHO` directory
* bring service dependencies online with `docker-compose -f docker-dev.yml up -d`
* build and test the project: `./gradlew build`
* start the service: `./gradlew bootRun`
* test it out: `curl localhost:8080/health`
* you can also use the comand line: `java -jar -Xms1500m -Xmx1500m -XX:MaxPermSize=200m echo-server/build/libs/echo-server-3.1.7-SNAPSHOT.jar --spring.profiles.active=dev`

##3. Switching between EDEN and ECHO Docker

* you can run either or.  Not both at the same time
* `docker stop $(docker ps -q)` stop all currently running images
* `eval $(docker-machine env default --shell=bash)`

## 8. Useful command
* start php server: `php -S localhost:8000 -t EDEN/sf2/web/`
* docker stop all: `docker stop $(docker ps -q)`
* docker-machine ls
* docker-machine restart default
* docker-compose -f docker-dev.yml ps
* docker-compose -f docker-dev.yml stop mysql
* docker-compose -f docker-dev.yml rm mysql
* docker-compose -f docker-dev.yml up -d mysql


## 9. Useful notes
* you will need to populate MySQL DB content as well as MongoDB content
* alias docker_stop_all='docker stop $(docker ps -q)'
* if you ever need to install, make sure you `docker-machine rm default` to remove the container, then go to VirtualBOx UI to delete the default machine, then start from docker installation
* you can change your default VM to have 4 cores and 8GB ram, make things faster





