---
layout: page
title: Обновление контейнеров (Rolling Updates) в Kubernetes
description: Обновление контейнеров (Rolling Updates) в Kubernetes
keywords: devops, linux, kubernetes, Обновление контейнеров (Rolling Updates) в Kubernetes
permalink: /devops/containers/kubernetes/kubeadm/rolling-updates/
---

# Обновление контейнеров (Rolling Updates) в Kubernetes

<br/>

Делаю:  
10.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=MoyixCuN3UQ&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=20

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

Имеются стратегии по умолчанию. Здесь мы скриптами переопределяем поведение по умолчанию.

<br/>

### Rolling Updates

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/8-nginx-rolling-update.yaml

<br/>

    $ kubectl get all -o wide
    NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
    pod/nginx-deploy-84b67f57c4-b5jcx   1/1     Running   0          40s   10.244.2.10   node2.k8s   <none>           <none>
    pod/nginx-deploy-84b67f57c4-gcksx   1/1     Running   0          40s   10.244.1.9    node1.k8s   <none>           <none>
    pod/nginx-deploy-84b67f57c4-l6mh6   1/1     Running   0          40s   10.244.1.10   node1.k8s   <none>           <none>
    pod/nginx-deploy-84b67f57c4-ll2l8   1/1     Running   0          40s   10.244.2.9    node2.k8s   <none>           <none>

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   139m   <none>

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
    deployment.apps/nginx-deploy   4/4     4            4           40s   nginx        nginx:1.14   run=nginx

    NAME                                      DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES       SELECTOR
    replicaset.apps/nginx-deploy-84b67f57c4   4         4         4       40s   nginx        nginx:1.14   pod-template-hash=84b67f57c4,run=nginx

<br/>

Меняю версию nginx.  
nginx:1.14.2

Добавляю аннотацию.

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kubernetes.io/change-cause: "Updated to version 1.14.2"
  labels:
    run: nginx
  name: nginx-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      run: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
  minReadySeconds: 5
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        name: nginx
EOF

```

<br/>

    $ kubectl get all -o wide
    NAME                               READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
    pod/nginx-deploy-b6b6cc494-8pb7g   1/1     Running   0          29s   10.244.1.11   node1.k8s   <none>           <none>
    pod/nginx-deploy-b6b6cc494-gtjw8   1/1     Running   0          15s   10.244.1.12   node1.k8s   <none>           <none>
    pod/nginx-deploy-b6b6cc494-tgvzl   1/1     Running   0          22s   10.244.2.11   node2.k8s   <none>           <none>
    pod/nginx-deploy-b6b6cc494-wdd2n   1/1     Running   0          8s    10.244.2.12   node2.k8s   <none>           <none>

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   142m   <none>

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
    deployment.apps/nginx-deploy   4/4     4            4           2m55s   nginx        nginx:1.14.2   run=nginx

    NAME                                      DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         SELECTOR
    replicaset.apps/nginx-deploy-84b67f57c4   0         0         0       2m55s   nginx        nginx:1.14     pod-template-hash=84b67f57c4,run=nginx
    replicaset.apps/nginx-deploy-b6b6cc494    4         4         4       29s     nginx        nginx:1.14.2   pod-template-hash=b6b6cc494,run=nginx

<br/>

    $ kubectl rollout status deployment nginx-deploy

<br/>

    $ kubectl rollout history deployment nginx-deploy
    deployment.extensions/nginx-deploy
    REVISION  CHANGE-CAUSE
    1         <none>
    2         Updated to version 1.14.2

<br/>

    $ kubectl rollout history deployment nginx-deploy --revision 2
    deployment.extensions/nginx-deploy with revision #2
    Pod Template:
      Labels:	pod-template-hash=b6b6cc494
      run=nginx
      Annotations:	kubernetes.io/change-cause: Updated to version 1.14.2
      Containers:
      nginx:
        Image:	nginx:1.14.2
        Port:	<none>
        Host Port:	<none>
        Environment:	<none>
        Mounts:	<none>
      Volumes:	<none>

<br/>

    $ kubectl rollout undo deployment nginx-deploy --to-revision=1

<br/>

    $ kubectl get all -o wide
    NAME                                READY   STATUS        RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
    pod/nginx-deploy-84b67f57c4-76vrw   1/1     Running       0          3s      10.244.2.14   node2.k8s   <none>           <none>
    pod/nginx-deploy-84b67f57c4-ff8jl   1/1     Running       0          17s     10.244.2.13   node2.k8s   <none>           <none>
    pod/nginx-deploy-84b67f57c4-m8bmq   1/1     Running       0          25s     10.244.1.13   node1.k8s   <none>           <none>
    pod/nginx-deploy-84b67f57c4-njfwh   1/1     Running       0          10s     10.244.1.14   node1.k8s   <none>           <none>
    pod/nginx-deploy-b6b6cc494-8pb7g    0/1     Terminating   0          4m49s   10.244.1.11   node1.k8s   <none>           <none>

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   146m   <none>

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES       SELECTOR
    deployment.apps/nginx-deploy   4/4     4            3           7m15s   nginx        nginx:1.14   run=nginx

    NAME                                      DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         SELECTOR
    replicaset.apps/nginx-deploy-84b67f57c4   4         4         4       7m15s   nginx        nginx:1.14     pod-template-hash=84b67f57c4,run=nginx
    replicaset.apps/nginx-deploy-b6b6cc494    0         0         0       4m49s   nginx        nginx:1.14.2   pod-template-hash=b6b6cc494,run=nginx

<br/>

### Обновление в командной строке

Версии с обновлением в командной строке, кажутся неправильными.

<br/>

    $ kubectl set image deployment nginx-deploy nginx=ngix:1.15

<br/>

    $ kubectl annotate deployment nginx-deploy kubernetes.io/change-cause="Updated to version 1.15"

<br/>

    $ kubectl rollout status deployment nginx-deploy

    // чего-то проблемы со скачиванием образов на некоторых нодах.
    Waiting for deployment "nginx-deploy" rollout to finish: 1 out of 4 new replicas have been updated...

<br/>

    $ kubectl rollout history deployment nginx-deploy
    deployment.extensions/nginx-deploy
    REVISION  CHANGE-CAUSE
    2         Update nginx to 1.14.2
    3         <none>
    4         Updated to version 1.15

<br/>

    // Удаление !!!
    $ kubectl delete deploy nginx-deploy

<br/>

### Recreate

Контейнеры будут пересоздаваться а не обновляться. Без всяких там запарок, что должно быть доступно какое-то число pod и т.д. (насколько я понял)

<br/>

strategy меняется на Recreate

```
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      run: nginx
  strategy:
    type: Recreate
  minReadySeconds: 5
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        name: nginx
EOF

```

<br/>

    $ kubectl delete deploy nginx-deploy
