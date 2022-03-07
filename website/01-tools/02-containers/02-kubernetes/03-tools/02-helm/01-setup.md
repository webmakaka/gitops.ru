---
layout: page
title: Подготовка и установка Helm
description: Подготовка и установка Helm
keywords: devops, containers, kubernetes, linux, helm, setup
permalink: /tools/containers/kubernetes/tools/helm/setup/
---

# Подготовка и установка Helm

<br/>

Делаю:  
04.11.2021

<br/>

```
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

$ helm version --short --client
v3.7.1+g1d11fcb
```

<br/>

### Просто для прмера. Репо протухшее!

```
$ helm repo add stable https://charts.helm.sh/stable

// Удалить тухлое heml репо
// $ helm repo remove stable
```

<br/>

```
$ helm repo update
```
