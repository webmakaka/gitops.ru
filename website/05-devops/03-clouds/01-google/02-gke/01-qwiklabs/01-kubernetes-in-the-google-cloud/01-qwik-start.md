---
layout: page
title: Kubernetes Engine Qwik Start
description: Kubernetes Engine Qwik Start
keywords: Kubernetes Engine Qwik Start
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/qwik-start/
---

# [GSP100] Kubernetes Engine: Qwik Start

<br/>

Делаю!  
20.05.2019

<br/>

    $ gcloud auth list
    $ gcloud config list project

    $ gcloud config set compute/zone us-central1-a

    $ gcloud container clusters create marleycluster1

    $ gcloud container clusters get-credentials marleycluster1

    $ kubectl run hello-server --image=gcr.io/google-samples/hello-app:1.0 --port 8080

    $ kubectl expose deployment hello-server --type="LoadBalancer"

    $ kubectl get service hello-server

    $ kubectl get service hello-server
    NAME           TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)          AGE
    hello-server   LoadBalancer   10.7.254.2   35.238.204.137   8080:31703/TCP   83s

<br/>

http://35.238.204.137:8080/

<br/>

    $ gcloud container clusters delete marleycluster1
