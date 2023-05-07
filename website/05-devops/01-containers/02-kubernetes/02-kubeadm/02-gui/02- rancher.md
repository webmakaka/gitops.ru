---
layout: page
title: Устанавливаем WEB GUI для управления Kubernetes кластером (Rancher)
description: Устанавливаем WEB GUI для управления Kubernetes кластером (Rancher)
keywords: devops, linux, kubernetes, Устанавливаем WEB GUI для управления Kubernetes кластером (Rancher)
permalink: /devops/containers/kubernetes/kubeadm/gui/rancher/
---

# Устанавливаем WEB GUI для управления Kubernetes кластером (Rancher)

<br/>

Делаю: 10.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=jF5L6IgZ5To&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=19

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

### Установка

https://rancher.com/

<br/>

### На хост машине

    $ docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -v /opt/rancher:/var/lib/rancher rancher/rancher:latest

http://localhost

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-01.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-02.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-03.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-04.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-05.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

    $ curl --insecure -sfL https://192.168.0.1/v3/import/6nhff4lmlzpk89nj65mk8w8jnktvl8x7zzlmcdd5strzjmzc9474vz.yaml | kubectl apply -f -

    $ kubectl apply -f https://192.168.0.1/v3/import/6nhff4lmlzpk89nj65mk8w8jnktvl8x7zzlmcdd5strzjmzc9474vz.yaml

<br/>

    $ kubectl -n cattle-system get all
    NAME                                        READY   STATUS    RESTARTS   AGE
    pod/cattle-cluster-agent-6855b5c9c5-ffllr   1/1     Running   0          62s
    pod/cattle-node-agent-5bddv                 1/1     Running   0          51s
    pod/cattle-node-agent-7bhk4                 1/1     Running   0          47s

    NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    daemonset.apps/cattle-node-agent   2         2         2       2            2           <none>          100s

    NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/cattle-cluster-agent   1/1     1            1           100s

    NAME                                              DESIRED   CURRENT   READY   AGE
    replicaset.apps/cattle-cluster-agent-5d9c7cdb89   0         0         0       100s
    replicaset.apps/cattle-cluster-agent-6855b5c9c5   1         1         1       62s

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-06.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-07.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-08.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-09.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-10.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-11.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-12.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-13.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-14.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-15.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-16.png 'kubernetes GUI Rancher'){: .center-image }

<br/>

![kubernetes GUI Rancher](/img/devops/containers/kubernetes/kubeadm/gui/rancher/rancher-17.png 'kubernetes GUI Rancher'){: .center-image }
