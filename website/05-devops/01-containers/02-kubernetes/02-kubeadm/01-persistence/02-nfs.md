---
layout: page
title: NFS Persistent Volume in Kubernetes Cluster
description: NFS Persistent Volume in Kubernetes Cluster
keywords: devops, linux, kubernetes, NFS Persistent Volume in Kubernetes Cluster
permalink: /devops/containers/kubernetes/kubeadm/persistence/nfs/
---

# NFS Persistent Volume in Kubernetes Cluster

Делаю:  
24.10.2019

<br/>

    $ kubectl version --short
    Client Version: v1.16.2
    Server Version: v1.16.2

<br/>

По материалам индуса:

https://www.youtube.com/watch?v=to14wmNmRCI&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=21

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

Добавляю еще 1 виртуалку на которой будет смонтирован NFS раздел.

    $ rm -rf ~/vagrant-kubernetes-nfs-serv && mkdir ~/vagrant-kubernetes-nfs-serv && cd ~/vagrant-kubernetes-nfs-serv

<br/>

// Создаем Vagrantfile для виртуалки

```
$ cat <<EOF >> Vagrantfile

# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  config.hostmanager.enabled = true
  config.hostmanager.include_offline = true

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.vm.define "nfs-serv.k8s" do |c|
    c.vm.hostname = "nfs-serv.k8s"
    c.vm.network "private_network", ip: "192.168.0.6"
  end
end
EOF
```

<br/>

    $ vagrant up

<br/>

    $ vagrant status
    Current machine states:

    nfs-serv.k8s              running (virtualbox)

<br/>

    $ vagrant ssh nfs-serv.k8s

<br/>

### Подготавливаем NFS сервер. Настраиваем экспорт. (На nfs-serv.k8s)

    // пинг master ноды кластера
    $ ping 192.168.0.10
    ok

    $ sudo yum install -y nfs-utils

    $ sudo systemctl enable nfs-server
    $ sudo systemctl start nfs-server

    $ sudo mkdir -p /srv/nfs/kubedata

    $ sudo chmod -R 777 /srv/nfs
    $ sudo chown nobody: /srv/nfs/kubedata

<br/>

**Для демо, не для продуктового использования! Мне сейчас не до правильных настроек на тестовом сервере.**

    $ sudo vi /etc/exports

```
/srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)
```

<!--
На youtube под видео, индусы предлагают решение, чтобы не использовать insecure.

-->

    $ sudo exportfs -rav

Посмотреть результаты:

    $ sudo exportfs -v
    $ sudo showmount -e

<br/>

Подготовка закончена!

<br/>

## Изучаем всевозможные варианты использования

    $ sudo vi /srv/nfs/kubedata/index.html

    <h1>NFS Server</h1>

<br/>

### На хост машине. Проверка что nfs монтируется на узлах

    // пароль kubeadmin
    $ ssh root@node1

    # showmount -e 192.168.0.6
    Export list for 192.168.0.6:
    /srv/nfs/kubedata *

    # mount -t nfs 192.168.0.6:/srv/nfs/kubedata /mnt/

    # mount | grep kubedata

    # ls /mnt/
    index.html

    // kubernetes может сам смонтировать разделы
    # umount /mnt

<br/>

    $ mkdir ~/tmp && cd ~/tmp/

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/4-pv-nfs.yaml


    $ vi 4-pv-nfs.yaml

    server: 192.168.0.6

    $ kubectl create -f 4-pv-nfs.yaml

<br/>

    $ kubectl get pv
    NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
    pv-nfs-pv1   1Gi        RWX            Retain           Bound    default/pvc-nfs-pv1   manual                  18s

<br/>

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/4-pvc-nfs.yaml

<br/>

    $ kubectl get pv,pvc
    NAME                          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
    persistentvolume/pv-nfs-pv1   1Gi        RWX            Retain           Bound    default/pvc-nfs-pv1   manual                  34s

    NAME                                STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/pvc-nfs-pv1   Bound    pv-nfs-pv1   1Gi        RWX            manual         22s

<br/>

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/4-nfs-nginx.yaml

<br/>

    $ kubectl get pods
    NAME                            READY   STATUS    RESTARTS   AGE
    nginx-deploy-69bd64468b-w9zmv   1/1     Running   0          17s

<br/>

    $ kubectl exec -it nginx-deploy-69bd64468b-w9zmv -- /bin/sh

    $ cd  /usr/share/nginx/html
    $ cat index.html
    <h1>NFS Server</h1>

    $ exit

<br/>

    $ kubectl expose deploy nginx-deploy --port 80 --type NodePort

<br/>

    $ kubectl get svc nginx-deploy
    NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    nginx-deploy   NodePort   10.100.31.163   <none>        80:30133/TCP   103s

<br/>

    $ curl node1:30133
    <h1>NFS Server</h1>

<br/>

### Удаление созданного на кластере

    $ kubectl delete svc nginx-deploy
    $ kubectl delete deploy nginx-deploy
    $ kubectl delete pvc pvc-nfs-pv1
    $ kubectl delete pv pv-nfs-pv1
