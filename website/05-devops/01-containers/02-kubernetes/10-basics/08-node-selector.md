---
layout: page
title: Node Selector in Kubernetes
description: Node Selector in Kubernetes
keywords: devops, linux, kubernetes, Node Selector in Kubernetes
permalink: /devops/containers/kubernetes/basics/node-selector/
---

# Node Selector in Kubernetes

<br/>

Делаю: 07.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=TFAASAfO_gg&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=10

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

    $ kubectl label node node2.k8s demoserver=true
    $ kubectl get node node2.k8s --show-labels
    NAME        STATUS   ROLES    AGE   VERSION   LABELS
    node2.k8s   Ready    <none>   53m   v1.14.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,demoserver=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=node2.k8s,kubernetes.io/os=linux

<br/>

    $ mkdir ~/tmp/node-selector && cd ~/tmp/node-selector

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/1-nginx-deployment.yaml

<br/>

    $ vi 1-nginx-deployment.yaml

<br/>

Дописываем nodeSelector  
И replicas: 1

```

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
      nodeSelector:
        demoserver: "true"

```

    $ kubectl create -f 1-nginx-deployment.yaml
    $ kubectl scale deploy nginx-deploy --replicas=2

<br/>

    $ kubectl get pods -o wide
    NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE        NOMINATED NODE   READINESS GATES
    nginx-deploy-564845db65-kqbw4   1/1     Running   0          17s   10.244.2.3   node2.k8s   <none>           <none>
    nginx-deploy-564845db65-l9jqc   1/1     Running   0          85s   10.244.2.2   node2.k8s   <none>           <none>

<br/>

### Удаляем все это добро

    $ kubectl delete -f 1-nginx-deployment.yaml
    deployment.extensions "nginx-deploy" deleted
