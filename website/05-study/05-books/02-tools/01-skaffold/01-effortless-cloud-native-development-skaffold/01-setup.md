---
layout: page
title: Building CI/CD Systems Using Tekton - Подготовка стенда
description: Building CI/CD Systems Using Tekton - Подготовка стенда
keywords: books, ci-cd, tekton, Подготовка стенда
permalink: /study/books/tools/skaffold/setup/
---

# Подготовка стенда

<br/>

Делаю:  
24.10.2021

<br/>

### Подключение к бесплатному облаку от Google

https://shell.cloud.google.com/

<br/>

**Инсталлим google-cloud-sdk**

https://cloud.google.com/sdk/docs/install

<br/>

```
$ gcloud auth login
$ gcloud cloud-shell ssh
```

<br/>

1. Инсталляция [MiniKube](/containers/k8s/setup/minikube/) (Ingress и остальное можно не устанавливать)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/containers/k8s/tools/kubectl/)

3. Инсталляция [Skaffold](/containers/k8s/tools/scaffold/)

4. Инсталляция [JDK17](//javadev.org/devtools/jdk/setup/linux/)

<br/>

```
$ java -version
java version "17.0.1" 2021-10-19 LTS
```

<br/>

```
$ sudo apt install -y jq
```

<br/>

### Установить контекст как локальный кластер

```
$ export \
  PROFILE=marley-minikube
```

<br/>

```
$ skaffold config set --kube-context ${PROFILE} local-cluster true
```

<br/>

```
$ cd ~/tmp/
$ git clone https://github.com/PacktPublishing/Effortless-Cloud-Native-App-Development-Using-Skaffold
```
