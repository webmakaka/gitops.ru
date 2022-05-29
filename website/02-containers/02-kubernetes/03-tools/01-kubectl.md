---
layout: page
title: Инсталляция kubectl в ubuntu 20.04
description: Инсталляция kubectl в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, kubectl
permalink: /containers/kubernetes/tools/kubectl/
---

# Инсталляция kubectl в ubuntu 20.04

Делаю:  
08.03.2022

<br/>

### Инсталляция kubectl (клиента для работы с kubernetes)

<br/>

```shell
// Текущая стабильная версия kubernetes (v1.23.4)
$ echo $(curl -sS https://storage.googleapis.com/kubernetes-release/release/stable.txt)


// Установка
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

<br/>

```
$ kubectl version --client --short
Client Version: v1.23.4


// Если будет нужно удалить
// $ sudo rm -rf /usr/local/bin/kubectl
```
