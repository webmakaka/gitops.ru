---
layout: page
title: Инсталляция scaffold в ubuntu 20.04
description: Инсталляция scaffold в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, tools, scaffold
permalink: /containers/k8s/tools/scaffold/
---

# Инсталляция scaffold в ubuntu 20.04

Делаю:  
23.10.2021

<br/>

**scaffold - инструмент для разработки в kubernetes**

```
$ cd ~/tmp/
$ curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64

$ sudo mv skaffold /usr/local/bin
$ chmod +x /usr/local/bin/skaffold

$ skaffold version
v1.33.0
```

<br/>

### Skaffold config

```
$ skaffold config
```

<br/>

```
$ cat ~/.skaffold/config
```

<br/>

### Объяснить skaffold, что используется local Kubernetes cluster

<br/>

If your Kubernetes context is set to a local Kubernetes cluster, then there is no need to push an image to a remote Kubernetes cluster. Instead, Skaffold will move the image to the local Docker daemon to speed up the development cycle.

<br/>

https://skaffold.dev/docs/environment/local-cluster/

<br/>

```
build:
  local:
    push: false
```

<br/>

```
$ kubectl config current-context
marley-minikube
```

<br/>

```
$ source minikube docker-env -p marley-minikube
$ skaffold config set --kube-context marley-minikube local-cluster true
```

<br/>

### Добавление insecure-registries (не работает)

```
$ skaffold config set --global insecure-registries localhost:5000
$ cat ~/.skaffold/config
```

Можно посмотреть работающий вариант в доке по инсталляции.
