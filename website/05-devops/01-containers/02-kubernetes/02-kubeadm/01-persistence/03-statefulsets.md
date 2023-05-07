---
layout: page
title: Kubernetes Statefulsets
description: Kubernetes Statefulsets
keywords: devops, linux, kubernetes, Kubernetes Statefulsets
permalink: /devops/containers/kubernetes/kubeadm/persistence/statefulsets/
---

# Kubernetes Statefulsets

Делаю: 03.04.2019

<br/>

По материалам из видео индуса.

https://www.youtube.com/watch?v=r_ZEpPTCcPE&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=22

<br/>

![kubernetes Statefulsets](/img/devops/containers/kubernetes/kubeadm/persistence/Statefulsets.png 'kubernetes Statefulsets'){: .center-image }

<br/>

Тоже самое, что и в <a href="/devops/containers/kubernetes/kubeadm/persistence/nfs/">NFS</a>, только

    $ sudo mkdir -p /srv/nfs/kubedata/{pv0,pv1,pv2,pv3,pv4}

    $ sudo exportfs -rav

<br/>

    $ mkdir ~/tmp && cd ~/tmp/

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/9-sts-pv.yaml

<br/>

    $ vi 9-sts-pv.yaml

    server: 192.168.0.6

<br/>

    $ kubectl create -f 9-sts-pv.yaml

    $ kubectl get pv
    NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
    pv-nfs-pv0   200Mi      RWO            Retain           Available           manual                  39s
    pv-nfs-pv1   200Mi      RWO            Retain           Available           manual                  39s
    pv-nfs-pv2   200Mi      RWO            Retain           Available           manual                  39s
    pv-nfs-pv3   200Mi      RWO            Retain           Available           manual                  39s
    pv-nfs-pv4   200Mi      RWO            Retain           Available           manual                  39s

<br/>

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/9-sts-nginx.yaml

<br/>

    $ vi 9-sts-nginx.yaml

    replicas: 4

    $ kubectl create -f 9-sts-nginx.yaml


    $ kubectl get pv,pvc
    NAME                          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                     STORAGECLASS   REASON   AGE
    persistentvolume/pv-nfs-pv0   200Mi      RWO            Retain           Bound       default/www-nginx-sts-0   manual                  10m
    persistentvolume/pv-nfs-pv1   200Mi      RWO            Retain           Available                             manual                  10m
    persistentvolume/pv-nfs-pv2   200Mi      RWO            Retain           Available                             manual                  10m
    persistentvolume/pv-nfs-pv3   200Mi      RWO            Retain           Available                             manual                  10m
    persistentvolume/pv-nfs-pv4   200Mi      RWO            Retain           Available                             manual                  10m

    NAME                                    STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/www-nginx-sts-0   Bound    pv-nfs-pv0   200Mi      RWO            manual         49s

<br/>

    $ kubectl get pods
    NAME          READY   STATUS    RESTARTS   AGE
    nginx-sts-0   1/1     Running   0          13m
    nginx-sts-1   1/1     Running   0          51s
    nginx-sts-2   1/1     Running   0          33s
    nginx-sts-3   1/1     Running   0          27s

<br/>

    $ kubectl exec -it nginx-sts-2 -- /bin/sh

    $ cd /var/www
    $ touch hello

    $ ctrl^D

<br/>

    $ kubectl delete pod nginx-sts-2

<br/>

    $ kubectl exec -it nginx-sts-2 -- /bin/sh

    $ ls /var/www

<br/>

    $ kubectl scale sts nginx-sts --replicas=0
    $ kubectl delete sts nginx-sts
    $ kubectl delete svc nginx-headless
    $ kubectl delete pvc --all
    $ kubectl delete pv --all

<br/>

    $ vi 9-sts-nginx.yaml

    // разкомментировать
    #podManagementPolicy: Parallel

<br/>

    Повторить
