---
layout: page
title: Подготовка и установка Helm
description: Подготовка и установка Helm
keywords: devops, containers, kubernetes, linux, helm, setup
permalink: /tools/containers/kubernetes/tools/packages/helm/setup/
---

# Подготовка и установка Helm

<br/>

**Делаю:**  
2024.02.02

<br/>

```
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

$ helm version --short --client
v3.14.0+g3fc9f4b
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

<br/>

### Команды, которые приходилось выполнять

```
$ helm pull --insecure-skip-tls-verify oci://registry/release/service-helmchart --version 24020104 --untar
```

<br/>

```
$ helm template . -f servicename-3.yaml
```

<br/>

```
$ helm install -n myNS servicename . -f servicename-3.yaml
```
