---
layout: page
title: Визуализация работы контейнеров - kube-ops-view and kubebox
description: Визуализация работы контейнеров - kube-ops-view and kubebox
keywords: devops, linux, kubernetes, Визуализация работы контейнеров - kube-ops-view and kubebox
permalink: /devops/containers/kubernetes/monitoring/kube-ops-view-and-kubebox/
---

# Визуализация работы контейнеров: kube-ops-view and kubebox

<br/>

Делаю: 22.04.2019

По материалам из видео индуса:

https://www.youtube.com/watch?v=auVLHYSZM_A&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=36

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

### Устанавливаем kube-ops-view

https://github.com/hjacobs/kube-ops-view

<br/>

    $ mkdir /tmp/kube-ops-view && cd /tmp/kube-ops-view
    $ git clone https://github.com/hjacobs/kube-ops-view.git
    $ cd kube-ops-view/deploy/

<br/>

    $ rm -f ingress.yaml
    $ vi service.yaml

    type: ClusterIP меняем на NodePort

<br/>

    $ kubectl create -f .

<br/>

    $ kubectl get all

<br/>

http://node1:32178

http://node1:32178/#scale=2.0

<br/>

![kube-ops-view](/img/devops/containers/kubernetes/monitoring/kube-ops-view.png 'kube-ops-view'){: .center-image }

<br/>

    $ kubectl delete -f .

<br/>

### KubeBox

https://github.com/astefanutti/kubebox

    $ curl -Lo kubebox https://github.com/astefanutti/kubebox/releases/download/v0.4.0/kubebox-linux && chmod +x kubebox

    $ ./kubebox

    Tab + n - выбор namespace

<br/>

![KubeBox console](/img/devops/containers/kubernetes/monitoring/kubebox-console.png 'KubeBox console'){: .center-image }

<br/>

    $ cd ~/tmp

    $ wget https://raw.github.com/astefanutti/kubebox/master/kubernetes.yaml

    $ vi kubernetes.yaml

Добавляем в spec

    spec:
        type: NodePort

Удаляем секцию с ingress

    $ kubectl create -f kubernetes.yaml

<br/>

    $ kubectl get all
    NAME                           READY   STATUS    RESTARTS   AGE
    pod/kubebox-57b8bcc6cf-746jf   1/1     Running   0          15s

    NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    service/kubebox      NodePort    10.103.75.15   <none>        8080:30868/TCP   16s
    service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          26m

    NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/kubebox   1/1     1            1           16s

    NAME                                 DESIRED   CURRENT   READY   AGE
    replicaset.apps/kubebox-57b8bcc6cf   1         1         1       16s

<br/>

![KubeBox Web](/img/devops/containers/kubernetes/monitoring/kubebox-web.png 'KubeBox Web'){: .center-image }

    $ kubectl delete -f kubernetes.yaml
