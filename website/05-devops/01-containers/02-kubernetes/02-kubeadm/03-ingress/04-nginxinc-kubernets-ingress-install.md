---
layout: page
title: Создаем NginxInc Kubernetes Ingress контроллер
description: Создаем NginxInc Kubernetes Ingress контроллер
keywords: devops, linux, kubernetes, Создаем NginxInc Kubernetes Ingress контроллер
permalink: /devops/containers/kubernetes/kubeadm/ingress/nginxinc-kubernets-ingress-install/
---

# Создаем NginxInc Kubernetes Ingress контроллер

Делаю:  
24.10.2019

<br/>

    $ kubectl version --short
    Client Version: v1.16.2
    Server Version: v1.16.2

<br/>

Скрипты:  
https://github.com/nginxinc/kubernetes-ingress

<br/>

    $ kubectl create -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/ns-and-sa.yaml

    $ kubectl create -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/default-server-secret.yaml

    $ kubectl create -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/nginx-config.yaml

    $ kubectl create -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/rbac/rbac.yaml

    $ kubectl create -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/daemon-set/nginx-ingress.yaml

<br/>

    $ kubectl get ns
    NAME              STATUS   AGE
    default           Active   18m
    kube-node-lease   Active   18m
    kube-public       Active   18m
    kube-system       Active   18m
    nginx-ingress     Active   3s

<br/>

    $ kubectl -n nginx-ingress get all
    NAME                      READY   STATUS    RESTARTS   AGE
    pod/nginx-ingress-cldqh   1/1     Running   0          21s
    pod/nginx-ingress-dsp27   1/1     Running   0          21s

    NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    daemonset.apps/nginx-ingress   2         2         2       2            2           <none>          21s
