---
layout: post
title: How I test LimeSurvey submssion
---

## From Mike:

Let me preface this with the fact that Kevin will have many improvements to this process if he is able to help.  All of this can be done more easily if we invest some time into better admin tools and/or survey support. This was what I was doing as a work-around.

* Port the survey from Prod to DEV for testing.  No need to do anything on Prod.
* Run survey creation ECHO API on ECHO Server.  This need to be done on Prod when ready.
* Create an assessment if there is one. 
	* I typically export the questions from LimeSurvey and concatenate the question group and question title to create a platform agnostic identifier. The export should include all the answer options. These go into a google doc that can be shared with the PM. See https://docs.google.com/spreadsheets/u/1/d/1enhe7ex1Bk-SXkDf0dRHzPqSvbgDRltUZgQpQ3CY1YA/edit?usp=drive_web
	* Most assessments only require statement expressions with the following:
		* The survey2_question_id
		* Point value
		* Type (See SurveyStatementExpression.Type)
		* An operator (See SurveyStatementExpression.Operator)
		* An operand (the value to compare e.g. the answer)
	* Create a lookup using the identifier created with the question group and the question title to lookup the survey2 ID. You will need these for dev and prod envs.
	* Once you have the data you need for the assessment, you can export the csv and use the tool in misc/tools/limesurvey2/assessment to create the JSON blob you can import into echo
	* Use the ECHO API to import the assessment
	* An assessment that has no expressions e.g. no possible points, will render a datatable without the dashboard that shows scores.
* Create a survey tag - see the ECHO API for tags - tags are what make the survey visible in the intelligence section.
* Always review the survey config in the LS admin to make sure the settings are correct. See other active surveys for examples. Things like:
	* Redirect URL - this is what triggers the import into eden/echo
	* Token based persistence - all surveys should use token based persistence. A token table will be created when you activate the survey
	* If the token is not active, you will get an error when you attempt to start a survey task.

-----

* PM create survey within Limesurvey
    * Limesurvey is in Docker container now.
    * use `docker-compose ps` to get your docker limesurvey ip     
    * on DEV1, `https://dev-svc-1-survey.sourceintelligence.net/admin/` adn admin and regular admin paswd.
    * on your local, `http://boot2docker:8001/admin`. You can update the admin id email and paswd in `docker_ls2` table.
    * use this command to copy dev1 mongo eden content to your local computer, then you can load it into your docker mongo or whatever. `docker run -it --rm -v $PWD/dump:/root/dump docker.sourceintelligence.net/mongo:2.6 mongodump --host dev-1.node.us-west-2.s44 --db docker_eden` You have to run it under docker environment of course, for example the environment you start up the container.
    * use `bsondump collection.bson > collection.json` to convert the download to json if you need.  You need to have local mongo installation to have bsondump.
* PM give a spreadsheet of survey field definition with score definition to Mike. You can ask either Mike or Broke to get a copy of that spreadsheet.
* Mike enters them (using ECHO script) to EDEN DB, especially score definitions.  `survey2_*` tables. Mike run command (ECHO) to import the survey from limesurvey to mongo and EDEN DB. These are the survey2_* tables in EDEN db.
    * do `docker exec -it dev1_echo_1 /bin/bash` to switch your self to echo container
    * do `curl localhost:8080/survey` to display a list of available APIs.  We need to run 2. 1st to generate survey2 records by survey import api, 2nd to generate all the assessment and expression records by assessment creation api.
    * the 1st one we need is `localhost:8080/survey/provider/{customerCompanyId}/{providerName whihc is LS2}/{providerId which is the survey id}`, then data will be ported into EDEN survey2_* tables.
    * The 2nd one, for self_assessment fields, Mike already updated the ECHO API to support it.
    * `curl -X POST -H "Content-Type: application/json" -d '{"name":"Results","isActive":"true","isLowerBetter":"false","ignoreUnansweredQuestions":"true","numDecimalPlaces":"1"}' localhost:8080/survey/LS2/674424/assessment`
    * ECHO PR #241 would explain the self assessment fields.  
* Here is the sampel to use ECHO API to create survey tag for survey 462943 pre AWS era:
    * `curl -X POST -H "Content-Type: application/json" -d '{"type":"DOMAIN","value":"Simple Test"}' localhost:8080/survey/LS2/462943/tag`   
* User filled in survey, after submittion, Limesurvey store data in its own DB, ECHO put all data in Mongo, some definition in EDEN. `survey2_*` tables.
* When customer go to EDEN Intelligence page, survey result can be view there with score.
* When the survey page loaded there, EDEN calls ECHO to pull data from Mongo and calculate score on the fly based on score definition in EDEN db. Score data not saved, just calculated for display and export purpose.
* Here is the prod limesurvey site: `http://survey2.sourceintelligence.net/admin/`
* Here are the 8 commands I ran to re-create Macy's survey tags:

	```
curl -X DELETE localhost:8080/survey/tag/6
curl -X DELETE localhost:8080/survey/tag/14
curl -X DELETE localhost:8080/survey/tag/17
curl -X DELETE localhost:8080/survey/tag/18
curl -X POST -H "Content-Type: application/json" -d '{"type":"DOMAIN","value":"Trim Survey"}' localhost:8080/survey/LS2/774668/tag
curl -X POST -H "Content-Type: application/json" -d '{"type":"DOMAIN","value":"Trim Survey"}' localhost:8080/survey/LS2/984933/tag
curl -X POST -H "Content-Type: application/json" -d '{"type":"DOMAIN","value":"CM Policy"}' localhost:8080/survey/LS2/67574/tag
curl -X POST -H "Content-Type: application/json" -d '{"type":"DOMAIN","value":"CM Policy"}' localhost:8080/survey/LS2/74785/tag
	```
* Here is to deploy to prod:
	
	```
	connect vpn
	ssh -p 2200 deploy@swarmctl.query.s44
	docker exec -it echo_server_a_1 bash
	```
	
---

## The survey deployment process

* Get the node scripts from source control //MISC/tools/limesurvey2/assessment directory
* The needed files are index.js, package.json
* This is node applicaiton, so you need to install node
* There are 2 node library modules need to install `sudo npm install fast-csv --save` and `sudo npm install jsonfiles --save`
* You need to build expressions.csv similiar as the one in the source control.
* Update index.js content for the survey name and the environment variable between dev and prod.
* Then run `node index.js`, it will generated the assessment json for you to use when run ECHO API.  Those json files in the source control are example samples.
* Once you have all the json needed, you need to run ECHO API on the ECHO server.
* Here is how to get youself to dev echo docker container to deploy to dev-1:
	
	```
	connect vpn
	ssh dev-1
	docker exec -it dev1_echo_1 /bin/bash 
	```
* Here is to get yourself to prod echo docker container to deploy to prod:
	
	```
	connect vpn
	ssh -p 2200 deploy@swarmctl.query.s44
	docker exec -it echo_server_a_1 bash
	```
* Then first step you need to import survey, for example `curl localhost:8080/survey/provider/19939/LS2/774668`
* Then second step is to create assessment if the target survey has any assessment defined.  If there's none, then you are done this step.  No need to run this step. The API command for example `curl -X POST -H "Content-Type: application/json" -d '{"name":"Results","isActive":"true","isLowerBetter":"false","ignoreUnansweredQuestions":"true","numDecimalPlaces":"1"}' localhost:8080/survey/LS2/774668/assessment` or load a json file you previously created if the json is too long to type.
* Then finally the third and last step is to create survey tag for example `curl -X POST -H "Content-Type: application/json" -d '{"type":"DOMAIN","value":"Trim Survey"}' localhost:8080/survey/LS2/774668/tag`
 
---
<!---->![Dev Setting](https://mingyuansung.github.io/graphic/<!---->BridgeKeepr_Dev_Setting.png)

---

## BEFORE 2017
---

## Install Our Modified LimeSurvey
* Get the source from [our github](https://github.com/Source-Intelligence/LimeSurvey)
* I copy the whole source structure under //EDEN/sf2/web so I can keep using my localhost url without setting vhost.  You can choose your own setup.
* go to http://localhost/limesurvey/admin and follow the onscreen instruction to create a DB in your local MySQL server.  I used the name `limesurvey3` and you can use anything of your own. After completion, you will have a blank DB created in your local MySQL server.
* if you want to copy prod data, you can.  Just make sure to modify the follwoing.
    * you may want to keep your own `lime_users` record if you do not know prod user password. just make sure to update the uid in other tables for example `owner_id` in table `lime_surveys`     
    * `adminemail`, `bounce_email` value in the table `lime_survey`
    * `email` value in the table `lime_users` in case email send out to the world
    * `surveyls_url` in table `lime_surveys_languagesettings`. this is the end of survey auto forward url address. change to match your testing vhost.  For my own test, I use the value `http://localhost/app_dev.php/sourcelink/task/support/survey/end?token={TOKEN}`
    * these are minimum needed tables:
        * `lime_question_attributes`
        * `lime_questions`
        * `lime_groups`
        * `lime_surveys`
        * `lime_surveys_languagesettings`
        * `lime_survey_{survey-id}`
        * `lime_tokens_{survey-id}`
        * `lime_users`
* under EDEN parameters.yml here the changed value to match my directory and db setting:

```
        lime_survey2.database_driver: pdo_mysql
        lime_survey2.database_host: 127.0.0.1
        lime_survey2.database_port:
        lime_survey2.database_name: limesurvey3
        lime_survey2.database_user: s44admin
        lime_survey2.database_password: test
        lime_survey2.base_url: http://localhost/limesurvey
        lime_survey2.rpc_user:
        lime_survey2.rpc_password:
        lime_survey2.getdoc_baseurl:

        # Additional Lime Survey parameters
        lime_survey.getdoc_baseurl: http://localhost/limesurvey/s44_custom/getdoc.php
```



## Mongo DB
* you need to have local mongo db running. if not, please refer to installation note in a different docuemnt to install it
* and you need rockmongo as admin UI or any other tools you are familiar with
* on my computer, I use `local` as the name of DB (you can use whatever name) and setup your parameters.yml

```
        # MongoDB settings
        mongodb.database_name: local
        mongodb.connection.server: mongodb://localhost:27017
        mongodb.connection.options: { connect: true }

        # MongoDB session storage
        session.storage_handler.mongodb.server: mongodb://localhost:27017
        session.storage_handler.mongodb.options: { connect: true }
```

* create a table name `task_flow`
* I insert the following value for my test. change the values for your LS test.

```
    {
      "_id": ObjectId("53eb995504e835a603b938de"),
      "comment": "Macy's 2014 Trim Survey",
      "name": "Macy's 2014 Trim Survey",
      "steps": [
        {
          "name": "survey",
          "initParams": {
            "surveyId": 92662
          }
        }
      ]
    }
```
    
* the surveyId is the id from limesurvey db for the survey you want to test
* the objectid value will match the `flow_id` value in table `task_campaign`
* you will be able to find an existing survey name you want to test in `task_campaign` table
* make sure you update the `company_id` value in table `task` linked by `campaign_id` to match your testing login user's company
    
    
## Testing (make sure you use your own survey url)

* you updated the `company_id` in table `task` for your testing login user's company
* update `state_current_step` to `survey` and `state_completed_steps_json` to `[]` and `state_compelted_at` to null and `first_compelted_at` to null.
* you need to login as `role_social_admin`
* if the survey has already been submitted, go to table `lime_survey_result` and look for `task_id` to match your test, set `compelted_at` to null. please use null instead of blank.
* go to limesurvey db, update table `lime_tokens_74785` that the number is the survey number, set `usesleft` to `1` and `compelted` to `N`
* go to limesurvey db, update table `lime_surveys_language` for survey id is 74785, set `surveyls_url` to `http://localhost/app_dev.php/sourcelink/task/support/survey/end?token={TOKEN}`
* now you login and go to `http://localhost/app_dev.php/sourcelink/profile#tasks`, the task will be available for you to fillout.

## Example
* now let's say I want to test "2014 Conflict Minerals Policy Upload" limesurvey.
* I go to EDEN `task_campaign` table look for name contains "2014 Conflict Minerals Policy Upload" and I find record with id 330 and 331.
* I then go to EDEN `task` table and look for `campaign_id` equals 330 or 331.
* I find many. then I filter by company_id=1
* I pick randonly record id=82782 for my test.
* set this record `state_current_step` to "survey" and `state_compelted_steps` to "[]" and reset `state_comeplted_at` and `first_comeplted_at` value to NULL.
* go to EDEN `lime_survey_result` table to search if any record has `task_id` equal to 82782.
* I find one, then set the `completed_at` value to NULL.
* from this record, I know the survey id is 67574 and token is e3f18f0ab3174b27978a11be77d57067. these values should match values in limesurvey db.
* now you need an entry in mongo db under the task flow as follows.

```
    {
      "_id": ObjectId("542c64e204e835f22ce58d04"),
      "comment": "Macy's 2014 Conflict Minerals Policy Acknowledgement",
      "name": "Macy's 2014 Conflict Minerals Policy Acknowledgement",
      "steps": [
        {
          "name": "survey",
          "initParams": {
            "surveyId": 74785
          }
        }
      ]
    }
```

* at this moment, refresh `http://localhost/app_dev.php/sourcelink/profile#tasks` this survey task should be listed under Active Reqeusts section and GO button appears.
* now I copy PROD limesurevey table `lime_survey_67574` and `lime_tokens_76574` to my local limesurvey db.
* I go to my local `lime_tokens_67574` and look for token=e3f18f0ab3174b27978a11be77d57067 record. then I update this record `compelted` to "N" and `usedleft` to "1".
* Now I am ready to go back to `http://localhost/app_dev.php/sourcelink/profile#tasks` and click on the GO button to test the survey.


## Here are a survey high level algorithm
* When you login EDEN and if a lime survey is waiting for you, you click and go to for example: http://localhost/app_dev.php/sourcelink/task/support/campaign/4cbb8338-1a14-402f-abb5-513729e5caa1/participate
* Then EDEN will forward you to http://localhost/app_dev.php/sourcelink/task/cm-data/370d7c72-2442-4aa9-b5c7-b0f2b2e992b7
* This page will show you some some survey reason and you can click to enter the lime survey.
* Then EDEN will forward you to lime for eample: https://survey2.sourceintelligence.net/92662?token=d2c2fbeef18d43b6a18eead80cb56055
* After you complete the survey and click submit, lime will forward you to http://localhost/app_dev.php/sourcelink/task/support/survey/end?token=370d7c72-2442-4aa9-b5c7-b0f2b2e992b7 This is defined in lime `lime_surveys_languagesettings` table.
* EDEN will then update couple db content, then forward user again for example to http://localhost/app_dev.php/sourcelink/task/cm-data/370d7c72-2442-4aa9-b5c7-b0f2b2e992b7
* Now this page will display a different content than above mentioned (you noticed this URL appears previously, right?).  
* It will show you survey already completed.
* Two important piece of codes:
* TaskControllerListener.assertTaskAccessibility() to check if continue loading the task or not.
* ProfileRequestsController.openRequestsDataTable() to show the Active Reqeusts Section listing.




