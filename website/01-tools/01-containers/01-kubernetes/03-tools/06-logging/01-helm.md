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

С 8 версией чего-то не оч. все хорошо! Нужно разбираться!

<br/>

### Elastic Search

<br/>

https://github.com/elastic/helm-charts/tree/main/elasticsearch/examples/minikube

<br/>

```
$ minikube --profile ${PROFILE} addons enable default-storageclass
$ minikube --profile ${PROFILE} addons enable storage-provisioner
```

<br/>

```
$ kubectl create namespace logging
```

<br/>

```
$ helm repo add elastic https://helm.elastic.co
```

<br/>

```
// $ helm search repo elastic/elasticsearch --versions

// $ helm show values elastic/elasticsearch

$ helm repo update
```

<br/>

```
$ cd ~/tmp

$ curl -O https://raw.githubusercontent.com/elastic/Helm-charts/master/elasticsearch/examples/minikube/values.yaml

$ helm upgrade \
  --namespace logging \
  --install elasticsearch elastic/elasticsearch \
  --values ./values.yaml \
  --version 8.5.1
```

<br/>

```
// Watch all cluster members come up
// $ kubectl get pods --namespace=logging -l app=elasticsearch-master -w
```

<br/>

```
// Retrieve elastic user's password
// $ kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```

<br/>

```
// Нужно как-то передать пароль, чтобы норм отрабатывало
// Test cluster health using Helm test
// $ helm --namespace=logging test elasticsearch
```

<br/>

```
// Если нужно проверить
// $ kubectl --namespace logging port-forward svc/elasticsearch-master 9200
```

<br/>

```
// [OK!]
// $ curl -k -u elastic:password https://localhost:9200
```

<br/>

```json
{
  "name": "elasticsearch-master-0",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "5AIZquHDR9GNLpp-o7oNCA",
  "version": {
    "number": "8.5.1",
    "build_flavor": "default",
    "build_type": "docker",
    "build_hash": "c1310c45fc534583afe2c1c03046491efba2bba2",
    "build_date": "2022-11-09T21:02:20.169855900Z",
    "build_snapshot": false,
    "lucene_version": "9.4.1",
    "minimum_wire_compatibility_version": "7.17.0",
    "minimum_index_compatibility_version": "7.0.0"
  },
  "tagline": "You Know, for Search"
}
```

<br/>

```
// Удаление
// $ helm delete \
  --namespace logging \
  elasticsearch
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

```
// Удаление
// $ helm delete \
  --namespace logging \
  kibana
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

```
// Удаление
// $ helm delete \
  --namespace logging \
  metricbeat
```
