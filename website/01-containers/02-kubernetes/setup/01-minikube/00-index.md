---
layout: page
title: Инсталляция и подготовка minikube для работы в ubuntu 20.04
description: Инсталляция и подготовка minikube для работы в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu
permalink: /containers/kubernetes/setup/minikube/
---

# Инсталляция и подготовка minikube для работы в ubuntu 20.04

## Инсталляция minikube в ubuntu 20.04.1

<br/>

**minikube** - подготовленная виртуальная машина или контейнер с мини kubernetes сервером.
Вполне подойдет для изучения kubernetes, особенно на слабых компьютерах и ноутбуках.

<br/>

Делаю:  
28.09.2020

```shell
-- Последняя версия (v1.13.1):
$ curl -s https://api.github.com/repos/kubernetes/minikube/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/'

-- Установка
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

```

<br/>

```
$ minikube version
minikube version: v1.16.0
commit: 9f1e482427589ff8451c4723b6ba53bb9742fbb1
```

<br/>

## Запуск и останов minikube

<br/>

Делаю:  
21.02.2021

Можно использовать VirtualBox или Docker. Для всех случаев, когда нужно работать не с каким-то выделенным сервером на виртуалке с minikube, стоит использовать docker.

<br/>

### Запуск по умолчанию для своих примеров:

<br/>

```
$ {
    minikube --profile my-profile config set memory 8192
    minikube --profile my-profile config set cpus 4
    minikube --profile my-profile config set disk-size 20g

    // minikube --profile my-profile config set vm-driver virtualbox
    minikube --profile my-profile config set vm-driver docker

    minikube --profile my-profile config set kubernetes-version v1.20.4
    minikube start --profile my-profile --embed-certs
}
```

<br/>

    // Удалить
    // $ minikube --profile my-profile stop && minikube --profile my-profile delete

<br/>

    // Enable ingress
    $ minikube addons --profile my-profile enable ingress

<br/>

### Добавляю "Metal LB"

Metal LB позволит получить внешний IP в миникубе на локалхосте. Аналогично тому, как это происходит в облаках, когда облачный сервис выделяет ip адрес, к котому можно будет подключиться извне.

<br/>

```
$ LATEST_VERSION=$(curl --silent "https://api.github.com/repos/metallb/metallb/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

$ echo ${LATEST_VERSION}
```

<br/>

```
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/${LATEST_VERSION}/manifests/namespace.yaml

$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/${LATEST_VERSION}/manifests/metallb.yaml

# On first install only
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

<br/>

```
$ minikube --profile my-profile ip
192.168.49.2
```

<br/>

Задаем диапазон ip адресов, которые можно выдать виртуальному сервису. Нужно, чтобы он был в той же подсети, что и ip minikube.

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: custom-ip-space
      protocol: layer2
      addresses:
      - 192.168.49.20-192.168.49.30
EOF
```

<!--

<br/>

```
$ export INGRESS_HOST=$(kubectl \
 --namespace istio-system \
 get service istio-ingressgateway \
 --output jsonpath="{.status.loadBalancer.ingress[0].ip}")

$ echo ${INGRESS_HOST}
```
-->

```
$ kubectl get pods --all-namespaces
```

<br/>

### Дополнительная инфа

```
$ minikube --profile my-profile config view
- cpus: 4
- kubernetes-version: v1.20.2
- memory: 8192
- vm-driver: docker
```

<br/>

```
// Подключиться к minikube по ssh
$ minikube --profile my-profile ssh
```

Или еще вариант

```
$ minikube --profile my-profile ip
$ export MINIKUBE_IP=192.168.99.100
$ ssh -i ~/.minikube/machines/my-profile/id_rsa docker@${MINIKUBE_IP}
```

<br/>

```
$ kubectl get events
$ kubectl get events --sort-by=.metadata.creationTimestamp
```

<br/>

```
// Editor по умолчанию vscode
$ export KUBE_EDITOR="code -w"
```

<br/>

```
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.159:2376"
export DOCKER_CERT_PATH="/home/marley/.minikube/certs"

# Run this command to configure your shell:

# eval $(minikube docker-env)
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

https://github.com/burrsutter/9stepsawesome/
