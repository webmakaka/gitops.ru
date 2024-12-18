---
layout: page
title: Подготовка окружения для тестов Istio в minikube
description: Подготовка окружения для тестов Istio в minikube
keywords: devops, containers, kubernetes, service-mesh, istio, minikube, setup
permalink: /tools/containers/kubernetes/utils/service-mesh/istio/setup/
---

# Подготовка окружения для тестов Istio в minikube

<br/>

Делаю:  
17.07.2022

<br/>

https://istio.io/docs/setup/getting-started/#download

<br/>

1. Инсталляция [MiniKube](/tools/containers/kubernetes/minikube/setup/)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/tools/containers/kubernetes/utils/kubectl/)

<br/>

### Устанавливаю istioctl на локальном хосте

<br/>

```
$ cd ~/tmp/
$ export LATEST_VERSION=$(curl --silent "https://api.github.com/repos/istio/istio/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

$ echo ${LATEST_VERSION}

// Если меньше 1.14.1
$ export LATEST_VERSION=1.14.1

$ curl -L https://istio.io/downloadIstio | sh - && chmod +x ./istio-${LATEST_VERSION}/bin/istioctl && sudo mv ./istio-${LATEST_VERSION}/bin/istioctl /usr/local/bin/
```

<br/>

```
$ istioctl version
no running Istio pods in "istio-system"
1.14.1
```

<br/>

### Установка ресурсов istio на minikube

```
$ istioctl experimental precheck
✔ No issues found when checking the cluster. Istio is safe to install or upgrade!
```

<br/>

```
// Install Istio using the demo profile
$ istioctl install --skip-confirmation \
  --set profile=demo \
  --set meshConfig.accessLogFile=/dev/stdout \
  --set meshConfig.accessLogEncoding=JSON
```

<br/>

```
$ kubectl -n istio-system wait --timeout=600s --for=condition=available deployment --all
```

<br/>

```
// install Kiali, Jaeger, Prometheus, and Grafana
$ istio_version=$(istioctl version --short --remote=false)

$ echo "Installing integrations for Istio v$istio_version"

$ {
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/kiali.yaml
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/jaeger.yaml
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/prometheus.yaml
    kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/grafana.yaml
}
```

<br/>

```
$ kubectl -n istio-system wait --timeout=600s --for=condition=available deployment --all
```

<br/>

```
$ kubectl -n istio-system get deploy
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
grafana                1/1     1            1           76s
istio-egressgateway    1/1     1            1           5m7s
istio-ingressgateway   1/1     1            1           5m7s
istiod                 1/1     1            1           5m21s
jaeger                 1/1     1            1           92s
kiali                  1/1     1            1           99s
prometheus             1/1     1            1           83s
```

<br/>

### Запуск сервисов istio (Старый вариант)

UPD. Оказазось istio уже есть среди предустановленных расширений на minikube, и можно просто активироваь.

    $ minikube addons --profile istio-lab enable istio

Но чего-то ранее не заработало из коробки на 16.9. Не хочу сейчас пробовать. Поэтому, будем ставить сами.

<br/>

**Дока:**  
https://istio.io/docs/setup/additional-setup/config-profiles/

<br/>

```
$ istioctl profile list
Istio configuration profiles:
    default
    demo
    empty
    external
    minimal
    openshift
    preview
    remote
```

<br/>

```
// $ istioctl manifest install -y --set profile=demo
$ istioctl manifest install -y --set profile=default
```

<br/>

```
// После выполнения данной команды, **новые** pod будут "проксируемыми". Т.е. старые нужно пересоздать.
$ kubectl label namespace default istio-injection=enabled
```

<br/>

```
$ kubectl get ns --show-labels | grep istio
default           Active   12m     istio-injection=enabled,kubernetes.io/metadata.name=default
istio-system      Active   2m35s   kubernetes.io/metadata.name=istio-system
```

<br/>

### [Добавляю Metal LB](/tools/containers/kubernetes/minikube/setup/)

<br/>

**Всевозможные проверки**

<br/>

    $ watch kubectl -n istio-system get all

<br/>

    // 13
    $ kubectl get crds | grep istio | wc -l
    13

<br/>

```
$ kubectl get pods -n istio-system
NAME                                   READY   STATUS    RESTARTS   AGE
istio-ingressgateway-8dbb57f65-hc5dz   1/1     Running   0          10m
istiod-7859559dd-68rdz                 1/1     Running   0          10m
```

<br/>

```
$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.106.144.8   192.168.49.20   15021:32220/TCP,80:32270/TCP,443:31357/TCP,15012:30385/TCP,15443:30134/TCP   3m27s
istiod                 ClusterIP      10.99.91.237   <none>          15010/TCP,15012/TCP,443/TCP,15014/TCP                                        3m45s
```

<!--

<br/>

### Дополнительные сервисы (Prometheus, Grafana, Kiali, Jaeger):

<br/>

```
$ export LATEST_VERSION=$(curl --silent "https://api.github.com/repos/istio/istio/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

$ cd ~/tmp/istio-${LATEST_VERSION}/samples/addons/
$ kubectl apply -n istio-system -f ./
```

<br/>

Чтобы запустился только Kiali нужно повторить

<br/>

```
$ kubectl apply -n istio-system -f ./kiali.yaml
```

<br/>

```
$ kubectl -n istio-system get pods
NAME                                   READY   STATUS    RESTARTS   AGE
grafana-68cc7d6d78-gvhsf               1/1     Running   0          32s
istio-ingressgateway-8dbb57f65-hc5dz   1/1     Running   0          17m
istiod-7859559dd-68rdz                 1/1     Running   0          18m
jaeger-5d44bc5c5d-4f4gl                1/1     Running   0          32s
kiali-fd9f88575-tml7d                  1/1     Running   0          31s
prometheus-77b49cb997-hs7nq            2/2     Running   0          31s
```

-->

<br/>

```
$ kubectl -n istio-system port-forward svc/grafana 3000
$ kubectl -n istio-system port-forward svc/prometheus 9090
$ kubectl -n istio-system port-forward svc/kiali 20001
```

<br/>

Пока непонятно как работать с jaeger

<br/>

```
$ kubectl -n istio-system port-forward svc/jaeger-collector 14268
```

<br/>

### Zipkin и Prometheus Operator

```
$ cd ~/tmp/istio-1.9.0/samples/addons/extras
$ kubectl apply -n istio-system -f ./
```

<br/>

```
$ kubectl apply -n istio-system -f ./prometheus-operator.yaml
unable to recognize "./prometheus-operator.yaml": no matches for kind "PodMonitor" in version "monitoring.coreos.com/v1"
unable to recognize "./prometheus-operator.yaml": no matches for kind "ServiceMonitor" in version "monitoring.coreos.com/v1"
```

<br/>

В общем, нужно еще и добавлять из стандартной установки компоненты, чтобы он понимал, что за ServiceMonitor и PodMonitor.

Пока неактуально.
