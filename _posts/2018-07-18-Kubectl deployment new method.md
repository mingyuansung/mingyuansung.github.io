---
layout: post
title: Kubectl in Deployment
---

# 2018 Kubectl EDEN and ECHO Deployment


## Preparation

* Install Docker for Mac `brew install kubernetes-cli`
* Get from Kevin your AWS credentials in `~/.aws/credentials`
* Get from [our repository](https://github.com/Source-Intelligence/k8s-prep/blob/master/kube-config.yaml) our `~/.kube/config`
* each project now has a `manifest.yaml` file, which is similar to the old `docker-compose.prod.yml`.  when you wan to deploy to prod, change the project version in `manifest.yaml` like you used to in `docker-compose.prod.yml` (but this time there's no A/B groups to worry about).
* instead of `fly deploy:prod` and going through the prompts, you just `kubectl --context prod1 apply -f manifest.yaml` and k8s does the rest.there's a number of ways to observe the deployment in action:
    * `kubectl --context prod1 get -f manifest.yaml -o wide -w` to watch changes to the Deployment
    * `kubectl --context prod1 get pods -l app=<name> -w` to watch changes to Pods with an `app` label matching `<name>`, eg: `quantum`, `eden`, etc
    * `kubectl --context prod1 rollout status -f manifest.yaml` to watch changes to the ReplicaSets
    * Watch logs in Kibana and/or DataDog.  You can also view logs right in your terminal with `kubectl logs` but afaik you can't tail more than one container, which isn't very helpful during a deployment, so its probably better to stick to Kibana and DataDog.

## Dev DB Access

* Make dev mysql available at localhost:3307 and dev mongo at localhost:27018
* Then use you db app point to 3307 to connect MySQL, 27018 to Mongodb
* `kubectl -n db port-forward deployment/mysql 3307:3306`
* `kubectl -n db port-forward deployment/mongodb 27018:27017`


## EDEN DEV Deployment

* `kubectl config use-context dev2` to set your default context
* `kubectl config current-context` to check your current default context
* `kubectl --context dev2 get deployment eden -o wide` to check current version of eden on dev2 which is your current default context
* `kubectl --context dev4 get deployment eden -o wide` to check the dev4 current eden revision ignoring your current default context setting which is dev2
* `kubectl --context dev2 set image deployment/eden eden=docker.sourceintelligence.net/eden:1.2.3` to deploy eden verion 1.2.3 to dev2
* `kubectl --context dev2 get pods -l app=eden -w` to watch changes to eden Pod on dev2
* `kubectl --context dev2 logs -f deployment/eden` view eden log
* `kubectl --context dev2 get pods -l app=eden` to get the container ID, then `kubectl --context dev2 logs eden-66795f589b-5rmcm` to view that eden container log

## EDEN PROD Deployment
* Go to your local eden directory
* Make sure you update CHANGELOG.md and run `composer revision X.XX.X` following the official deployment procedure
* update `manifest.yaml` for the revision
* instead of `fly deploy:prod` and going through the prompts, you just `kubectl --context prod1 apply -f manifest.yaml` and k8s does the rest.there's a number of ways to observe the deployment in action:
    * `kubectl --context prod1 get -f manifest.yaml -o wide -w` to watch changes to the Deployment
    * `kubectl --context prod1 get pods -l app=eden -w` to watch changes to Pods with an `app` label matching `eden`
    * `kubectl --context prod1 rollout status -f manifest.yaml` to watch changes to the ReplicaSets
    * Watch logs in Kibana and/or DataDog.  You can also view logs right in your terminal with `kubectl logs` but afaik you can't tail more than one container, which isn't very helpful during a deployment, so its probably better to stick to Kibana and DataDog.

## ECHO Deployment

* the same procedure as EDEN deployment
* probably `kubectl --context prod1 apply -f manifest.yaml && kubectl --context prod1 get -f manifest.yaml -o wide -w`

## Test
* remove all images you do `docker rmi -f $(docker images -a -q)`


## Misc
* `kubectl --context dev1 get pods -l app=echo -w` to see echo pod information on dev1
* it will show for example:

```
NAME                    READY     STATUS    RESTARTS   AGE
echo-56ccb87968-dpxzx   1/1       Running   0          1d

```
* then you can do `kubectl -n dev1 exec -it echo-56ccb87968-dpxzx bash` to put yourself in the echo container on dev1
* you can also do `kubectl --context prod1 get pods` to get a listing of all pods on Prod1
