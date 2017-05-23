---
layout: post
title: DEV box deployment
---

## 2017 EDEN release to Production

Preparation

* You need to have access and connect to development VPN
* You need to install flightplan by "npm install -g flightplan"
* You need to install PHPRelease, follow the "Install" instructions to install PHPRelease, described in https://github.com/c9s/PHPRelease/
* You need to have access to "http://kibana.query.s44:5601/"

Step-by-step guide

* Update your local EDEN master branch to the latest
* Merge the release target PR branch
* Do "phprelease --bump-minor" for example to bump the revision number in EDEN/sf2/src/Source44/Support/Version.php
* Update CHANGELOG.MD following the existing format
* Commit these 2 changes
* Push to SI master
* Wait till the image been build
* Test the build on dev server
* Go to your local EDEN master branch when things are fine
* Do `fly status:prod` to know which group of EDEN servers is running now, a or b?
* Vim docker-compose.prod.yml to update the next EDEN group to the target revision number
* Do `fly deploy:prod` to deploy
* Before you choose to dump the existing group of server, use `kibana.query.s44:5601` to check the target eden server completed build or not
* You can then go to Prod site to check/verify EDEN revision from UI
* After you're done, go back to veriy your local docker-compose.prod.yml for the correct EDEN revision
* Commit and Push to SI master
* YOU ARE DONE

## 2016 Docker way to release

Make sure you have the folling in your `~/.ssh/config` for our dev servers definition

```
Host dev01
  User dev01
  HostName controller01-sfo1-do.sourceintelligence.net
  ForwardAgent yes
Host dev02
  User dev02
  HostName controller01-sfo1-do.sourceintelligence.net
  ForwardAgent yes
Host dev03
  User dev03
  HostName controller01-sfo1-do.sourceintelligence.net
  ForwardAgent yes
```

1. merge your branch into master or use your branch
2. fetch the master branch to your local machine (`git fetch SI`)
3. create whatever tag you want (`git tag tagname`)
4. push a tag (`git push SI tagname`)
5. watch for the build status of your tag in #dockerhub 
6. `ssh dev03` or the other
7. edit `docker-compose.overrride.yml`, specify the new tag, save the change
8. `docker-compose up -d` or you can just `docker-compose up -d eden`
9. watch the log for the deploy to finish `docker logs -f dev03_eden_1`




   

---

## Before 2016

---

## Preparation Job for releasing from your local
* clone to your desktop from https://github.com/source-intelligence/deploy.git
* check your /etc/hosts to have these entries and verify your DEV access:
    * 192.237.167.95  s44-devbox-1
    * 104.130.195.197 s44-devbox-2
* check with your boss if you have no devs and sudo permission on DEV
* then do the following installations:
    * `sudo easy_install pip`
    * `sudo pip install Fabric`
    * `fav --version` to verify your installation
    * `sudo pip install jsonpickle`
    * `sudo pip install props`
* copy your local `//deploy/fabfile/props.dist.py` to be `//deploy/fabfile/props.py`

## Release directly on the DEV box
* login of course to the DEV box
* `cd /srv/s44/eden`
* `phing app:all`
* answer all the questions
* if the build seems to hang on installing vendors, `Ctrl+c` out, go to `releases/<your release>/sf2`, then run `composer install`
* repeat if hangs and you will get through
* go to vendor direcotry and do `df -h` to check if those numbers change to tell if the process hangs or not


## Release from your local computer
* `fab fullstack -H msung@s44-devbox-1`
* pick the branch you want to release to DEV and answer all the questions
* if your local computer account is the same as your account on DEV, you can just run `fab dev1` to target DEV1 release

## Other

* if you don't want to enter password again and again on DEV box, you can do:
    * on dev box `mkdir ~/.ssh`
    * on dev box `chmod 700 ~/.ssh`
    * on local box `scp ~/.ssh/id* msung@s44-devbox-1:~/.ssh/.` 
    * on local box `cat id_rsa.pub | ssh msung@s44-devbox-1 'cat >> .ssh/authorized_keys'`
* when later release process prompt for `Enter passphrase for key '/home/msung/.ssh/id_rsa'`, it will be your local computer password, not your DEV password unless you update it to be the same





