---
layout: page
title: Logging in Kubernetes with EFK Stack | Подготовка окружения
description: Logging in Kubernetes with EFK Stack | Подготовка окружения
keywords: courses, containers, kubernetes, tools, logging, elastic, fluentd, kibana, Подготовка окружения
permalink: /courses/containers/kubernetes/tools/logging/efk/logging-in-kubernetes-with-efk-stack/env/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack | The Complete Guide [ENG, 2021] : Подготовка окружения

<br/>

Делаю:  
05.11.2021

<br/>

1. Инсталляция [MiniKube](/tools/containers/kubernetes/minikube/setup/)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/tools/containers/kubernetes/tools/kubectl/)

3. Инсталляция [Helm](/tools/containers/kubernetes/tools/packages/helm/setup/)

4. Инсталляция [Elastic Search, Kibana, Fluentd](/tools/containers/kubernetes/tools/packages/helm/logging/)

<br/>

```
$ kubectl --namespace logging get pods
NAME                             READY   STATUS    RESTARTS      AGE
elasticsearch-master-0           1/1     Running   0             3m58s
elasticsearch-master-1           1/1     Running   0             3m58s
elasticsearch-master-2           1/1     Running   0             3m58s
fluentd-0                        1/1     Running   0             2m44s
fluentd-hvntt                    1/1     Running   3 (48s ago)   2m45s
kibana-kibana-56689685dc-8prxl   1/1     Running   0             3m17s
```
