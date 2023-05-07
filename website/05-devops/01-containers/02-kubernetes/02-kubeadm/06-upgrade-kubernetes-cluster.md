---
layout: page
title: Upgrade Kubernetes Cluster
description: Upgrade Kubernetes Cluster
keywords: devops, linux, kubernetes, Upgrade Kubernetes Cluster
permalink: /devops/containers/kubernetes/kubeadm/upgrade-kubernetes-cluster/
---

# Upgrade Kubernetes Cluster

<br/>

Делаю: 11.04.2019

<br/>
**Не для production**
<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=-MZ-l2HG368&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=23

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>. **Для версии 1.11.6**

<br/>

Оф. док:
https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/

<br/>

### Информация

    $ kubectl version --short
    Client Version: v1.14.1
    Server Version: v1.11.9


    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   8m    v1.11.6
    node1.k8s    Ready    <none>   6m    v1.11.6
    node2.k8s    Ready    <none>   4m    v1.11.6


    $ kubectl get nodes -o wide
    docker://1.13.1

<br/>

###

    $ kubectl run nginx --image nginx
    $ kubectl scale deploy nginx --replicas=2

<br/>

### Поехали обновляться до 1.12

    $ kubectl get pods -o wide
    NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE        NOMINATED NODE
    nginx-64f497f8fd-d9thn   1/1     Running   0          32s   10.244.1.2   node1.k8s   <none>
    nginx-64f497f8fd-hd789   1/1     Running   0          10m   10.244.2.2   node2.k8s   <none>

<br/>

### Обновляем мастер

<br/>

на хосте:

    $ kubectl drain master.k8s --ignore-daemonsets

    $ kubectl get nodes
    NAME         STATUS                     ROLES    AGE   VERSION
    master.k8s   Ready,SchedulingDisabled   master   38m   v1.12.0
    node1.k8s    Ready                      <none>   36m   v1.11.6
    node2.k8s    Ready                      <none>   34m   v1.11.6

<br/>

    // Пароль: kubeadmin
    $ ssh root@master

    # yum upgrade -y kubeadm-1.12.0 kubelet-1.12.0

<br/>

    # kubeadm upgrade plan
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [upgrade] Fetching available versions to upgrade to
    [upgrade/versions] Cluster version: v1.11.9
    [upgrade/versions] kubeadm version: v1.12.0
    [upgrade/versions] Latest stable version: v1.14.1
    [upgrade/versions] Latest version in the v1.11 series: v1.11.9
    [upgrade/versions] WARNING: No recommended etcd for requested kubernetes version (v1.14.1)

    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     3 x v1.11.6   v1.14.1

    Upgrade to the latest stable version:

    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.11.9   v1.14.1
    Controller Manager   v1.11.9   v1.14.1
    Scheduler            v1.11.9   v1.14.1
    Kube Proxy           v1.11.9   v1.14.1
    CoreDNS              1.1.3     1.2.2
    Etcd                 3.2.18    N/A

    You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.14.1

    Note: Before you can perform this upgrade, you have to update kubeadm to v1.14.1.

<br/>

    # kubeadm upgrade apply v1.12.0
    # systemctl daemon-reload
    # systemctl restart kubelet

<br/>

На хосте

    $ kubectl uncordon master.k8s

<br/>

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   40m   v1.12.0
    node1.k8s    Ready    <none>   38m   v1.11.6
    node2.k8s    Ready    <none>   35m   v1.11.6

<br/>

### Обновляем узылы

    $ kubectl drain node1.k8s --ignore-daemonsets

    $ kubectl get nodes
    NAME         STATUS                     ROLES    AGE   VERSION
    master.k8s   Ready                      master   42m   v1.12.0
    node1.k8s    Ready,SchedulingDisabled   <none>   40m   v1.11.6
    node2.k8s    Ready                      <none>   38m   v1.11.6

<br/>

    $ ssh root@node1
    # yum upgrade -y kubeadm-1.12.0 kubelet-1.12.0
    # kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
    # systemctl daemon-reload
    # systemctl restart kubelet

<br/>

На хосте

    $ kubectl uncordon node1.k8s

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   48m   v1.12.0
    node1.k8s    Ready    <none>   46m   v1.12.0
    node2.k8s    Ready    <none>   44m   v1.11.6

<br/>

Повторяем для node2.k8s

<br/>

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   50m   v1.12.0
    node1.k8s    Ready    <none>   48m   v1.12.0
    node2.k8s    Ready    <none>   46m   v1.12.0

<br/>

## Обновление до версии 1.3

**Докер у нас старый. По хорошему нужно обновить и его!**

### Обновляем master

    $ kubectl drain master.k8s --ignore-daemonsets

<br/>

    $ ssh root@master

<br/>

    # yum upgrade -y kubeadm-1.13.* kubelet-1.13.*

<br/>

    # kubeadm upgrade plan
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [upgrade] Fetching available versions to upgrade to
    [upgrade/versions] Cluster version: v1.12.0
    [upgrade/versions] kubeadm version: v1.13.5
    I0411 12:52:05.577434   21682 version.go:237] remote version is much newer: v1.14.1; falling back to: stable-1.13
    [upgrade/versions] Latest stable version: v1.13.5
    [upgrade/versions] Latest version in the v1.12 series: v1.12.7

    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     2 x v1.12.0   v1.12.7
                1 x v1.14.1   v1.12.7

    Upgrade to the latest version in the v1.12 series:

    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.12.0   v1.12.7
    Controller Manager   v1.12.0   v1.12.7
    Scheduler            v1.12.0   v1.12.7
    Kube Proxy           v1.12.0   v1.12.7
    CoreDNS              1.2.2     1.2.6
    Etcd                 3.2.24    3.2.24

    You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.12.7

    _____________________________________________________________________

    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     2 x v1.12.0   v1.13.5
                1 x v1.14.1   v1.13.5

    Upgrade to the latest stable version:

    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.12.0   v1.13.5
    Controller Manager   v1.12.0   v1.13.5
    Scheduler            v1.12.0   v1.13.5
    Kube Proxy           v1.12.0   v1.13.5
    CoreDNS              1.2.2     1.2.6
    Etcd                 3.2.24    3.2.24

    You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.13.5

    _____________________________________________________________________

<br/>

    # kubeadm upgrade apply v1.13.5 -y

<br/>

    # systemctl daemon-reload
    # systemctl restart kubelet

<br/>

На хосте

    $ kubectl uncordon master.k8s

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   79m   v1.13.5
    node1.k8s    Ready    <none>   76m   v1.12.0
    node2.k8s    Ready    <none>   74m   v1.12.0

<br/>

### Обновляем узылы

    $ kubectl drain node1.k8s --ignore-daemonsets

    $ kubectl get nodes
    NAME         STATUS                     ROLES    AGE   VERSION
    master.k8s   Ready                      master   42m   v1.12.0
    node1.k8s    Ready,SchedulingDisabled   <none>   40m   v1.11.6
    node2.k8s    Ready                      <none>   38m   v1.11.6

<br/>

    $ ssh root@node1
    # yum upgrade -y kubeadm-1.13.* kubelet-1.13.*
    # kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
    # systemctl daemon-reload
    # systemctl restart kubelet

<br/>

На хосте

    $ kubectl uncordon node1.k8s

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   80m   v1.13.5
    node1.k8s    Ready    <none>   78m   v1.13.5
    node2.k8s    Ready    <none>   76m   v1.12.0

<br/>

Повторяем для node2.k8s

<br/>

    $ kubectl get nodes
    NAME STATUS ROLES AGE VERSION
    master.k8s Ready master 82m v1.13.5
    node1.k8s Ready <none> 80m v1.13.5
    node2.k8s Ready <none> 78m v1.13.5

<br/>

## Обновление до версии 1.14.1

<br/>

### Обновляем мастер

    $ kubectl drain master.k8s --ignore-daemonsets

<br/>

    $ ssh root@master

    # yum upgrade -y kubeadm kubelet
    # kubeadm version
    v1.14.1

    # kubelet --version
    Kubernetes v1.14.1


    # kubeadm upgrade plan
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [upgrade] Fetching available versions to upgrade to
    [upgrade/versions] Cluster version: v1.13.5
    [upgrade/versions] kubeadm version: v1.14.1
    [upgrade/versions] Latest stable version: v1.14.1
    [upgrade/versions] Latest version in the v1.13 series: v1.13.5

    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     3 x v1.13.5   v1.14.1

    Upgrade to the latest stable version:

    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.13.5   v1.14.1
    Controller Manager   v1.13.5   v1.14.1
    Scheduler            v1.13.5   v1.14.1
    Kube Proxy           v1.13.5   v1.14.1
    CoreDNS              1.2.6     1.3.1
    Etcd                 3.2.24    3.3.10

    You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.14.1

    _____________________________________________________________________

<br/>

    # kubeadm upgrade apply v1.14.1 -y

<br/>

    # systemctl daemon-reload
    # systemctl restart kubelet

<br/>

На хосте

    $ kubectl uncordon master.k8s

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   90m   v1.14.1
    node1.k8s    Ready    <none>   87m   v1.13.5
    node2.k8s    Ready    <none>   85m   v1.13.5

<br/>

### Обновляем узылы

    $ kubectl drain node1.k8s --ignore-daemonsets

    $ kubectl get nodes
    NAME         STATUS                     ROLES    AGE   VERSION
    master.k8s   Ready                      master   42m   v1.12.0
    node1.k8s    Ready,SchedulingDisabled   <none>   40m   v1.11.6
    node2.k8s    Ready                      <none>   38m   v1.11.6

<br/>

    $ ssh root@node1
    # yum upgrade -y kubeadm kubelet
    # kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
    # systemctl daemon-reload
    # systemctl restart kubelet

<br/>

На хосте

    $ kubectl uncordon node1.k8s

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   91m   v1.14.1
    node1.k8s    Ready    <none>   89m   v1.14.1
    node2.k8s    Ready    <none>   87m   v1.13.5

<br/>

Повторяем для node2.k8s

<br/>

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE   VERSION
    master.k8s   Ready    master   93m   v1.14.1
    node1.k8s    Ready    <none>   91m   v1.14.1
    node2.k8s    Ready    <none>   89m   v1.14.1

<br/>

    $ kubectl get pods
    NAME                     READY   STATUS    RESTARTS   AGE
    nginx-64f497f8fd-kvjww   1/1     Running   0          7m45s
    nginx-64f497f8fd-vhv7p   1/1     Running   0          7m45s
