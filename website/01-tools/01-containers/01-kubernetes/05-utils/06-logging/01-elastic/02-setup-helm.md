---
layout: page
title: Инсталляция с помощью Helm инструментов логирования Elastic
description: Инсталляция с помощью Helm инструментов логирования Elastic
keywords: gitops, containers, helm, logging, elastic search, kibana, metricbeat
permalink: /tools/containers/kubernetes/utils/logging/elastic/setup/helm/
---

# Инсталляция с помощью Helm инструментов логирования Elastic

<br/>

Делаю:  
2024.03.30

**Из за бана РФ, ничего не работает с российских IP. Нужно менять image на те, что лежат на dockerhub**

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

### Fluentd

<br/>

Делаю:  
2024.04.14

<br/>

```
// $ helm search repo bitnami/fluentd --versions
```

<br/>

```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install fluentd bitnami/fluentd -n logging
```

<br/>

```
// $ helm list -n logging
NAME   	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
fluentd	logging  	1       	2024-04-14 16:33:36.312796172 +0300 MSK	deployed	fluentd-6.1.1	1.16.5
```

<br/>

```
$ kubectl get pods -n logging
NAME                      READY   STATUS    RESTARTS      AGE
es-cluster-0              1/1     Running   0             42m
es-cluster-1              1/1     Running   0             41m
es-cluster-2              1/1     Running   0             41m
fluentd-0                 1/1     Running   0             2m39s
fluentd-p9bd9             1/1     Running   3 (80s ago)   2m39s
kibana-576c557879-jdjst   1/1     Running   0             39m
```

<br/>

```
$ kubectl get configmaps -n logging
NAME                    DATA   AGE
fluentd-aggregator-cm   4      2m57s
fluentd-forwarder-cm    4      2m57s
kube-root-ca.crt        1      42m
```

<br/>

```
$ kubectl get statefulsets -n logging
NAME         READY   AGE
es-cluster   3/3     97m
fluentd      1/1     58m
```

<br/>

### Metricbeat (Не пробовал)

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
