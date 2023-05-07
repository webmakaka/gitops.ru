---
layout: page
title: ConfigMaps in Kubernetes
description: ConfigMaps in Kubernetes
keywords: devops, linux, kubernetes, ConfigMaps in Kubernetes
permalink: /devops/containers/kubernetes/basics/config-maps/
---

# ConfigMaps in Kubernetes

<br/>

Делаю:  
08.04.2019

<br/>

**По материалам из видео индуса:**

https://www.youtube.com/watch?v=upmLONFGNBs&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=16

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

### ConfigMaps

```
$ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/6-configmap-1.yaml
```

<br/>

```
$ kubectl get configmaps
NAME             DATA   AGE
demo-configmap   2      16s
```

<br/>

```
$ kubectl describe cm demo-configmap
Name:         demo-configmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
channel.name:
----
justmeandopensource
channel.owner:
----
Venkat Nagappan
Events:  <none>
```

<br/>

    $ kubectl get cm demo-configmap -o yaml

<br/>

### Создать ConfigMaps в командной строке

    $ kubectl create configmap demo-configmap-1 --from-literal=channel.name=justmeandopensource --from-literal=channel.owner="Venkat Nagappan"

    $ kubectl get cm demo-configmap-1
    NAME               DATA   AGE
    demo-configmap-1   2      19s

<br/>

### Еще пример

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/6-pod-configmap-env.yaml

    $ kubectl get pods
    NAME      READY   STATUS    RESTARTS   AGE
    busybox   1/1     Running   0          8s

    $ kubectl exec -it busybox sh

    / # echo $CHANNELNAME
    justmeandopensource

    / # env | grep -i channel
    CHANNELOWNER=Venkat Nagappan
    CHANNELNAME=justmeandopensource

    ctrl^D

    $ kubectl delete pod busybox

<br/>

### Еще пример

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/6-pod-configmap-volume.yaml

    $ kubectl exec -it busybox sh

    / # ls /mydata/
    channel.name   channel.owner

    / # cat /mydata/channel.name; echo
    justmeandopensource

    / # cat /mydata/channel.owner; echo
    Venkat Nagappan

    ctrl^D

    $ kubectl delete cm demo-configmap-1

<br/>

    $ kubectl edit cm demo-configmap

    меняем channel.name: на I love kubernetes


    $ kubectl exec -it busybox sh

    / # cat /mydata/channel.name; echo
    I love kubernetes

    ctrl^D

    $ kubectl delete pod busybox

<br/>

### Еще пример

    $ rm -rf ~/tmp/k8s/config-maps && mkdir -p ~/tmp/k8s/config-maps && cd ~/tmp/k8s/config-maps

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/misc/my.cnf

    $ kubectl create configmap mysql-demo-config --from-file="my.cnf"

    $ kubectl get cm
    NAME                DATA   AGE
    demo-configmap      2      27m
    mysql-demo-config   1      49s

    $ kubectl get cm mysql-demo-config -o yaml

    $ kubectl delete cm mysql-demo-config

<br/>

### Еще пример

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/6-configmap-2.yaml

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/6-pod-configmap-mysql-volume.yaml

<br/>

    $ kubectl exec -it busybox sh

    / # cat /mydata/my.cnf
    [mysqld]
    pid-file        = /var/run/mysqld/mysqld.pid
    socket          = /var/run/mysqld/mysqld.sock
    port            = 3306
    datadir         = /var/lib/mysql
    default-storage-engine = InnoDB
    character-set-server = utf8
    bind-address            = 127.0.0.1
    general_log_file        = /var/log/mysql/mysql.log
    log_error = /var/log/mysql/error.log
