---
layout: page
title: Resource Quotas & Limits in Kubernetes
description: Resource Quotas & Limits in Kubernetes
keywords: devops, linux, kubernetes, Resource Quotas & Limits in Kubernetes
permalink: /devops/containers/kubernetes/basics/resource-quotas-and-limits/
---

# Resource Quotas & Limits in Kubernetes

<br/>

Делаю: 09.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=4C-0idGOi2A&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=17

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

### Resource Quotas

    $ kubectl create namespace quota-demo-ns

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-quota-count.yaml

    $ kubectl -n quota-demo-ns describe quota quota-demo1
    Name:       quota-demo1
    Namespace:  quota-demo-ns
    Resource    Used  Hard
    --------    ----  ----
    configmaps  0     1
    pods        0     2

<br/>

    $ kubectl -n quota-demo-ns create configmap cm1 --from-literal=name=venkatn

    $ kubectl -n quota-demo-ns describe quota quota-demo1
    Name:       quota-demo1
    Namespace:  quota-demo-ns
    Resource    Used  Hard
    --------    ----  ----
    configmaps  1     1
    pods        0     2

    // Ошибка!!!
    $ kubectl -n quota-demo-ns create configmap cm2 --from-literal=name=venkatn

    $ kubectl -n quota-demo-ns delete cm cm1

<br/>

    $ kubectl -n quota-demo-ns run nginx --image=nginx --replicas=1

    $ kubectl -n quota-demo-ns describe quota quota-demo1
    Name:       quota-demo1
    Namespace:  quota-demo-ns
    Resource    Used  Hard
    --------    ----  ----
    configmaps  0     1
    pods        1     2


    $ kubectl -n quota-demo-ns scale deploy nginx --replicas=3

    $ kubectl -n quota-demo-ns get all
    NAME                         READY   STATUS    RESTARTS   AGE
    pod/nginx-7db9fccd9b-g7vgr   1/1     Running   0          2m50s
    pod/nginx-7db9fccd9b-ttzfd   1/1     Running   0          60s

    NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx   2/3     2            2           2m50s

    NAME                               DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-7db9fccd9b   3         2         2       2m50s

<br/>

    $ kubectl -n quota-demo-ns delete deploy nginx

    $ kubectl -n quota-demo-ns delete quota quota-demo1

<br/>

### Resource Limits (CPU, Memory etc.)

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-quota-mem.yaml

    $ kubectl -n quota-demo-ns describe quota quota-demo-mem
    Name:          quota-demo-mem
    Namespace:     quota-demo-ns
    Resource       Used  Hard
    --------       ----  ----
    limits.memory  0     500Mi

<br/>

    // Ошибка должны быть определены лимиты памяти
    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-pod-quota-mem.yaml

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: quota-demo-ns
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      limits:
        memory: "100Mi"
EOF
```

<br/>

    $ kubectl -n quota-demo-ns describe quota quota-demo-mem
    Name:          quota-demo-mem
    Namespace:     quota-demo-ns
    Resource       Used   Hard
    --------       ----   ----
    limits.memory  100Mi  500Mi

<br/>

    $ kubectl delete -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-pod-quota-mem.yaml

    $ kubectl -n quota-demo-ns delete quota quota-demo-mem

<br/>

### Limit range

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-quota-limitrange.yaml


    $ kubectl -n quota-demo-ns describe limitrange mem-limitrange
    Name:       mem-limitrange
    Namespace:  quota-demo-ns
    Type        Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
    ----        --------  ---  ---  ---------------  -------------  -----------------------
    Container   memory    -    -    50Mi             300Mi          -

<br/>

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-pod-quota-mem.yaml

    $ kubectl delete -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-pod-quota-mem.yaml

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/7-quota-limitrange.yaml

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-demo-mem
  namespace: quota-demo-ns
spec:
  hard:
    limits.memory: "500Mi"
    requests.memory: "100Mi"
EOF

```

<br/>

Добавили строку: requests.memory: "100Mi"

<br/>

    $ kubectl -n quota-demo-ns describe quota quota-demo-mem
    Name:            quota-demo-mem
    Namespace:       quota-demo-ns
    Resource         Used  Hard
    --------         ----  ----
    limits.memory    0     500Mi
    requests.memory  0     100Mi

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: quota-demo-ns
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      limits:
        memory: "200Mi"
EOF

```

<br/>

Ошибка, тк просит больше 100.

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: quota-demo-ns
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "50Mi"
EOF

```
