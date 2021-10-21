---
layout: page
title: Инсталляция scaffold в ubuntu 20.04
description: Инсталляция scaffold в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, tools, scaffold
permalink: /containers/k8s/tools/scaffold/setup/
---

# Инсталляция scaffold в ubuntu 20.04

Делаю:  
21.09.2021

<br/>

**scaffold - инструмент для разработки в kubernetes**

```
$ curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64

$ sudo mv skaffold /usr/local/bin
$ chmod +x /usr/local/bin/skaffold


$ skaffold version
v1.33.0
```

<br/>

### Добавление insecure-registries (не работает)

```
$ skaffold config set --global insecure-registries localhost:5000
$ cat ~/.skaffold/config
```

<br/>

### Local Cluster (не работает)

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
