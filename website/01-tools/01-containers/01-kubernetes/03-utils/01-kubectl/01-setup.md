---
layout: page
title: Инсталляция kubectl в ubuntu 22.04
description: Инсталляция kubectl в ubuntu 22.04
keywords: tools, containers, kubernetes, kubectl, setup
permalink: /tools/containers/kubernetes/utils/kubectl/setup/
---

# Инсталляция kubectl в ubuntu 22.04

Делаю:  
2024.10.19

<br/>

### Инсталляция kubectl (клиента для работы с kubernetes)

<br/>

```shell
// Текущая стабильная версия kubectl (v1.31.0)
$ echo $(curl -sS https://storage.googleapis.com/kubernetes-release/release/stable.txt)


// Установка
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

<br/>

```
// $ kubectl version --client --output=yaml
$ kubectl version --client
Client Version: v1.31.0
Kustomize Version: v5.4.2

// Если будет нужно удалить
// $ sudo rm -rf /usr/local/bin/kubectl
```

<br/>

### Вариант установки из репо (Не проверялось)

Обратить внимание на kubernetes-xenial.

```
$ sudo apt-get update && sudo apt-get install -y apt-transport-https
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo touch /etc/apt/sources.list.d/kubernetes.list
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
```
