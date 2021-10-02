---
layout: page
title: Инсталляция и подготовка minikube для работы в ubuntu 20.04
description: Инсталляция и подготовка minikube для работы в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu
permalink: /containers/k8s/setup/minikube/
---

# Инсталляция и подготовка minikube для работы в ubuntu 20.04

<br/>

## Инсталляция minikube в ubuntu 20.04

<br/>

**minikube** - подготовленная виртуальная машина или контейнер с мини kubernetes сервером.
Вполне подойдет для изучения kubernetes, особенно на слабых компьютерах и ноутбуках.

<br/>

Делаю:  
12.09.2021

```shell
// Узнать последнюю версию (v1.23.0):
$ curl -s https://api.github.com/repos/kubernetes/minikube/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/'

// Установка
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

```

<br/>

```
$ minikube version
minikube version: v1.23.2
commit: 0a0ad764652082477c00d51d2475284b5d39ceed
```

<br/>

## Запуск и останов minikube

<br/>

Можно использовать VirtualBox или Docker.
Для всех случаев, когда нужно работать не с каким-то выделенным сервером на виртуалке с minikube, стоит использовать docker.

<br/>

### Запуск по умолчанию для своих примеров:

<br/>

```
$ {
    minikube --profile marley-minikube config set memory 8192
    minikube --profile marley-minikube config set cpus 4
    minikube --profile marley-minikube config set disk-size 20g

    // minikube --profile marley-minikube config set vm-driver virtualbox
    minikube --profile marley-minikube config set vm-driver docker

    minikube --profile marley-minikube config set kubernetes-version v1.22.1
    minikube start --profile marley-minikube --embed-certs

    // Enable ingress
    minikube addons --profile marley-minikube enable ingress

    // Enable registry
    // minikube addons --profile marley-minikube enable registry
}
```

<br/>

    // При необходимости можно будет удалить профиль и все созданное в профиле следующей командой.
    // $ minikube --profile marley-minikube stop && minikube --profile marley-minikube delete

    // Стартовать остановленный minikube
    // $ minikube --profile marley-minikube start

<br/>

    // Получить список установленных расширений
    $ minikube addons --profile marley-minikube list

<br/>

Далее нужно установить [kubectl](/containers/k8s/setup/tools/kubectl/)

<br/>

```
// Подключиться к dashboard можно следующей командой
$ minikube --profile marley-minikube dashboard
```

<br/>

```
// Получить токен для авторизации в kubernetes dashboard
$ kubectl -n kube-system describe secret $(qrunctl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

<br/>

### Добавить "Metal LB" (При необходимости)

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
$ minikube --profile marley-minikube ip
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

### Дополнительная инфа по развернутому kuberntes кластеру

```
$ minikube --profile marley-minikube config view
- cpus: 4
- disk-size: 20g
- kubernetes-version: v1.22.1
- memory: 8192
- vm-driver: docker
```

<br/>

```
// Подключиться к minikube по ssh
$ minikube --profile marley-minikube ssh
```

Или еще вариант

```
$ minikube --profile marley-minikube ip
$ export MINIKUBE_IP=192.168.99.100
$ ssh -i ~/.minikube/machines/marley-minikube/id_rsa docker@${MINIKUBE_IP}
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

<br/>

### Local registry

// Установка настройка  
https://github.com/kameshsampath/minikube-helpers/tree/master/registry

// Что-то нужное по tekton
https://developers.redhat.com/blog/2019/07/11/deploying-an-internal-container-registry-with-minikube-add-ons#what_do_we_need_

<br/>

```
$ minikube start --profile marley-minikube
$ minikube start --profile addons enable registry
```

<br/>

```
$ cd ~/tmp
$ git clone https://github.com/kameshsampath/minikube-helpers
$ cd minikube-helpers/registry
```

<br/>

```
$ kubectl apply -n kube-system \
  -f registry-aliases-config.yaml \
  -f node-etc-hosts-update.yaml \
  -f patch-coredns-job.yaml
```

<br/>

```
$ minikube --profile marley-minikube ssh -- sudo cat /etc/hosts
```

<br/>

**Testing**

<br/>

```
$ cd ~/tmp
$ git clone https://github.com/kameshsampath/minikube-registry-aliases-demo

$ cd minikube-registry-aliases-demo
```

<br/>

```
eval $(minikube --profile marley-minikube docker-env)
```

<br/>

```
skaffold dev --port-forward
```

<br/>

```
curl localhost:8080
```
