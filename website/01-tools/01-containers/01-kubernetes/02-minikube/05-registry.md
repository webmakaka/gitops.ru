---
layout: page
title: Удаленное подключение к хосту с minikube в ubuntu 20.04
description: Удаленное подключение к хосту с minikube в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu, remote
permalink: /tools/containers/kubernetes/minikube/setup/registry/
---

# Registry в Minikube

<br/>

**Делаю:**  
04.10.2021

<br/>

// Установка настройка  
https://github.com/kameshsampath/minikube-helpers/tree/master/registry

// Что-то нужное по tekton
https://developers.redhat.com/blog/2019/07/11/deploying-an-internal-container-registry-with-minikube-add-ons#what_do_we_need_

<br/>

```
$ minikube start --profile ${PROFILE}
$ minikube start --profile addons enable registry
```

<br/>

```
$ cd ~/tmp
$ git clone https://github.com/kameshsampath/minikube-helpers
$ cd minikube-helpers/registry
```

<br/>

```
$ kubectl apply -n kube-system \
  -f registry-aliases-config.yaml \
  -f node-etc-hosts-update.yaml \
  -f patch-coredns-job.yaml
```

<br/>

```
$ minikube --profile marley-minikube ssh -- sudo cat /etc/hosts
```

<br/>

**Testing**

<br/>

```
$ cd ~/tmp
$ git clone https://github.com/kameshsampath/minikube-registry-aliases-demo

$ cd minikube-registry-aliases-demo
```

<br/>

```
$ eval $(minikube --profile ${PROFILE} docker-env)
```

<br/>

```
$ skaffold dev --port-forward
```

<br/>

```
$ curl localhost:8080
```
