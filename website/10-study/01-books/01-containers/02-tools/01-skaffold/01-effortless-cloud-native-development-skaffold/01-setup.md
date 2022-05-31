---
layout: page
title: Building CI/CD Systems Using Tekton - Подготовка стенда
description: Building CI/CD Systems Using Tekton - Подготовка стенда
keywords: books, ci-cd, tekton, Подготовка стенда
permalink: /study/books/containers/kubernetes/tools/skaffold/setup/
---

# Подготовка стенда

<br/>

Делаю:  
24.10.2021

<br/>

### Подключение к бесплатному облаку от Google

Описание [здесь](/containers/kubernetes/google-cloud-shell/)

<br/>

```
$ gcloud cloud-shell ssh
```

<br/>

1. Инсталляция [MiniKube](/containers/kubernetes/minikube/setup/) (Ingress и остальное можно не устанавливать)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/containers/kubernetes/tools/kubectl/)

3. Инсталляция [Skaffold](/containers/kubernetes/tools/scaffold/)

4. Инсталляция [JDK17](//javadev.org/devtools/jdk/setup/linux/)

<br/>

```
$ java -version
java version "17.0.3.1" 2022-04-22 LTS
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