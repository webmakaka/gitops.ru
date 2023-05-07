---
layout: page
title: Переименование узлов Kubernetes Cluster
description: Переименование узлов Kubernetes Cluster
keywords: devops, linux, kubernetes, Переименование узлов Kubernetes Cluster
permalink: /devops/containers/kubernetes/kubeadm/renaming-kubernetes-nodes/
---

# Переименование узлов Kubernetes Cluster

<br/>

Делаю: 09.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=TqoA9HwFLVU&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=18

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

### Renaming Kubernetes Nodes

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   92m   v1.14.1
    node1.k8s    Ready    <none>   90m   v1.14.1
    node2.k8s    Ready    <none>   87m   v1.14.1

<br/>

    $ kubectl run nginx --image nginx --replicas=2

<br/>

    $ kubectl get all -o wide
    NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE        NOMINATED NODE   READINESS GATES
    pod/nginx-7db9fccd9b-8thfw   1/1     Running   0          28s   10.244.2.4   node2.k8s   <none>           <none>
    pod/nginx-7db9fccd9b-jvhsp   1/1     Running   0          28s   10.244.1.5   node1.k8s   <none>           <none>

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   93m   <none>

    NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
    deployment.apps/nginx   2/2     2            2           28s   nginx        nginx    run=nginx

    NAME                               DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES   SELECTOR
    replicaset.apps/nginx-7db9fccd9b   2         2         2       28s   nginx        nginx    pod-template-hash=7db9fccd9b,run=nginx

<br/>

    // Пароль: kubeadmin
    $ ssh root@node1.k8s

    # hostnamectl set-hostname kubeworker1.example.com

    # reboot

<br/>

    $ ssh root@node1.k8s

    # vi /etc/hosts
    192.168.0.11 kubeworker1.example.com kubeworker1

<br/>

    $ kubectl delete node node1.k8s

<br/>

    [root@kubeworker1 ~]# kubeadm reset
    [Y]

<br/>

    $ ssh root@master.k8s

    # kubeadm token create --print-join-command
    kubeadm join 192.168.0.10:6443 --token gl09a3.jnrvo3z6kprgii3p     --discovery-token-ca-cert-hash sha256:c25162d5fdc412f95e4180552f18e305c006f22e281c6318098ef8b480543abf

<br/>

Выполняю ее на [root@kubeworker1 ~]

<br/>

    $ kubectl get nodes
    NAME                      STATUS   ROLES    AGE    VERSION
    kubeworker1.example.com   Ready    <none>   96s    v1.14.1
    master.k8s                Ready    master   113m   v1.14.1
    node2.k8s                 Ready    <none>   108m   v1.14.1

<br/>

    $ kubectl run nginx --image nginx --replicas=2

    $ kubectl scale deploy nginx --replicas=4

<br/>

    $ kubectl get pods -o wide
    NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE                      NOMINATED NODE   READINESS GATES
    nginx-7db9fccd9b-8thfw   1/1     Running   0          24m   10.244.2.4   node2.k8s                 <none>           <none>
    nginx-7db9fccd9b-d4nxg   1/1     Running   0          15m   10.244.2.5   node2.k8s                 <none>           <none>
    nginx-7db9fccd9b-r8m2p   1/1     Running   0          9s    10.244.3.3   kubeworker1.example.com   <none>           <none>
    nginx-7db9fccd9b-zgtvf   1/1     Running   0          9s    10.244.3.2   kubeworker1.example.com   <none>           <none>

<br/>

### Еще один способ. Вроде хуже

    // Ошибка
    $ kubectl drain node2.k8s

    $ kubectl drain node2.k8s --ignore-daemonsets

<br/>

    $ ssh root@node2.k8s
    # kubeadm reset
    [Y]

<br/>

    # vi /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

    Добавил в конец после EXTRA_ARGS параметр --hostname-override=kubeworker2.example.com

<br/>

    # systemctl daemon-reload

<br/>

    # kubeadm reset
    [Y]

<br/>

    // Ранее показано как получить
    # kubeadm token create --print-join-command
    kubeadm join 192.168.0.10:6443 --token gl09a3.jnrvo3z6kprgii3p     --discovery-token-ca-cert-hash sha256:c25162d5fdc412f95e4180552f18e305c006f22e281c6318098ef8b480543abf

<br/>

    $ kubectl get nodes
    NAME                      STATUS                        ROLES    AGE    VERSION
    kubeworker1.example.com   Ready                         <none>   23m    v1.14.1
    kubeworker2.example.com   Ready                         <none>   48s    v1.14.1
    master.k8s                Ready                         master   134m   v1.14.1
    node2.k8s                 NotReady,SchedulingDisabled   <none>   130m   v1.14.1

<br/>

    $ kubectl delete node node2.k8s
