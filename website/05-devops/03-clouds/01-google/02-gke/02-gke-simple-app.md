---
layout: page
title: Запуск простейшего приложения в GKE
description: Запуск простейшего приложения в GKE
keywords: Запуск простейшего приложения в GKE
permalink: /devops/clouds/google/gke/gke-simple-app/
---

# Запуск простейшего приложения в GKE

Делаю!  
21.05.2019

<br/>

### Немного отражает примеры из видео индуса:

https://www.youtube.com/watch?v=5Cb8YrlpuYU&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=28

<br/>

### Подготовка GKE кластера:

    $ gcloud container clusters create quick-demo --zone "europe-west1-c" --machine-type "f1-micro"

    $ gcloud container clusters get-credentials quick-demo

    $ kubectl version --short
    $ kubectl cluster-info
    $ kubectl get nodes

    $ kubectl get cs

<br/>

    $ gcloud compute instances list

<br/>

### Деплой приложения в GKE кластер

    $ gcloud container clusters list
    NAME        LOCATION        MASTER_VERSION  MASTER_IP       MACHINE_TYPE  NODE_VERSION   NUM_NODES  STATUS
    quick-demo  europe-west1-c  1.12.7-gke.10   35.205.197.108  f1-micro      1.12.7-gke.10  3          RUNNING

<br/>

    $  kubectl get nodes
    NAME                                        STATUS   ROLES    AGE     VERSION
    gke-quick-demo-default-pool-5cc938c4-8h28   Ready    <none>   2m37s   v1.12.7-gke.10
    gke-quick-demo-default-pool-5cc938c4-8q1f   Ready    <none>   2m30s   v1.12.7-gke.10
    gke-quick-demo-default-pool-5cc938c4-k3st   Ready    <none>   2m31s   v1.12.7-gke.10

<br/>

    $ kubectl run nginx-deploy --image nginx --replicas 2
    $ kubectl scale deploy nginx-deploy --replicas 4

<br/>

    $ kubectl get pods
    NAME                            READY   STATUS              RESTARTS   AGE
    nginx-deploy-74f8cd9b44-8fqxn   1/1     Running             0          30s
    nginx-deploy-74f8cd9b44-8xwvj   0/1     ContainerCreating   0          24s
    nginx-deploy-74f8cd9b44-n2fnh   1/1     Running             0          24s
    nginx-deploy-74f8cd9b44-vnwvq   1/1     Running             0          30s

<br/>

    $ kubectl expose deployment nginx-deploy --port 80 --type LoadBalancer

<br/>

    $ kubectl get services
    NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
    kubernetes     ClusterIP      10.51.240.1     <none>          443/TCP        7m12s
    nginx-deploy   LoadBalancer   10.51.254.251   35.205.65.201   80:30339/TCP   66s

<br/>

http://35.205.65.201:80

<br/>
 
    // Если нужно удалить
    $ gcloud container clusters delete quick-demo
