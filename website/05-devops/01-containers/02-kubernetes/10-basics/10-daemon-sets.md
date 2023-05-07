---
layout: page
title: Kubernetes DaemonSets
description: Kubernetes DaemonSets
keywords: devops, linux, kubernetes, Kubernetes DaemonSets
permalink: /devops/containers/kubernetes/basics/daemon-sets/
---

# Kubernetes DaemonSets

Делаю: 02.04.2019

<br/>

По материалам из видео индуса.

https://www.youtube.com/watch?v=PWBpy4IlfMQ&index=11&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0

<br/>

Смысл - показать как развернуть приложение на узлы кластера. Если установлена метка, то приложение будет развернуто на узлы, где присутствует данная метка. Если такого требования нет, то прилжение будет установлено на все узлы кластера.

<br/>

    $ kubectl label node node2.k8s gpupresent="true"

    $ kubectl get nodes -l gpupresent="true"
    NAME        STATUS   ROLES    AGE   VERSION
    node2.k8s   Ready    <none>   24m   v1.14.0

<br/>

    $ mkdir ~/tmp && cd ~/tmp/

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/1-nginx-daemonset.yaml


    $ vi 1-nginx-daemonset.yaml

Дописываем nodeSelector

```

apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-daemonset
spec:
  selector:
    matchLabels:
      demotype: nginx-daemonset-demo
  template:
    metadata:
      labels:
        demotype: nginx-daemonset-demo
    spec:
      containers:
      - image: nginx
        name: nginx
      nodeSelector:
        gpupresent: "true"

```

<br/>

    $ kubectl create -f 1-nginx-daemonset.yaml

    $ kubectl get pods -o wide

    $ kubectl get daemonsets

    $ kubectl describe daemonsets nginx-daemonset

    $ kubectl describe pod

    $ kubectl delete daemonset nginx-daemonset
