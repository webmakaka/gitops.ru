---
layout: page
title: Building CI/CD Systems Using Tekton - Installation and Getting Started
description: Building CI/CD Systems Using Tekton - Installation and Getting Started
keywords: books, ci-cd, tekton, Installation and Getting Started
permalink: /study/books/ci-cd/tekton/building-ci-cd-systems-using-tekton/installation-and-getting-started/
---

# Chapter 3. Installation and Getting Started

<br/>

Делаю:  
24.10.2021

<br/>

### Подключение к бесплатному облаку от Google

Описание [здесь](/tools/containers/kubernetes/google-cloud-shell/)

<br/>

```
$ gcloud cloud-shell ssh
```

<br/>

1. Инсталляция [MiniKube](/tools/containers/kubernetes/minikube/setup/) (Ingress и остальное можно не устанавливать)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/tools/containers/kubernetes/tools/kubectl/)

<br/>

#### Инсталляция Tekton CLI

<br/>

```
$ cd ~/tmp/
```

<br/>

```
$ vi tekton-setup.sh
```

<br/>

```
#!/bin/bash

export LATEST_VERSION=$(curl --silent "https://api.github.com/repos/tektoncd/cli/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

export LATEST_VERSION_SHORT=$(curl --silent "https://api.github.com/repos/tektoncd/cli/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/' | cut -c 2-)

curl -LO "https://github.com/tektoncd/cli/releases/download/${LATEST_VERSION}/tkn_${LATEST_VERSION_SHORT}_$(uname -s)_$(uname -m).tar.gz"

sudo tar xvzf tkn_${LATEST_VERSION_SHORT}_$(uname -s)_$(uname -m).tar.gz -C /usr/local/bin/ tkn
```

<br/>

```
$ chmod +x tekton-setup.sh
$ ./tekton-setup.sh
```

<br/>

```
$ tkn version
Client version: 0.21.0
```

<br/>

#### Добавляем Tekton CRD в MiniKube

<br/>

```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

<br/>

#### Добавление Tekton Dashboard в MiniKube (Если нужно)

<br/>

```
$ kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml
```

<br/>

**Подключиться к dashboard**

<br/>

```
$ kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 8080:9097
```

<br/>

https://shell.cloud.google.com/

Вверху справа 3-й значок слева

Preview on port 8080

Открывается окно Tekton Dashboard
