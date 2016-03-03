---
layout: post
title: 2016 DOCKER EDEN, ECHO Installation extra after Kevin's note
---  
 
## 0. Docker
* install [Docker Toolbox](https://www.docker.com/products/docker-toolbox) first

## 1. EDEN

* get the latest EDEN master branch
* cd to your EDEN directory
* `docker-machine start default` to start the default docker machine
* start a PHP server at a differnt terminal session `php -S localhost:8000 -t EDEN/sf2/web/`
* Make sure you have Mike’s [fullstack setup](https://github.com/mnichols1313/EDEN/commit/82334eaa58b7d48b62b14e4fe5cb04712d0e5766) downloaded to your EDEN directory
* `docker-compose -f docker-fullstack.yml up` to bring down (update) needed docker images supporting EDEN operation
* you are done



## 2. ECHO

* get the latest ECHO master branch
* make sure you have Java 7 installed
* cd to your ECHO directory
* bring docker service dependencies online with `docker-compose -f docker-dev.yml up -d`
* build and test the project: `./gradlew build`
* start the ECHO server service: `./gradlew bootRun`
* test it out: `curl localhost:8080/health`
* you can also use the comand line to start ECHO server: `java -jar -Xms1500m -Xmx1500m -XX:MaxPermSize=200m echo-server/build/libs/echo-server-3.2.1-SNAPSHOT.jar --spring.profiles.active=dev` make sure the jar name is the one you want to use

## 3. Switching between EDEN and ECHO Docker

* you can run either or.  Not both at the same time because their Docker supporting images will compete for the same network port
* cd to either EDEN or ECHO directory
* `eval $(docker-machine env default --shell=bash)`
* `docker stop $(docker ps -q)` stop all currently running images
* then start the service you want to start
* if you wan to use EDEN MySQL with ECHO or ECHO MySQL with EDEN, you do:

~~~
        cd echo
        docker-dev up -d 
        docker-dev stop mysql
        cd ../eden
        docker-dev up -d mysql
~~~
* you can do the same thing for all the other supporting docker images for example mongodb, etc.
* if you want to run EDEN and ECHO at the same time, you will have to start ECHO either from command line or from withing intelliJ

## 4. Useful command
* start php server: `php -S localhost:8000 -t EDEN/sf2/web/`
* docker stop all: `docker stop $(docker ps -q)`
* docker-machine ls
* docker-machine restart default
* docker-compose -f docker-dev.yml ps
* to redownload for example MySQL image:

```
docker-compose -f docker-dev.yml stop mysql
docker-compose -f docker-dev.yml rm mysql
docker-compose -f docker-dev.yml up -d mysql
```
* to redownload all images

```
docker-compose -f docker-dev.yml stop
docker-compose -f docker-dev.yml rm -f
docker-compose -f docker-dev.yml up -d
```


## 5. Useful notes
* you will need to populate MySQL DB content as well as MongoDB content and make sure the DB name is `docker_eden`
* alias docker_stop_all='docker stop $(docker ps -q)'
* if you ever need to reinstall, make sure you `docker-machine rm default` to remove the container, then go to VirtualBox UI to delete the default machine, then start from docker toolbox installation
* you probably should change your default VM to have 4 cores and 8GB ram through VirtualBox UI, make things run faster
* Read [Mike's note](https://github.com/Source-Intelligence/svctemplate/wiki/Docker-Tips) for some helpful alias and how to set up data container to preserve DB data
* I am using [RoboMongo](https://robomongo.org/download) to manage MongoDB
* Keep smiling

## 6. My environment start up Script
* I use apple script to start my development environment
* I created a file named sieden.scpt 
    
```
tell application "iTerm"		tell the current terminal		tell the current session			set name to "PHP_Server"			write text "green"			write text "php -S localhost:8000 -t EDEN/sf2/web/"		end tell	end tell		make new terminal	tell the current terminal		activate current session		launch session "Default Session"		tell the last session			write text "title Docker_EDEN"			write text "green"			write text "cd ~/EDEN"			write text "bash --login '/Applications/Docker/Docker Quickstart Terminal.app/Contents/Resources/Scripts/start.sh'"			write text "clear;docker-machine start default"			write text "title Docker_EDEN"			write text "docker-compose -f docker-fullstack.yml up"		end tell	end tell		delay 20		make new terminal	tell the current terminal		activate current session		launch session "Default Session"		tell the last session			write text "title ECHO_Server"			write text "green"			write text "cd ~/ECHO ; java -jar -Xms1500m -Xmx1500m -XX:MaxPermSize=200m echo-server/build/libs/echo-server-3.2.1-SNAPSHOT.jar --spring.profiles.active=dev"		end tell	end tell		make new terminal	tell the current terminal		activate current session		launch session "Default Session"		tell the last session			write text "title WORK"			write text "cd EDEN;clear;git status"		end tell	end tellend tell

```
* Then in my `.profile` I put in the alias as `alias eden='osascript sieden.scpt'`
* also define the color green as `alias green=' echo -n -e "\033]6;1;bg;green;brightness;213\a"'`
* When computer bootup, I can open an [iterm](https://iterm2.com/), then type `eden` to start the environment for me. You can Ctrl-c out of each tab to restart for example PHP server, or stop command line ECHO for IntelliJ debugger or just use Docker ECHO.

## Run local EDEN with Docker and local ECHO

* In order to do this, you will need to implement the local setting as Mike stated in the section `Optional Confiuration` in [EDEN readme](https://github.com/Source-Intelligence/EDEN) to bypass Docker ECHO setting.  So EDEN will not call Docker ECHO but call local ECHO instead. Basically you need to create a new file  `parameters_local.yml` to tell ECHO is at your local.  And comment out ECHO server from docker setting from `parameters.yml`.
* Put in `echo.stub: true` instead to use Echo stub instread of real Echo.  Now this may get changed pretty soon. 
* Then start PHP server, then EDEN docker, then ECHO from either command line or from withing IntelliJ.  Just like the start up script I made ealier.
* You can open EDEN and also ECHO project in IntelliJ and run debugger on both at the same time.



## HAVE FUN


