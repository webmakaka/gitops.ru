---
layout: page
title: Инсталляция scaffold в ubuntu 20.04
description: Инсталляция scaffold в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, tools, scaffold
permalink: /tools/containers/kubernetes/utils/scaffold/
---

# Инсталляция scaffold в ubuntu 20.04

Делаю:  
05.05.2022

<br/>

**scaffold - инструмент, помогающий в разработке kubernetes. Автоматическое обновление контейнеров, при изменении исходиников. LiveReload, только сразу в контейнерах**

<br/>

```
$ cd ~/tmp/
$ curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64

$ sudo mv skaffold /usr/local/bin
$ chmod +x /usr/local/bin/skaffold

$ skaffold version
v1.38.0
```

<br/>

### Skaffold config (Дополнительные, необязательные настройки)

```
$ skaffold config
```

<br/>

```
$ cat ~/.skaffold/config
```

<br/>

### Настроить skaffold, чтобы использовался local Kubernetes cluster

<br/>

```
$ export \
    PROFILE=${USER}-minikube
```

<br/>

```
$ minikube docker-env -p ${PROFILE}
$ eval $(minikube -p ${PROFILE} docker-env)
```

<br/>

```
$ skaffold config set --kube-context ${PROFILE} local-cluster true
```

<br/>

If your Kubernetes context is set to a local Kubernetes cluster, then there is no need to push an image to a remote Kubernetes cluster. Instead, Skaffold will move the image to the local Docker daemon to speed up the development cycle.

<br/>

https://skaffold.dev/docs/environment/local-cluster/

<br/>

### Добавление insecure-registries (возможно не работает, нужно поразбираться)

```
$ skaffold config set --global insecure-registries localhost:5000
$ cat ~/.skaffold/config
```
