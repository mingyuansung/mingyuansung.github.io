---
layout: post
title: How I test LimeSurvey submssion
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


##Here are a survey high level algorithm
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




