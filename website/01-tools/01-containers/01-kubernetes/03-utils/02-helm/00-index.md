---
layout: page
title: Helm в Linux
description: Helm в Linux
keywords: gitops, containers, helm
permalink: /tools/containers/kubernetes/utils/helm/
---

# Helm

<br/>

### [Инсталляция Helm в linux](/tools/containers/kubernetes/utils/helm/setup/)

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

### [Инсталляция с помощью Helm инструментов мониторинга (Prometheus, Grafana)](/tools/containers/kubernetes/utils/monitoring/prometheus-grafana/setup/helm/)

### [Инсталляция с помощью Helm инструментов логирования (Elastic Search, Kibana, FluentD)](/tools/containers/kubernetes/utils/logging/elastic/setup/helm/)

### [[Philippe Collignon] Packaging Applications with Helm for Kubernetes [ENG, 30 Jul 2019]](https://github.com/webmakaka/Packaging-Applications-with-Helm-for-Kubernetes)

### [[Linkedin] Cloud Native Development with Node.js, Docker, and Kubernetes](https://github.com/webmakaka/Cloud-Native-Development-with-Node.js-Docker-and-Kubernetes)
