---
layout: page
title: Helm в Linux
description: Helm в Linux
keywords: gitops, containers, helm
permalink: /containers/kubernetes/tools/helm/
---

# Helm

<br/>

### [Инсталляция Helm в linux](/containers/kubernetes/tools/helm/setup/)

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

### Elastic Search

```

$ kubectl create namespace logging

$ helm repo add elastic https://helm.elastic.co

$ helm repo update

$ cd ~/tmp

$ curl -O https://raw.githubusercontent.com/elastic/Helm-charts/master/elasticsearch/examples/minikube/values.yaml

$ helm upgrade --install elasticsearch elastic/elasticsearch --namespace logging -f ./values.yaml

```

<br/>

```
$ kubectl port-forward svc/elasticsearch-master 9200 --namespace logging
```

<br/>

### Kibana

```
$ helm upgrade --install kibana elastic/kibana --namespace logging
```

<br/>

```
$ kubectl port-forward deployment/kibana-kibana 5601 --namespace logging
```

<br/>

### Metricbeat (Вроде как лучшее решение чем Fluent-bit)

```
$ helm upgrade --install metricbeat elastic/metricbeat --namespace logging
```

<br/>

```
$ kubectl --namespace logging logs elasticsearch-master-0
```

https://logz.io/blog/deploying-the-elk-stack-on-kubernetes-with-helm/

<br/>

### Fluent-bit

https://docs.fluentbit.io/manual/installation/kubernetes

```
$ helm repo add fluent https://fluent.github.io/helm-charts
$ helm install fluent-bit fluent/fluent-bit
```

<br/>

### Install Fluentd

```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install fluentd bitnami/fluentd
```

<br/>

**Helm Chart repository links:**

<br/>

[Fluentd Chart](https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx)
[Elastic Search Chart](https://github.com/elastic/helm-charts/blob/master/elasticsearch)
[Kibana Chart](https://github.com/elastic/helm-charts/blob/master/kibana)
