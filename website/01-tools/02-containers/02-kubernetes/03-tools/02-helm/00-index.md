---
layout: page
title: Helm в Linux
description: Helm в Linux
keywords: gitops, containers, helm
permalink: /tools/containers/kubernetes/tools/helm/
---

# Helm

<br/>

### [Инсталляция Helm в linux](/tools/containers/kubernetes/tools/helm/setup/)

<br/>

### Install nginx-ingress controller

```
$ helm repo add stable https://charts.helm.sh/stable
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm install nginx-ingress ingress-nginx/ingress-nginx
```

<br/>

https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx

<br/>

### [Инсталляция с помощью Helm инструментов мониторинга (Prometheus, Grafana)](/tools/containers/kubernetes/tools/helm/monitoring/)

### [Инсталляция с помощью Helm инструментов логирования (Elastic Search, Kibana, FluentD)](/tools/containers/kubernetes/tools/helm/logging/)
