---
layout: post
title: 2016 DOCKER EDEN, ECHO Installation extra after reading Kevin's note
---  
 
## 0. Docker
* install [Docker Toolbox](https://www.docker.com/products/docker-toolbox) first
* you may need to `docker login docker.sourceintelligence.net` with your own password
* put the following alias in you profile: 
`alias docker-dev='docker-compose -f docker-dev.yml $([ -f docker-compose.yml ] && echo -n ''-f docker-compose.yml'')'`

## 1. EDEN

* get the latest EDEN master branch
* cd to your EDEN directory
* `docker-machine start default` to start the default docker machine
* start a PHP server at a differnt terminal session `php -S localhost:8000 -t EDEN/sf2/web/`
* in EDEN home directory, create a file named `docker-compose.yml` with the content:

```
version: '2'
services:
  mysql:
    volumes_from:
    - 'container:mysql-data'
  mongodb:
    volumes_from:
    - 'container:mongodb-data'
  elasticsearch:
    volumes_from:
    - 'container:elasticsearch-data'
    
```
* `docker-dev up -d` to bring down (update) needed docker images supporting EDEN operation and start them
* you are done



## 2. ECHO

* get the latest ECHO master branch
* make sure you have Java 8 installed
* cd to your ECHO directory
* bring docker service dependencies online with `docker-compose -f docker-dev.yml up -d`
* build and test the project: `./gradlew build`
* start the ECHO server service: `./gradlew bootRun`
* test it out: `curl localhost:8080/health`
* you can also use the comand line to start ECHO server: `java -jar -Xms1500m -Xmx1500m -XX:MaxPermSize=200m echo-server/build/libs/echo-server-3.3.4-SNAPSHOT.jar --spring.profiles.active=dev` make sure the jar name is the one you want to use

## 3. Run EDEN with local ECHO

* with EDEN started with all supporting docker images, you do `docker stop eden_echo_1` to stop the echo docker image
* make sure your EDEN parameters_local.yml has the echo server point to stub or localhost
* Start your local ECHO or run ECHO from intelliJ


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
* docker-dev up -d : Starts the things listed in the local docker-dev.yml
* docker-dev ps : Shows you what is running in docker container
* docker-dev up -d echo : Starts echo in docker container
* docker-dev stop echo : Stops echo in docker container
* docker-dev logs echo : Shows the stdout from echo
* docker-dev logs : Shows the stdout from all running images in container
* docker-dev stop : Stops all images in container
* docker ps -a : lists all the containers
   

## 5. Useful notes
* you will need to populate MySQL DB content as well as MongoDB content and make sure the DB name is `docker_eden`
* alias docker_stop_all='docker stop $(docker ps -q)'
* if you ever need to reinstall, make sure you `docker-machine rm default` to remove the container, then go to VirtualBox UI to delete the default machine, then start from docker toolbox installation
* you probably should change your default VM to have 4 cores and 8GB ram through VirtualBox UI, make things run faster
* Read [Mike's note](https://github.com/Source-Intelligence/svctemplate/wiki/Docker-Tips) for some helpful alias and how to set up data container to preserve DB data
* I am using [RoboMongo](https://robomongo.org/download) to manage MongoDB
* On DEV server, `/srv/s44/echo/logs/echo-server.log` is the echo log and you can just scp it to your local and view.  I am doing `scp msung@s44-devbox-2:/srv/s44/echo/logs/echo-server.log /Users/mingsung/downloads/.`

## 6. My environment start up Script
* I use apple script to start my development environment, sometimes I still need to stop , start, reset, etc. etc. after the script running. There all timing issues.
* I created a file named sieden.scpt, use at your own risk: 
    
```
tell application "iTerm"
	
	tell the current terminal
		tell the current session
			set name to "PHP_Server"
			write text "green"
			write text "php -S localhost:8000 -t EDEN/sf2/web/"
		end tell
	end tell
	
	make new terminal
	tell the current terminal
		activate current session
		launch session "Default Session"
		tell the last session
			write text "title Docker_EDEN"			write text "green"			write text "cd ~/EDEN"			write text "bash --login '/Applications/Docker/Docker Quickstart Terminal.app/Contents/Resources/Scripts/start.sh'"			write text "clear"			write text "docker-machine restart default"			write text "eval $(docker-machine env default --shell=bash)"			write text "docker-dev up -d"			write text "docker stop eden_echo_1"
			write text "docker-dev logs"					end tell
	end tell
	
	delay 25
	
	make new terminal
	tell the current terminal
		activate current session
		launch session "Default Session"
		tell the last session
			write text "title ECHO_Server"
			write text "green"
			write text "cd ~/ECHO ; java -jar -Xms1500m -Xmx1500m -XX:MaxPermSize=200m echo-server/build/libs/echo-server-3.3.4-SNAPSHOT.jar --spring.profiles.active=dev"
		end tell
	end tell
	
	make new terminal
	tell the current terminal
		activate current session
		launch session "Default Session"
		tell the last session
			write text "title WORK"
			write text "cd EDEN;clear;git status"
		end tell
	end tell
end tell

```
* Then in my `.profile` I put in the alias as `alias eden='osascript sieden.scpt'`
* also define the color green as `alias green=' echo -n -e "\033]6;1;bg;green;brightness;213\a"'`
* When computer bootup, I can open an [iterm](https://iterm2.com/), then type `eden` to start the environment for me. You can Ctrl-c out of each tab to restart for example PHP server, or stop command line ECHO for IntelliJ debugger or just use Docker ECHO.

## Run local EDEN with Docker and local ECHO

* In order to do this, you will need to implement the local setting as Mike stated in the section `Optional Confiuration` in [EDEN readme](https://github.com/Source-Intelligence/EDEN) to bypass Docker ECHO setting.  So EDEN will not call Docker ECHO but call local ECHO instead. Basically you need to create a new file  `parameters_local.yml` to tell ECHO is at your local.  And comment out ECHO server from docker setting from `parameters.yml`.
* Put in `stub_services: [echo]` instead to use Echo stub instread of real Echo.  Now this may get changed pretty soon. 
* Then start PHP server, then EDEN docker, then ECHO from either command line or from withing IntelliJ.  Just like the start up script I made ealier.
* You can open EDEN and also ECHO project in IntelliJ and run debugger on both at the same time.



## HAVE FUN


