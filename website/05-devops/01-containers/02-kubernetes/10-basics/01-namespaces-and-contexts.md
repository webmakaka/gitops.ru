---
layout: page
title: Kubernetes Namespaces & Contexts
description: Kubernetes Namespaces & Contexts
keywords: devops, linux, kubernetes, Namespaces & Contexts
permalink: /devops/containers/kubernetes/basics/namespaces-and-contexts/
---

# Kubernetes Namespaces & Contexts

<br/>

Делаю: 06.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=2h6TAJirDqI&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=9

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

    $ kubectl get ns
    NAME              STATUS   AGE
    default           Active   3h12m
    kube-node-lease   Active   3h12m
    kube-public       Active   3h12m
    kube-system       Active   3h12m

<br/>

    $ kubectl --namespace kube-system get pods
    NAME                                 READY   STATUS    RESTARTS   AGE
    coredns-fb8b8dccf-rv6kf              1/1     Running   0          3h14m
    coredns-fb8b8dccf-x72r6              1/1     Running   0          3h14m
    etcd-master.k8s                      1/1     Running   0          3h13m
    kube-apiserver-master.k8s            1/1     Running   0          3h13m
    kube-controller-manager-master.k8s   1/1     Running   0          3h13m
    kube-flannel-ds-amd64-d2bv7          1/1     Running   0          3h14m
    kube-flannel-ds-amd64-dxmlp          1/1     Running   0          3h12m
    kube-flannel-ds-amd64-qzr5m          1/1     Running   2          3h10m
    kube-proxy-9n587                     1/1     Running   0          3h14m
    kube-proxy-mxfxj                     1/1     Running   0          3h10m
    kube-proxy-xhprk                     1/1     Running   0          3h12m
    kube-scheduler-master.k8s            1/1     Running   0          3h13m
    tiller-deploy-8458f6c667-j5ld8       1/1     Running   0          165m

<br/>

    $ kubectl create namespace demo

    $ kubectl config set-context --current --namespace=demo

    // $ kubectl delete namespace demo

<br/>

    $ kubectl config get-contexts

    $ kubectl config set-context kubesys --namespace=kube-system --user=kubernetes-admin --cluster=kubernetes

    $ kubectl config current-context
    kubernetes-admin@kubernetes

    $ kubectl config use-context kubesys

    $ kubectl get pods

    $ kubectl config set-context demo --namespace=demo --user=kubernetes-admin --cluster=kubernetes

    $ kubectl config get-contexts

    $ kubectl config use-context demo

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/1-nginx-pod.yaml

    $ kubectl get pods
    NAME    READY   STATUS    RESTARTS   AGE
    nginx   1/1     Running   0          27s

    $ kubectl -n demo get pods
    NAME    READY   STATUS    RESTARTS   AGE
    nginx   1/1     Running   0          87s

<br/>

### Удаление

    $ kubectl delete pod nginx
