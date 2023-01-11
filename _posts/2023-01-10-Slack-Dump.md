---
layout: post
title: Slack-Dump
---

# 2023 Slack Dump Any Interested


## ECHO Development Preparation

* cd echo
docker pull docker.sourceintelligence.net/svcdev
cp ../docker-compose.yml{,.`date +%s`}
docker run docker.sourceintelligence.net/svcdev > ../docker-compose.yml
docker-compose pull java8
* docker-compose run --rm java8
* as always.  and inside that container docker-compose up -d

## Stop, RM, start Rabbitmq image

* docker-compose stop rabbitmq \
    && docker-compose rm rabbitmq \
    && docker volume rm eden_rabbitmq \
    && docker-compose up -d rabbitmq

## related to BATCH-MSDS

* before my vacation i refactored the batch-msds module from the batch project out into its own batch-msds GH repository (the batch project is just being pulled in too many directions) and removed the the dependency on ElasticSearch, replacing it with Rabbit.  now that batch job writes msds info to rabbit and hubble consumes those messages and does the persistence.  this is nice because now hubble is the only place in our universe coupled to elasticsearch, which was my primary goal when i created that service from what used to be in echo.  that makes updating our ES cluster to 5.x easier.  i started looking at that upgrade locally and naturally everything exploded because their Java DSL has changed a little in the past 4 years (how dare they!)
* RIP s44-statbox-1, in case anyone missed it from a week or two ago.  R doodads are now on the batch host (gruul).
* the move toward running eden containers in production on edge hosts (like the other edge services) puts five hosts on the chopping block (s44-loadbox-1, s44-loadbox-2, s44-prodbox-3, s44-prodbox-5, s44-prodbox-7).  obviously before pulling that trigger, eden appendages like anymsds termination and direct login redirects, batch jobs, ad-hoc CLI stuff, have to be accounted for.

* this next msds batch job will be a little comical.  batch-msds at aws will boot, blast rabbitmq in ORD with messages, which hubble at ORD will consume and use to push updated msds data into elasticsearch back at aws


## fly deployment - the old way

* friendly reminder that the steps to deploy to production in the post-kubernetes world are:
* make sure you have the latest `~/.kube/config` file here `https://github.com/Source-Intelligence/k8s-prep/blob/master/kube-config.yaml`
* each project now has a `manifest.yaml` file, which is similar to the old `docker-compose.prod.yml`.  when you wan to deploy to prod, change the project version in `manifest.yaml` like you used to in `docker-compose.prod.yml` (but this time there's no A/B groups to worry about).
* instead of `fly deploy:prod` and going through the prompts, you just `kubectl --context prod1 apply -f manifest.yaml` and k8s does the rest.
there's a number of ways to observe the deployment in action:
* `kubectl --context prod1 get -f manifest.yaml -o wide -w` to watch changes to the Deployment
* `kubectl --context prod1 get pods -l app=<name> -w` to watch changes to Pods with an app label matching <name>, eg: quantum, eden, etc
* `kubectl --context prod1 rollout status -f manifest.yaml` to watch changes to the ReplicaSets
* Watch logs in Kibana and/or DataDog.  You can also view logs right in your terminal with kubectl logs but afaik you can't tail more than one container, which isn't very helpful during a deployment, so its probably better to stick to Kibana and DataDog. 

## Screen Shots related
