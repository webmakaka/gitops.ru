---
layout: page
title: Horizontal Pod Autoscaler in Kubernetes
description: Horizontal Pod Autoscaler in Kubernetes
keywords: devops, linux, kubernetes, Horizontal Pod Autoscaler in Kubernetes
permalink: /devops/containers/kubernetes/kubeadm/horizontal-pod-autoscaler/
---

# Horizontal Pod Autoscaler in Kubernetes

<br/>

Делаю:  
16.04.2019

По материалам из видео индуса:

https://www.youtube.com/watch?v=uxuyPru3_Lc&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=37

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

Рисунок индуса:

![Horizontal Pod Autoscaler](/img/devops/containers/kubernetes/kubeadm/Horizontal-Pod-Autoscaler.png 'Horizontal Pod Autoscaler'){: .center-image }

<br/>

### Устанавливаем Metrics server

https://github.com/kubernetes-incubator/metrics-server

    $ cd ~/tmp
    $ git clone https://github.com/kubernetes-incubator/metrics-server
    $ cd metrics-server/deploy/1.8+/

<br/>

    $ vi metrics-server-deployment.yaml

<br/>

После image: k8s.gcr.io/metrics-server-amd64:v0.3.1 добавили следующее

```
command:
    - /metrics-server
    - --kubelet-insecure-tls
```

<br/>

    $ kubectl create -f .

<br/>

    $ kubectl -n kube-system get pods | grep metrics-server
    metrics-server-8c667b587-kv88r       1/1     Running   0          19s

<br/>

    $ kubectl -n kube-system logs metrics-server-8c667b587-kv88r

<br/>

    $ kubectl top nodes
    NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
    master.k8s   218m         10%    1055Mi          60%
    node1.k8s    51m          2%     486Mi           27%
    node2.k8s    56m          2%     485Mi           27%

<br/>

### Nginx Deployment

    $ kubectl run nginx --image nginx
    $ kubectl expose deploy nginx --port 80 --type NodePort

<br/>

### Устанавливаем Resource Limits

    $ kubectl edit deploy nginx

В конце блока containers добавляем

```
resources:
    limits:
        cpu: "100m"
    requests:
        cpu: "100m"

```

<br/>

### Устанавливаем Horizontal Pod Autoscaler

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/004fd3d9c66f2eb453324d24e94eaabc65491895/yamls/10-hpa.yaml

<br/>

    // Можно тоже самое написать командой
    $ kubectl autoscale deploy nginx --min 1 --max 5 --cpu-percent 20

<br/>

    $ kubectl get all

    ***
    NAME                                        REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    horizontalpodautoscaler.autoscaling/nginx   Deployment/nginx   0%/20%    1         5         1          17s

<br/>

    $ kubectl top pods
    NAME                     CPU(cores)   MEMORY(bytes)
    nginx-6748b7bf68-h25bn   0m           2Mi

<br/>

    # apt install -y siege

    $ curl -I http://node1:32378
    OK

    $ siege -q -c 5 -t 2m http://node1:32378

<br/>

    // Автоматически увеличилось количество реплик
    $ kubectl get all
    ***
    NAME                                        REFERENCE          TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
    horizontalpodautoscaler.autoscaling/nginx   Deployment/nginx   101%/20%   1         5         4          9m23s

<br/>

    $ kubectl top pods
    NAME                     CPU(cores)   MEMORY(bytes)
    nginx-6748b7bf68-4lnqb   59m          2Mi
    nginx-6748b7bf68-h25bn   36m          2Mi
    nginx-6748b7bf68-jz897   43m          2Mi
    nginx-6748b7bf68-pnmzh   46m          2Mi
    nginx-6748b7bf68-zgwp7   46m          2Mi

<br/>

### Конец

После того как siege отработала, прошло еще минут 5-10 и количество реплик стало:

    $ kubectl get all
    ***
    NAME                                        REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    horizontalpodautoscaler.autoscaling/nginx   Deployment/nginx   0%/20%    1         5         1          18m

<br/>

    $ kubectl top pods
    NAME                     CPU(cores)   MEMORY(bytes)
    nginx-6748b7bf68-h25bn   0m           2Mi
