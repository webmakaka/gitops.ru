---
layout: page
title: HostPath
description: HostPath
keywords: devops, linux, kubernetes, HostPath
permalink: /devops/containers/kubernetes/kubeadm/persistence/hostpath/
---

# HostPath

Делаю: 31.03.2019

<br/>

По материалам из видео индуса.

https://www.youtube.com/watch?v=I9GMUn15Nes&index=14&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0

<br/>

### HostPath (не для продуктового использования)

Смысл показать, что если данные хранятся на одной ноде, а потом кластер переключится на другую. То данные автоматически не перенесутся и у приложения не будет каких-либо данных.

Подготовили кластер как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

    $ mkdir ~/vagrant-kubernetes-scripts && cd ~/vagrant-kubernetes-scripts

    # git clone https://bitbucket.org/sysadm-ru/kubernetes .


    $ cd yamls/

    $ ssh root@master

    Пароль root: kubeadmin

    # mkdir /kube
    # chmod 777 /kube/


    $ kubectl create -f 4-pv-hostpath.yaml

    $ kubectl get pv
    NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
    pv-hostpath   1Gi        RWO            Retain           Available           manual                  30s


    $ kubectl create -f 4-pvc-hostpath.yaml

    $ kubectl get pvc
    NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    pvc-hostpath   Bound    pv-hostpath   1Gi        RWO            manual         30s


    $ kubectl create -f 4-busybox-pv-hostpath.yaml

    $ kubectl describe pod busybox

    $ kubectl exec busybox touch /mydata/hello

    $ kubectl delete pod busybox


    $ kubectl get pv,pvc
    NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
    persistentvolume/pv-hostpath   1Gi        RWO            Retain           Bound    default/pvc-hostpath   manual                  12m

    NAME                                 STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/pvc-hostpath   Bound    pv-hostpath   1Gi        RWO            manual         9m47s


    $ kubectl delete pvc pvc-hostpath

    # ls /kube/

    $ kubectl create -f 4-pvc-hostpath.yaml

    $ kubectl delete pvc pvc-hostpath

    $ kubectl delete pv pv-hostpath
    persistentvolume "pv-hostpath" deleted
