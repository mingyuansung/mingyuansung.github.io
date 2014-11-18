---
layout: post
title: LimeSurvey local testing
---


## Install Our LimeSurvey
* Get the source from [our github](https://github.com/Source-Intelligence/LimeSurvey)
* I copy the whole source structure under //EDEN/sf2/web so I can keep using my localhost url without setting anohter vhost.  You can choose your own setup.
* go to http://localhost/limesurvey/admin and follow the onscreen instrcutoin to create a DB on your local MySQL server.  I used the name `limesurvey3` and you can use something of your own. After completion, you will have a blank DB created in your local MySQL server.
* if you want to copy prod data, you can.  Just make sure to modify the follwoing.
    * `adminemail`, `bounce_email` value in the table `lime_survey`
    * `email` value in the table `lime_users` in case email send out to the world
    * `surveyls_url` in table `lime_surveys_languagesettings`. this is the end of survey auto forward url address. change to match your testing vhost.  For my own test, I use the value `http://localhost/app_dev.php/sourcelink/task/support/survey/end?token={TOKEN}`
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
        lime_survey.redirect_url: http://localhost/limesurvey/index.php
    ```



## Testing example
* 



