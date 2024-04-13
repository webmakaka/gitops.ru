---
layout: page
title: Инсталляция инструментов логирования Elastic - Elasticsearch, Kibana, Fluentd
description: Инсталляция инструментов логирования Elastic - Elasticsearch, Kibana, Fluentd
keywords: gitops, containers, logging, elastic search, kibana, fluentd
permalink: /tools/containers/kubernetes/tools/logging/elastic/setup/
---

# Инсталляция инструментов логирования Elastic - Elasticsearch, Kibana, Fluentd

<br/>

Делаю:  
2024.04.13

**Из за бана РФ, ничего не работает с российских IP**

Не удается запустить. Ошибки.

<br/>

**YouTube**

https://www.youtube.com/watch?v=d7IATODGxUI

https://www.youtube.com/watch?v=rnKNfLArS7M

https://www.youtube.com/watch?v=fb3CKSJTV-Q

**GIT**  
https://github.com/wildmakaka/efk-setup

https://github.com/TechonTerget/efk_on_k8s/tree/feature

<br/>

### Предустановки

```
// In order to properly support the required persistent volume claims for the Elasticsearch StatefulSet, the default-storageclass and storage-provisioner minikube addons must be enabled.
$ minikube --profile ${PROFILE} addons enable default-storageclass
$ minikube --profile ${PROFILE} addons enable storage-provisioner
```

<br/>

### Качаем манифесты

```
$ cd ~/tmp
$ git clone https://github.com/wildmakaka/efk-setup
$ cd efk-setup
```

<br/>

### Elasticsearch

<br/>

```
$ cd elasticsearch
$ kubectl create -f statefulset.yaml
$ kubectl create -f service.yaml
```

<br/>

```
$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
es-cluster-0   1/1     Running   0          91s
es-cluster-1   1/1     Running   0          20s
es-cluster-2   1/1     Running   0          11s
```

<br/>

```
$ kubectl port-forward svc/elasticsearch 9200
```

<br/>

```
// Получить инфу по elastic
$ curl http://localhost:9200
{
  "name" : "es-cluster-0",
  "cluster_name" : "k8s-logs",
  "cluster_uuid" : "RdKE7J6wTXqI2C_uPLLUeQ",
  "version" : {
    "number" : "7.14.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "dd5a0a2acaa2045ff9624f3729fc8a6f40835aa1",
    "build_date" : "2021-07-29T20:49:32.864135063Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

<br/>

```
// Получить инфу по статусу работы elastic
$ curl localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "k8s-logs",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 0.0
}
```

<br/>

### Kibana

```
$ cd kibana
$ kubectl create -f deployment.yaml
$ kubectl create -f service.yaml
```

<br/>

```
$ kubectl port-forward svc/kibana 8080
```

<br/>

```
[OK!]
$ curl localhost:8080
```

<br/>

### Fluentd

```
$ cd fluentd
$ kubectl create -f clusterrole.yaml
$ kubectl create -f serviceaccount.yaml
$ kubectl create -f clusterrolebinding.yaml
$ kubectl create -f daemonset.yaml
```
