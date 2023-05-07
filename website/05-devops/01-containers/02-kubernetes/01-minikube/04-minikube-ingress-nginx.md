---
layout: page
title: Пример Ingress в minikube (Nginx)
description: Пример Ingress в minikube (Nginx)
keywords: devops, linux, kubernetes,  Пример Ingress в minikube (Nginx)
permalink: /devops/containers/kubernetes/kubeadm/minikube-ingress-nginx/
---

# Пример Ingress в minikube (Nginx)

Делаю: 24.04.2019

<br/>

### KubernetesInc Ingress Nginx

    $ minikube start

    $ minikube addons enable ingress

    $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

    $ kubectl apply -f https://raw.githubusercontent.com/webmakaka/cats-app/master/minikube-cats-app-deployment.yaml

    $ kubectl apply -f https://raw.githubusercontent.com/webmakaka/cats-app/master/minikube-cats-app-cluster-ip-service.yaml

    $ kubectl apply -f https://raw.githubusercontent.com/webmakaka/cats-app/master/minikube-cats-app-ingress-service.yaml

<br/>

    $ kubectl get ing
    NAME              HOSTS   ADDRESS   PORTS   AGE
    ingress-service   *                 80      24s

<br/>

    $ minikube ip
    192.168.99.119

<br/>

    https://192.168.99.119/

<br/>

Если все ОК. (У меня да).  
Должны появиться котики.

<br/>

### Еще 1 пример

Делаю: 17.04.2019

<br/>

Этот вариант будет работать (предположительно) только на minikube.

<br/>

По материалам статьи

https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

<br/>

    $ minikube start
    $ minikube addons enable ingress

    # kubectl apply -f https://gist.githubusercontent.com/matthewpalmer/d1147ed1066e9219e8c346f4e0dd0488/raw/05b8917f31bd76d9d28fc249d0dfc71523787462/apple.yaml

    # kubectl apply -f https://gist.githubusercontent.com/matthewpalmer/d70ca836c7d7c5da37660923915d9526/raw/242a8261a6acfbc377926d66ffa6e1f995fd251d/banana.yaml

    # kubectl apply -f https://gist.githubusercontent.com/matthewpalmer/9721cf9b3b719bd8ae3af00648cbb484/raw/d703d7330f4bba33c2588230f505e802275e2af9/ingress.yaml

    # kubectl get ing
    NAME              HOSTS   ADDRESS   PORTS   AGE
    example-ingress   *                 80      2m5s

    $ minikube ip
    192.168.99.121

    $ curl -kL http://192.168.99.121/apple
    apple

    $ curl -kL http://192.168.99.121/banana
    banana

<br/>

### Удаляем, если не нужно

    $ minikube stop
    $ minikube delete
