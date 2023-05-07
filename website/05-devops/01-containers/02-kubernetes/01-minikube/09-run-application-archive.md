---
layout: page
title: Запуск приложения в minikube
description: Запуск приложения в minikube
keywords: devops, linux, kubernetes,  Запуск приложения в minikube
permalink: /devops/containers/kubernetes/minikube/run-application-archive/
---

# Запуск приложения в minikube

Делаю:  
28.02.2019

<br/>

### Подготовка

    $ minikube start

<br/>

    $ minikube status
    host: Running
    kubelet: Running
    apiserver: Running
    kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100

<br/>

    $ kubectl cluster-info
    Kubernetes master is running at https://192.168.99.100:8443
    KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

<br/>

Вроде все норм. Поехали!

<br/>

### Запуск без конфигов JSON / YAML

    // Делал с ключом run-pod, были какие-то проблемы
    $ kubectl run nodejs-cats-app --image=webmakaka/cats-app --port=8080 --generator=run/v1

    kubectl run --generator=run/v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
    replicationcontroller/nodejs-cats-app created

<br/>

    $ kubectl get pods
    NAME                       READY   STATUS    RESTARTS   AGE
    nodejs-cats-app-nsk72   1/1     Running   0          23s

<br/>

### Создание объекта Service для доступа к приложению

rc - replicationcontroller

<br/>

    $ kubectl expose rc nodejs-cats-app --type=LoadBalancer --name nodejs-cats-app-load-balancer

<br/>

    // Можно не ждать External-IP. На minikube он не появится
    $ kubectl get services
    NAME                               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    kubernetes                         ClusterIP      10.96.0.1      <none>        443/TCP          104s
    nodejs-cats-app-load-balancer   LoadBalancer   10.96.51.251   <pending>     8080:30748/TCP   8s

<br/>

    $ minikube ip
    192.168.99.102

<br/>

    $ echo $(minikube service nodejs-cats-app-load-balancer --url)
    http://192.168.99.102:30748

<br/>

![Cats inside minikube](/img/devops/containers/kubernetes/nodejs-cats-app.png 'Cats inside minikube'){: .center-image }

<br/>

    $ kubectl get rc
    NAME                 DESIRED   CURRENT   READY   AGE
    nodejs-cats-app   1         1         1       4m41s

<br/>

### Изменение количества реплик

    $ kubectl scale rc nodejs-cats-app --replicas=3

<br/>

    $ kubectl get rc
    NAME                 DESIRED   CURRENT   READY   AGE
    nodejs-cats-app   3         3         3       5m6s

<br/>

    $ kubectl get pods
    NAME                       READY   STATUS    RESTARTS   AGE
    nodejs-cats-app-mtp56   1/1     Running   0          22s
    nodejs-cats-app-mvvnp   1/1     Running   0          22s
    nodejs-cats-app-nsk72   1/1     Running   0          5m21s

<br/>

    $ kubectl describe pod nodejs-cats-app-mvvnp

<br/>

### Dashboard

    $ minikube dashboard

<br/>

![minikube dashboard](/img/devops/containers/kubernetes/dashboard.png 'minikube dashboard'){: .center-image }

<br/>

### Пока котики, нам будет вас не хватать :(

<!--
    // Удалить все модули, службы и контроллер репликации. Секреты не удалятся.
    $ kubectl delete all --all

    // Удалить все созданные модули (Контроллер репликации будет поднимать модуль из-за команды run/v1)
    $ kubectl delete po --all

-->

    $ minikube stop

    // Вообще все удалить
    $ minikube delete
