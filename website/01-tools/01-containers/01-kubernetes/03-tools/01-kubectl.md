---
layout: page
title: Инсталляция kubectl в ubuntu 20.04
description: Инсталляция kubectl в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, kubectl
permalink: /tools/containers/kubernetes/tools/kubectl/
---

# Инсталляция kubectl в ubuntu 20.04

Делаю:  
19.01.2023

<br/>

### Инсталляция kubectl (клиента для работы с kubernetes)

<br/>

```shell
// Текущая стабильная версия kubectl (v1.26.1)
$ echo $(curl -sS https://storage.googleapis.com/kubernetes-release/release/stable.txt)


// Установка
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

<br/>

```
$ kubectl version --client --short
Client Version: v1.26.0
Kustomize Version: v4.5.7

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

<br/>

### Команды

```
// Скачать лог

$ export NAME_SPACE=myspace
$ export POD=mypod

$ kubectl --namespace ${NAME_SPACE} logs $(kubectl get pods --namespace ${NAME_SPACE} -l "app=${POD}" -o jsonpath="{.items[0].metadata.name}") > ~/logs/${POD}.logs.txt
```

```
// Скачать каталог
$ kubectl cp myns/mypod-with-id:/app ~/tmp/myappname/
```
