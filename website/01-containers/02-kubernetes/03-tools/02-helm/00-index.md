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

$ helm upgrade \
  --namespace logging \
  --install elasticsearch elastic/elasticsearch \
  --values ./values.yaml
```

<br/>

```
// Если нужно
// $ kubectl --namespace logging port-forward svc/elasticsearch-master 9200
```

<br/>

### Kibana

```
$ helm upgrade \
  --namespace logging \
  --install kibana elastic/kibana
```

<br/>

```
// Если нужно
// $ kubectl port-forward deployment/kibana-kibana 5601 --namespace logging
```

<br/>

### Metricbeat (Вроде как лучшее решение чем Fluent-bit)

Перестало работать!.
Нужно или самому править конфиги или подождать когда поправят для работы на последних версиях k8s.

<br/>

```
$ helm upgrade \
  --namespace logging \
  --install metricbeat elastic/metricbeat
```

<br/>

```
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: [unable to recognize "": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1", unable to recognize "": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"]
```

<br/>

```
$ kubectl \
  --namespace logging logs elasticsearch-master-0
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
$ helm upgrade \
  --namespace logging \
  --install fluentd bitnami/fluentd
```

<br/>

**Helm Chart repository links:**

<br/>

[Fluentd Chart](https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx)
[Elastic Search Chart](https://github.com/elastic/helm-charts/blob/master/elasticsearch)
[Kibana Chart](https://github.com/elastic/helm-charts/blob/master/kibana)
