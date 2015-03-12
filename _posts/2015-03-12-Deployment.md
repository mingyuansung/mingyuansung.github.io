---
layout: post
title: DEV box deployment
---


## Preparation Job for releasing from your local
* clone to your desktop https://github.com/source-intelligence/deploy.git
* check your /etc/hosts to have these entries and verify your access:
    * 192.237.167.95  s44-devbox-1
    * 104.130.195.197 s44-devbox-2
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
* if your local computer account is is the same as your account on DEV, you can just run `fab dev1` to target DEV1 release

## Notes

* 


