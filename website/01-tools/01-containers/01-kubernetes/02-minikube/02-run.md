---
layout: page
title: Запуск и останов minikube в ubuntu 22.04
description: Запуск и останов minikube в ubuntu 22.04
keywords: ubuntu, containers, kubernetes, minikube, run
permalink: /tools/containers/kubernetes/minikube/run/
---

# Запуск и останов minikube в ubuntu 22.04

<br/>

**Делаю:**  
2024.04.13

<br/>

Можно использовать VirtualBox или Docker.
Для всех случаев, когда нужно работать не с каким-то выделенным сервером на виртуалке с minikube, стоит использовать docker.

<br/>

### Запуск

<br/>

**driver может быть из популярных:**

- docker
- kvm2
- virtualbox
- д.р.

<br/>

```
// v1.29.3
$ LATEST_KUBERNETES_VERSION=$(curl -s https://api.github.com/repos/kubernetes/kubernetes/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
```

<br/>

```
$ echo ${LATEST_KUBERNETES_VERSION}
v1.29.3
```

<br/>

```
// Если младше 1.29.3
$ LATEST_KUBERNETES_VERSION=1.29.3
```

<br/>

```
$ export \
    PROFILE=${USER}-minikube \
    CPUS=4 \
    MEMORY=8G \
    HDD=20G \
    DRIVER=docker \
    KUBERNETES_VERSION=${LATEST_KUBERNETES_VERSION}
```

<br/>

```
$ {
    minikube --profile ${PROFILE} config set memory ${MEMORY}
    minikube --profile ${PROFILE} config set cpus ${CPUS}
    minikube --profile ${PROFILE} config set disk-size ${HDD}

    minikube --profile ${PROFILE} config set driver ${DRIVER}

    minikube --profile ${PROFILE} config set kubernetes-version ${KUBERNETES_VERSION}
    minikube start --profile ${PROFILE} --embed-certs

    // Enable ingress
    minikube addons --profile ${PROFILE} enable ingress

    // Enable registry
    // minikube addons --profile ${PROFILE} enable registry
}
```

<br/>

```
// При необходимости можно будет удалить профиль и все созданное в профиле следующей командой
// $ minikube --profile ${PROFILE} stop && minikube --profile ${PROFILE} delete

// Стартовать остановленный minikube
// $ minikube --profile ${PROFILE} start
```

<br/>

```
$ kubectl version
Client Version: v1.29.2
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.29.3
```

<br/>

```
// Получить список установленных расширений
// $ minikube addons --profile ${PROFILE} list
```

<br/>

Далее нужно установить командную утилиту для работы с кластером - [kubectl](/tools/containers/kubernetes/tools/kubectl/)

<br/>

```
// Получить текущий контекст
// $ kubectl config current-context
```

<br/>

### Сделать, чтобы Docker images хранились в выделенном ранее storage внутри контейнера, а не на основном хосте.

```
// Посмотреть команды которые установят переменные окружения
// $ minikube docker-env --profile ${PROFILE}

// Команда, которая установит нужные переменные
$ eval $(minikube -p ${PROFILE} docker-env)
```

<br/>

### Подключиться к UI (Не нужно, но можно)

<br/>

```
$ minikube addons --profile ${PROFILE} enable dashboard
```

<br/>

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
- cpus: 4
- disk-size: 20G
- driver: docker
- kubernetes-version: 1.29.2
- memory: 8G
```

<br/>

### Подключиться к minikube по ssh

<br/>

```
$ minikube --profile ${PROFILE} ssh
```

<br/>

Или еще вариант

<br/>

```
$ minikube --profile ${PROFILE} ip
$ export MINIKUBE_IP=192.168.99.100
$ ssh -i ~/.minikube/machines/${PROFILE}/id_rsa docker@${MINIKUBE_IP}
```

<br/>

### Остальное

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

## Дополнительно:

<br/>

### [Добавить "Metal LB" (При необходимости)](/tools/containers/kubernetes/tools/metal-lb/)

<br/>

### [Registry в Minikube](/tools/containers/kubernetes/minikube/setup/registry/)

<br/>

### [Ngrok Ingress Controller for Kubernetes (Доступ к kubernetes кластеру из интернетов)](/tools/containers/kubernetes/minikube/ngrok-ingress-controller/)
