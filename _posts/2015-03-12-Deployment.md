---
layout: post
title: DEV box deployment
---


## 2016 way to release

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





