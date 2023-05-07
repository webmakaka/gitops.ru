---
layout: page
title: Менеджер пакетов helm (начинаем разбираться)
description: Менеджер пакетов helm (начинаем разбираться)
keywords: devops, linux, kubernetes, Менеджер пакетов helm (начинаем разбираться)
permalink: /devops/containers/kubernetes/basics/init-containers/
---

# Init Containers in Kubernetes Cluster

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=YzaYqxW0wGs&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0

<br/>

Если init контейрнер не отработал, то и остальные не запустятся.

<br/>

**сессия 1:**

    $ watch kubectl get all -o wide

<br/>

**сессия 2:**

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/3-init-container.yaml


    $ kubectl describe deploy nginx-deploy

    $ kubectl expose deployment nginx-deploy --type NodePort --port 80

    $ curl http://node1.k8s:32256
    <h1>Hello Kubernetes</h1>

    $ kubectl scale deploy nginx-deploy --replicas=2

<br/>

### Удаление всего

    $ kubectl delete scv nginx-deploy
    $ kubectl delete deploy nginx-deploy
