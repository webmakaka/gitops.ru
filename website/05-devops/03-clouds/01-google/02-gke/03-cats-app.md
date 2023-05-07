---
layout: page
title: Запуск приложения с котиками в GKE (Load Balancer)
description: Запуск приложения с котиками в GKE (Load Balancer)
keywords: Запуск приложения с котиками в GKE (Load Balancer)
permalink: /devops/clouds/google/gke/google/cats-app/
---

# Запуск приложения с котиками в GKE (Load Balancer)

Делаю:  
21.05.2019

<br/>

    $ gcloud container clusters create cats-cluster \
        --num-nodes 2 \
        --machine-type n1-standard-1 \
        --zone us-central1-a

<br/>

    $ kubectl apply -f https://raw.githubusercontent.com/webmakaka/cats-app/master/minikube/minikube-cats-app-deployment.yaml

<br/>

    $ kubectl expose deployment minikube-cats-app-deployment --type="LoadBalancer"

<br/>

    $ kubectl get services

<br/>

http://35.226.108.248:8080/
