---
layout: post
title: Kubectl in Deployment
---

# 2018 Kubectl EDEN and ECHO Deployment


## Preparation

* Install Docker for Mac
* Get from Kevin your AWS credentials in `~/.aws/credentials`
* Get from [our repository](https://github.com/Source-Intelligence/k8s-prep/blob/master/kube-config.yaml) our `~/.kube/config`


## EDEN Deployment

* `kubectl config use-context dev2`
* `kubectl get deployment eden -o wide` to check current version of eden on dev2
* `kubectl set image deployment/eden eden=docker.sourceintelligence.net/eden:1.2.3` to deploy eden verion 1.2.3 to dev2
* `kubectl logs -f deployment/eden` view eden log

```
localhost$ cd eden
localhost\eden$ docker pull docker.sourceintelligence.net/svcdev
localhost\eden$ docker run docker.sourceintelligence.net/svcdev > ../docker-compose.yml
```


## ECHO Deployment



## Misc


## Screen Shots related

![Dev Setting](https://mingyuansung.github.io/graphic/echo_remote_debug_setting.png)