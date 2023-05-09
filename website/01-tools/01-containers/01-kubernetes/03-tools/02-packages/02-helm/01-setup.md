---
layout: page
title: Подготовка и установка Helm
description: Подготовка и установка Helm
keywords: devops, containers, kubernetes, linux, helm, setup
permalink: /tools/containers/kubernetes/tools/packages/helm/setup/
---

# Подготовка и установка Helm

<br/>

Делаю:  
30.04.2023

<br/>

```
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

$ helm version --short --client
v3.11.3+g3232493
```

<br/>

```
// LIST
$ helm repo list
```

<br/>

### Просто для примера. Репо протухшее!

```
$ helm repo add stable https://charts.helm.sh/stable

// Удалить тухлое heml репо
// $ helm repo remove stable
```

<br/>

```
// UPDATE
$ helm repo update
```
