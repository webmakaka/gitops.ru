---
layout: page
title: Запуск Jenkins в kuberntes с помощью heml
description: Запуск Jenkins в kuberntes с помощью heml
keywords: devops, linux, kubernetes, Запуск Jenkins в kuberntes с помощью heml
permalink: /devops/containers/kubernetes/packages/heml/jenkins/
---

# Запуск Jenkins в kuberntes с помощью heml

<br/>

Делаю: 06.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=ObGR0EfVPlg&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=26

<br/>

Предыдущее видео, в котором он рассказывает о helm обзорное и все повторяется в видео, ссылка на которое выше.

<br/>

### Running Jenkins in Kubernetes Cluster using Helm

<br/>

![kubernetes Helm Jenkins](/img/devops/containers/kubernetes/kubeadm/helm/helm-jenkins1.png 'kubernetes Helm Jenkins'){: .center-image }

<br/>

-   Подготовили кластер и окружение
-   Подняли Dynamic NFS
-   Инсталлировали helm

<br/>

UPD. Heml2 выпилен, предлагаю попробовать Helm3 как <a href="/devops/containers/kubernetes/packages/heml/setup/">здесь</a>.

<br/>

### Устанавливаем Jenkins

    $ helm search jenkins

    $ helm inspect values stable/jenkins > /tmp/jenkins.values

<br/>

    $ kubectl get storageclass
    NAME                  PROVISIONER       AGE
    managed-nfs-storage   example.com/nfs   42m

<br/>

    $ vi /tmp/jenkins.values

<br/>

    Значения которые менять не нужно - удалить.
    Те которые нужно переопределить оставить в файле.

<br/>

    После удаления, все, что осталось:

<br/>

```

Master:
  AdminUser: admin
  AdminPassword: admin
  ServiceType: NodePort
  NodePort: 32323

Persistence:
  StorageClass: "managed-nfs-storage"

rbac:
  install: true

```

<br/>

    $ helm install stable/jenkins --values /tmp/jenkins.values --name myjenkins

<br/>

    $ kubectl get pods
    NAME                                      READY   STATUS    RESTARTS   AGE
    myjenkins-6f995f57d6-2hjhg                1/1     Running   0          3m43s
    nfs-client-provisioner-67cd85d66d-9b5l8   1/1     Running   0          55m

<br/>

    $ kubectl get pvc
    NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
    myjenkins   Bound    pvc-81c13cb5-5862-11e9-8748-525400261060   8Gi        RWO            managed-nfs-storage   4m26s

<br/>

    $ helm status myjenkins | less

<br/>

http://node1.k8s:32323/

<br/>

![kubernetes Helm Jenkins](/img/devops/containers/kubernetes/kubeadm/helm/helm-jenkins2.png 'kubernetes Helm Jenkins'){: .center-image }

<br/>

Credentials --> Jenkins --> Global credentials --> Add Credentials

Kind --> Kubernetes Service Account

<br/>

![kubernetes Helm Jenkins](/img/devops/containers/kubernetes/kubeadm/helm/helm-jenkins3.png 'kubernetes Helm Jenkins'){: .center-image }

<br/>

Manage Jenkins --> Configure System

<br/>

![kubernetes Helm Jenkins](/img/devops/containers/kubernetes/kubeadm/helm/helm-jenkins4.png 'kubernetes Helm Jenkins'){: .center-image }

<br/>

New Item --> "demo-job"--> Freestyle Project

Build --> Execute shell

![kubernetes Helm Jenkins](/img/devops/containers/kubernetes/kubeadm/helm/helm-jenkins5.png 'kubernetes Helm Jenkins'){: .center-image }

```
echo Hi this is from inside the container
hostname
sleep 20
```

Build now

<br/>

![kubernetes Helm Jenkins](/img/devops/containers/kubernetes/kubeadm/helm/helm-jenkins6.png 'kubernetes Helm Jenkins'){: .center-image }

<br/>

### Удаление jenkins из kubernetes

    $ helm list
    $ helm delete myjenkins --purge
