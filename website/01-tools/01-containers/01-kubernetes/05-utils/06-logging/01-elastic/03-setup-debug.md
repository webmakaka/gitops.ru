---
layout: page
title: Инсталляция инструментов логирования Elastic - Elasticsearch, Kibana, Fluentd
description: Инсталляция инструментов логирования Elastic - Elasticsearch, Kibana, Fluentd
keywords: gitops, containers, logging, elastic search, kibana, fluentd
permalink: /tools/containers/kubernetes/utils/logging/elastic/setup/debug/
---

# Инсталляция инструментов логирования Elastic - Elasticsearch, Kibana, Fluentd

<br/>

Делаю:  
2024.04.13

<br/>

### Elasticsearch

<br/>

```
$ kubectl port-forward svc/elasticsearch 9200
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

A red status means one or more primary shards are unassigned.

<br/>

**View unassigned shards**

```
$ curl -XGET localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason | grep UNASSIGNED
```

<br/>

```
// Что вообще тут просиходит
$ curl -XGET localhost:9200/\_cluster/allocation/explain?pretty
```

<br/>

```
// Установить максимальное значение параметра необходимого свободного места на диске в 90%
$ curl -XPUT "localhost:9200/\_cluster/settings" -H 'Content-Type: application/json' -d'
{
"transient": {
"cluster.routing.allocation.disk.watermark.low": "90%"
}
}'
```

<br/>

```
// Отправить на починку. Может само поднимется
$ curl -XPOST 'localhost:9200/\_cluster/reroute?retry_failed' | jq
```

<br/>

```
// Если не помогло
https://elasticsearch-ru.github.io/faq/bad-cluster-status.html
```

<br/>

Заодно, посмотреть

```
$ kubectl get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS VOLUMEATTRIBUTESCLASS REASON AGE
pvc-281d88e2-4eeb-44db-a06a-cb239cf7bea0 3Gi RWO Delete Bound default/data-es-cluster-0 standard <unset> 108s
pvc-527fda76-f9a6-46f7-aaad-0641f3dc5863 3Gi RWO Delete Bound default/data-es-cluster-2 standard <unset> 25s
pvc-ccdaa703-8f54-405c-ae8f-48b36098e100 3Gi RWO Delete Bound default/data-es-cluster-1 standard <unset> 37s
```

<br/>

```
$ kubectl get pvc
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS VOLUMEATTRIBUTESCLASS AGE
data-es-cluster-0 Bound pvc-281d88e2-4eeb-44db-a06a-cb239cf7bea0 3Gi RWO standard <unset> 2m22s
data-es-cluster-1 Bound pvc-ccdaa703-8f54-405c-ae8f-48b36098e100 3Gi RWO standard <unset> 71s
data-es-cluster-2 Bound pvc-527fda76-f9a6-46f7-aaad-0641f3dc5863 3Gi RWO standard <unset> 58s
```

<br/>

### Kibana

<br/>

```
$ kubectl port-forward svc/kibana 8080
```

<br/>

```
$ curl localhost:8080
Kibana server is not ready yet
```

<br/>

Ждем минут 5. Повторяем. Если не работает, смотрим статус elastic сервера.
