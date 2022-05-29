---
layout: page
title: Инсталляция и подготовка minikube для работы в ubuntu 20.04
description: Инсталляция и подготовка minikube для работы в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu
permalink: /containers/kubernetes/minikube/setup/
---

# Инсталляция и подготовка minikube для работы в ubuntu 20.04

<br/>

## Инсталляция minikube в ubuntu 20.04

<br/>

**Делаю:**  
05.05.2022

<br/>

**minikube** - подготовленная виртуальная машина или контейнер с мини kubernetes сервером. Вполне подойдет для изучения kubernetes, особенно на слабых компьютерах и ноутбуках.

<br/>

```shell
// Узнать последнюю версию (v1.25.2):
$ curl -s https://api.github.com/repos/kubernetes/minikube/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/'

// Установка
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

<br/>

```
$ minikube version
minikube version: v1.25.2
commit: 362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
```

<br/>

## Запуск и останов minikube

<br/>

Можно использовать VirtualBox или Docker.
Для всех случаев, когда нужно работать не с каким-то выделенным сервером на виртуалке с minikube, стоит использовать docker.

<br/>

### Запуск

<br/>

**vm-driver может быть из популярных:**

- docker
- kvm2
- virtualbox
- д.р.

<br/>

```
// Пока проблемы с новой версией
// v1.24.0
$ LATEST_KUBERNETES_VERSION=$(curl -s https://api.github.com/repos/kubernetes/kubernetes/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
```

<br/>

```
$ LATEST_KUBERNETES_VERSION=v1.23.6
```

<br/>

```
$ export \
    PROFILE=${USER}-minikube \
    MEMORY=8192 \
    CPUS=4 \
    DRIVER=docker \
    KUBERNETES_VERSION=${LATEST_KUBERNETES_VERSION}
```

<br/>

```
$ {
    minikube --profile ${PROFILE} config set memory ${MEMORY}
    minikube --profile ${PROFILE} config set cpus ${CPUS}
    minikube --profile ${PROFILE} config set disk-size 20g

    minikube --profile ${PROFILE} config set vm-driver ${DRIVER}

    minikube --profile ${PROFILE} config set kubernetes-version ${KUBERNETES_VERSION}
    minikube start --profile ${PROFILE} --embed-certs

    // Enable ingress
    minikube addons --profile ${PROFILE} enable ingress

    // Enable registry
    // minikube addons --profile ${PROFILE} enable registry
}
```

<br/>

    // При необходимости можно будет удалить профиль и все созданное в профиле следующей командой
    // $ minikube --profile ${PROFILE} stop && minikube --profile ${PROFILE} delete

    // Стартовать остановленный minikube
    // $ minikube --profile ${PROFILE} start

<br/>

    // Получить список установленных расширений
    $ minikube addons --profile ${PROFILE} list

<br/>

Далее нужно установить командную утилиту для работы с кластером - [kubectl](/containers/kubernetes/tools/kubectl/)

<br/>

```
$ kubectl config current-context
```

<br/>

```
$ minikube docker-env --profile ${PROFILE}

export ****

$ eval $(minikube -p ${PROFILE} docker-env)
```

<br/>

### Подключиться к UI (Не нужно, но можно)

```
// Подключиться к dashboard можно следующей командой
// $ minikube --profile ${PROFILE} dashboard
```

<br/>

```
// Получить токен для авторизации в kubernetes dashboard
// $ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

<br/>

### Дополнительная инфа по развернутому kuberntes кластеру

```
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

<br/>

```
$ minikube --profile ${PROFILE} config view
- vm-driver: docker
- cpus: 4
- disk-size: 20g
- kubernetes-version: v1.23.4
- memory: 8192
```

<br/>

```
// Подключиться к minikube по ssh
$ minikube --profile ${PROFILE} ssh
```

<br/>

Или еще вариант

```
$ minikube --profile ${PROFILE} ip
$ export MINIKUBE_IP=192.168.99.100
$ ssh -i ~/.minikube/machines/${PROFILE}/id_rsa docker@${MINIKUBE_IP}
```

<br/>

```
$ kubectl get events
$ kubectl get events --sort-by=.metadata.creationTimestamp
```

<br/>

```
// Установить vscode как editor по умолчанию
$ export KUBE_EDITOR="code -w"
```

<br/>

```
// Расположение профайлов
~/.minikube/profiles

// outputs the current profile
$ minikube profile

// lists all existing profiles
$ minikube profile list
```

<br/>

**Дополнительно:**

<!--
https://github.com/burrsutter/9stepsawesome/
-->

<br/>

### [Добавить "Metal LB" (При необходимости)](/containers/kubernetes/tools/metal-lb/)

<br/>

### [Registry в Minikube](/containers/kubernetes/minikube/setup/registry/)
