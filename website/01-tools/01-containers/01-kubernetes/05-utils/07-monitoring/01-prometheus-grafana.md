---
layout: page
title: Запуск Prometheus (мониторинг) и Grafana (визуализация) в kuberntes cluster с помощью heml (только для теста)
description: Запуск Prometheus (мониторинг) и Grafana (визуализация) в kuberntes cluster с помощью heml (только для теста)
keywords: tools, containers, kubernetes, monitoring, prometheus, grafana, setup, helm
permalink: /tools/containers/kubernetes/utils/monitoring/prometheus-grafana/setup/helm/
---

# Инсталляция с помощью Helm инструментов мониторинга

<br/>

### Запуск Prometheus (мониторинг) и Grafana (визуализация) в kuberntes cluster с помощью heml (только для теста)

<br/>

Делаю:  
09.02.2021

<br/>

**Upd:**

Похоже все будет в ближайшее время кучей ставиться одной командой:

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack#multiple-releases

<br/>

```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

$ helm repo update

$ kubectl create namespace monitoring

// Без ключа serviceMonitorSelectorNilUsesHelmValues=false
// Не стартовали сервис мониторы и не добавлялись в target и configuration
$ helm install prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring \
  --set kubelet.serviceMonitor.https=true \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.ruleSelectorNilUsesHelmValues=false

$ export POD_NAME=prometheus-prometheus-stack-kube-prom-prometheus-0

$ kubectl --namespace monitoring port-forward $POD_NAME 9090

localhost:9090
```

<br/>
**Полезный конфиг взял здесь:**  
https://docs.fission.io/docs/observability/prometheus/

<br/>

### Как делалось ранее

Запускаю [локальный kubernetes кластер](https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-ubuntu-20.04)

<br/>

```
$ kubectl get nodes
NAME     STATUS   ROLES                  AGE     VERSION
master   Ready    control-plane,master   11m     v1.20.2
node1    Ready    <none>                 7m1s    v1.20.2
node2    Ready    <none>                 2m38s   v1.20.2
```

<br/>

Устанавливаю [Helm3](/tools/containers/kubernetes/utils/helm/setup/) на localhost.

<br/>

```
$ mkdir ~/kubernetes-configs && cd ~/kubernetes-configs
```

<br/>

**Вроде оф.дока:**

https://github.com/prometheus-community/helm-charts

<br/>

```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

<br/>

### 01. Prometheus

```
$ vi prometheus-values.yml
```

```
alertmanager:
  persistentVolume:
    enabled: false
server:
  persistentVolume:
    enabled: false
```

<br/>

```
$ kubectl create namespace prometheus

$ helm install prometheus \
    prometheus-community/prometheus \
    -f prometheus-values.yml \
    --namespace prometheus

$ kubectl get pods -n prometheus
```

<br/>

```
$ helm list -n prometheus
NAME      	NAMESPACE 	REVISION	UPDATED                                	STATUS  	CHART            	APP VERSION
prometheus	prometheus	1       	2021-02-04 15:12:10.794089007 +0300 MSK	deployed	prometheus-13.2.1	2.24.0
```

<br/>

```
$ export POD_NAME=$(kubectl get pods --namespace prometheus -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")

$ kubectl --namespace prometheus port-forward $POD_NAME 9090
```

http://localhost:9090/graph

<br/>

### 02. Grafana

<br/>

https://grafana.com/docs/loki/latest/installation/helm/

```
$ helm repo add grafana https://grafana.github.io/helm-charts
```

<br/>

```
$ helm repo update
```

<br/>

    $ vi grafana-values.yml

```
adminPassword: password
```

```
$ kubectl create namespace loki

$ helm upgrade --install loki \
    grafana/loki \
    --namespace=loki
```

<br/>

```
$ kubectl create namespace grafana

$ helm install loki-grafana \
    grafana/grafana \
    -f grafana-values.yml \
    --namespace grafana
```

<br/>

```
$ kubectl get pods -n grafana
NAME                           READY   STATUS    RESTARTS   AGE
loki-grafana-bd544cfcb-5rgds   1/1     Running   0          56s
```

<br/>

```
$ export POD_NAME=$(kubectl get pods --namespace grafana -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=loki-grafana" -o jsonpath="{.items[0].metadata.name}")

$ kubectl --namespace grafana port-forward $POD_NAME 3000
```

<br/>

http://localhost:3000/

<br/>

```
admin / password
```

<br/>

Add data source - Prometheus

Name: Prometheus

URL: http://prometheus-server.prometheus.svc.cluster.local

Save and test

<!--

<br/>

    // ISSSUE
    $ export POD_NAME=$(kubectl get pods --namespace grafana -l "app=grafana" -o jsonpath="{.items[0].metadata.name}")


    $ export POD_NAME=grafana-7f58b98f94-8szjv

    $ kubectl --namespace grafana port-forward $POD_NAME 3000

<br/>


https://medium.com/@at_ishikawa/install-prometheus-and-grafana-by-helm-9784c73a3e97

kubectl describe pod grafana-7f58b98f94-8szjv --namespace grafana

-->

<br/>

### Пример запуска приложения с метриками:

<a href="/courses/ci-cd/implementing-a-full-ci-cd-pipeline/monitoring/">Здесь</a>

<!--
<br>

    $ vi grafana-ext.yml

```
kind: Service
apiVersion: v1
metadata:
  namespace: grafana
  name: grafana-ext
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
  -   protocol: TCP
      port: 3000
      nodePort: 30001
```

<br/>

    $ kubectl apply -f grafana-ext.yml

<br/>

### Проверка

    $ kubectl get pods -n prometheus
    NAME                                            READY   STATUS    RESTARTS   AGE
    prometheus-alertmanager-5bffbcfdbc-svlrm        2/2     Running   0          7m14s
    prometheus-kube-state-metrics-95d956569-5nqhs   1/1     Running   0          7m14s
    prometheus-node-exporter-227kr                  1/1     Running   0          7m14s
    prometheus-node-exporter-l4x8j                  1/1     Running   0          7m14s
    prometheus-pushgateway-594cd6ff6b-ztww8         1/1     Running   0          7m14s
    prometheus-server-6dc75cbb56-swfl9              2/2     Running   0          7m14s

<br/>

    $ kubectl get pods -n grafana
    NAME                       READY   STATUS    RESTARTS   AGE
    grafana-7f58b98f94-8szjv   1/1     Running   0          2m18s

<br/>

    $ kubectl get svc -n grafana
    NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    grafana       ClusterIP   10.102.141.13   <none>        80/TCP           5m2s
    grafana-ext   NodePort    10.105.70.12    <none>        3000:30001/TCP   3m26s

<br/>

http://node1.k8s:30001 -->
