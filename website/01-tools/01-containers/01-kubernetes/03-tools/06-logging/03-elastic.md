---
layout: page
title: Инсталляция с помощью Helm инструментов логирования Elastic
description: Инсталляция с помощью Helm инструментов логирования Elastic
keywords: gitops, containers, helm, logging, elastic search, kibana, metricbeat
permalink: /tools/containers/kubernetes/tools/logging/elastic/setup/helm/
---

# Инсталляция с помощью Helm инструментов логирования Elastic

<br/>

Делаю:  
2024.03.30

**Из за бана РФ, ничего не работает с российских IP**

<br/>

### Elastic Search

```
$ kubectl create namespace logging
```

<br/>

```
$ helm repo add elastic https://helm.elastic.co

$ helm show values elastic/elasticsearch

$ helm repo update

$ cd ~/tmp
```

<br/>

```
$ curl -O https://raw.githubusercontent.com/elastic/Helm-charts/master/elasticsearch/examples/minikube/values.yaml

$ helm upgrade \
  --namespace logging \
  --install elasticsearch elastic/elasticsearch \
  --values ./values.yaml
```

<br/>

```
$ helm test elasticsearch
```

<br/>

```
$ kubectl get pods -n logging
NAME                     READY   STATUS    RESTARTS   AGE
elasticsearch-master-0   0/1     Running   0          60s
elasticsearch-master-1   0/1     Running   0          60s
elasticsearch-master-2   0/1     Running   0          60s
```

<br/>

```
// Если нужно
// $ kubectl --namespace logging port-forward svc/elasticsearch-master 9200
```

<br/>

### Kibana

<br/>

```
// $ helm show values elastic/kibana
```

<br/>

```
$ helm upgrade \
  --namespace logging \
  --install kibana elastic/kibana
```

<br/>

```
$ helm test kibana
```

<br/>

```
// Если нужно
// $ kubectl port-forward deployment/kibana-kibana 5601 --namespace logging
```

<br/>

### Metricbeat

<br/>

```
// $ helm show values elastic/metricbeat
```

<br/>

```
$ helm upgrade \
  --namespace logging \
  --install metricbeat elastic/metricbeat
```

<br/>

**Хорошая дока:**  
https://phoenixnap.com/kb/elasticsearch-helm-chart

<br/>

**Helm Chart repository links:**

<br/>

[Elastic Search Chart](https://github.com/elastic/helm-charts/blob/master/elasticsearch)
[Kibana Chart](https://github.com/elastic/helm-charts/blob/master/kibana)
