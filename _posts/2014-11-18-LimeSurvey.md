---
layout: post
title: What I know about Bloomery
---



## Install Our Modified LimeSurvey
* For bloomery, if you pull the latest and build it, the documentation will be built and available locally at `<your git dir>/bloomery/bloomery-server/build/asciidoc/html5/index.html`.  All of the endpoints are described there with sample payloads.
* It doesn't have its own db, uses the EDEN db only
* The `rs_smelter` tables will only get records when someone in quantum (answering a DF1502 request) enters smelters. It is like `cmrt_smelter`
* ECHO calls bloomery to export an RS DF1502 program into a CMRT
bloomery is called to retrieve the smelters
* They have implemented a DF1502 program for RS, that asks the same questions as the EDEN CMRT task flow, answers are stored in boson
* all the answers to the questions come from boson, but boson only stores the `rs_smelter.id` for each smelter that was entered.  So when all the answer data is passed to ECHO for export, ECHO calls bloomery to look up the `rs_smelter.id's` and fill in the smelter info
* ECHO doesn't look up anything in boson, the boson data is passed in the export request
* The path is quantum -> boson -> ToGo -> ECHO -> bloomery -> CMRT stored in docstore
* ToGo monitors and creates an "exportDone" event, quantum watches for that event
* What is the parameter to do smelter name search? `http://localhost:8080/processors/name-or-alias?q=mySearchString`

   
    
## Testing 
* The files containing the sql are in `bloomery-core/src/main/resources/db-migration` - the 2 tables are `rs_smelter` and `rs_smelter_provider` Those are the 2 tables that are similar to the `cmrt_smelter` table
* I run the `docker-compose up -d` (uses the docker-compose.yml that is in the bloomery repo) to bring up services and then run bloomery in IntelliJ if I want to debug it.
* hit endpoints with Postman
* There is a whole set of endpoints in bloomery that were written to handle smelter admin - so they are not used now.  I didn't take them out, but if they are never going to be used you might take them out eventually
* For some reason, on my local in `DocstoreClient.java` (and `BloomeryClient.java`) I have to set

```
private static final String FILES_URL = "http://boot2docker:32769/files";
```

instead of

```
"http://docstore/files
```

* I committed it without the port numbers because `consul` should be looking up docstore and bloomery services.  But my consul doesn't do it.  So I have to specify `boot2docker:<port number>`
* if the exception is because `docstore` isn't found, you can try changing that.  The port number shows when you do `docker-compose ps`
* Under Intellij 2016.3, the project module has more layers and the "echo-cmc_main" language level is "7", although the dependencies points to Java 1.8.  And "echo-cmc" sources language level is 8 from the beginning.  So kind of strange that I thought "echo-cmc" should be the base setting and apparently not.  Change all to level 8 fixed the compiling error.

![Dev Setting](https://mingyuansung.github.io/graphic/bloomery_conf_0.png)
![Dev Setting](https://mingyuansung.github.io/graphic/bloomery_conf_1.png)
![Dev Setting](https://mingyuansung.github.io/graphic/bloomery_conf_2.png)
![Dev Setting](https://mingyuansung.github.io/graphic/bloomery_conf_3.png)
![Dev Setting](https://mingyuansung.github.io/graphic/bloomery_postman_1.png)
![Dev Setting](https://mingyuansung.github.io/graphic/bloomery_postman_2.png)
