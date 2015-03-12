---
layout: post
title: DEV box deployment
---


## Preparation Job for releasing from your local
* clone to your desktop from https://github.com/source-intelligence/deploy.git
* check your /etc/hosts to have these entries and verify your DEV access:
    * 192.237.167.95  s44-devbox-1
    * 104.130.195.197 s44-devbox-2
* check with your boss if you have no devs and sudo permission on DEV
* `sudo easy_install pip`
* `sudo pip install Fabric`
* `fav --version` to verify your installation
* `sudo pip install jsonpickle`
* `sudo pip install props`
* copy your local `//deploy/fabfile/props.dist.py` to be `//deploy/fabfile/props.py`

## Release directly on DEV box
* `cd /srv/s44/eden`
* `phing app:all`


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





