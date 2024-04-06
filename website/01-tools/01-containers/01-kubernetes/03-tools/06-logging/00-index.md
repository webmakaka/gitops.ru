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
2024.04.06

**Из за бана РФ, ничего не работает с российских IP**

<br/>

**YouTube**  
https://www.youtube.com/watch?v=fb3CKSJTV-Q

**GIT**  
https://github.com/Bhoopesh123/efk-setup

<br/>

**Можно заюзать image:**

<br/>

```
webmakaka/elasticsearch:7.14.0
```

```
webmakaka/kibana:7.14.0
```

<br/>

### Предустановки

```
// Не знаю зачем могут требоваться
$ minikube --profile ${PROFILE} addons enable default-storageclass
$ minikube --profile ${PROFILE} addons enable storage-provisioner
```

<br/>

### Качаем манифесты

```
$ cd ~/tmp
$ git clone https://github.com/Bhoopesh123/efk-setup
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

### Elelasticsearch Debug

<br/>

```
$ curl http://localhost:9200
{
  "name" : "es-cluster-0",
  "cluster_name" : "k8s-logs",
  "cluster_uuid" : "ca__gfA_RH6Dq9P1nsW8Ww",
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
  "unassigned_shards" : 6,
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

<br/>

**View unassigned shards**

```
$ curl -XGET localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason| grep UNASSIGNED
```

<br/>

```
.geoip_databases 0 r UNASSIGNED REPLICA_ADDED
.geoip_databases 0 p UNASSIGNED INDEX_CREATED
```

<br/>

```
$ curl -XGET localhost:9200/_cluster/allocation/explain?pretty
```

<br/>

```json
{
  "index": ".geoip_databases",
  "shard": 0,
  "primary": false,
  "current_state": "unassigned",
  "unassigned_info": {
    "reason": "REPLICA_ADDED",
    "at": "2024-04-06T18:51:46.758Z",
    "last_allocation_status": "no_attempt"
  },
  "can_allocate": "no",
  "allocate_explanation": "cannot allocate because allocation is not permitted to any of the nodes",
  "node_allocation_decisions": [
    {
      "node_id": "MNuO4hSJRsW1HOvXUD45HA",
      "node_name": "es-cluster-2",
      "transport_address": "10.244.0.8:9300",
      "node_attributes": {
        "ml.machine_memory": "16143712256",
        "ml.max_open_jobs": "512",
        "xpack.installed": "true",
        "ml.max_jvm_size": "536870912",
        "transform.node": "true"
      },
      "node_decision": "no",
      "deciders": [
        {
          "decider": "replica_after_primary_active",
          "decision": "NO",
          "explanation": "primary shard for this replica is not yet active"
        },
        {
          "decider": "disk_threshold",
          "decision": "NO",
          "explanation": "the node is above the low watermark cluster setting [cluster.routing.allocation.disk.watermark.low=85%], using more disk space than the maximum allowed [85.0%], actual free: [8.19771138621111%]"
        },
        {
          "decider": "throttling",
          "decision": "NO",
          "explanation": "primary shard for this replica is not yet active"
        }
      ]
    },
    {
      "node_id": "UHXFgRJzTLiRpOK-idLP8A",
      "node_name": "es-cluster-1",
      "transport_address": "10.244.0.7:9300",
      "node_attributes": {
        "ml.machine_memory": "16143712256",
        "ml.max_open_jobs": "512",
        "xpack.installed": "true",
        "ml.max_jvm_size": "536870912",
        "transform.node": "true"
      },
      "node_decision": "no",
      "deciders": [
        {
          "decider": "replica_after_primary_active",
          "decision": "NO",
          "explanation": "primary shard for this replica is not yet active"
        },
        {
          "decider": "disk_threshold",
          "decision": "NO",
          "explanation": "the node is above the low watermark cluster setting [cluster.routing.allocation.disk.watermark.low=85%], using more disk space than the maximum allowed [85.0%], actual free: [8.19771138621111%]"
        },
        {
          "decider": "throttling",
          "decision": "NO",
          "explanation": "primary shard for this replica is not yet active"
        }
      ]
    },
    {
      "node_id": "UgoC8ZK-RyW7uEGiqGJIEg",
      "node_name": "es-cluster-0",
      "transport_address": "10.244.0.6:9300",
      "node_attributes": {
        "ml.machine_memory": "16143712256",
        "ml.max_open_jobs": "512",
        "xpack.installed": "true",
        "ml.max_jvm_size": "536870912",
        "transform.node": "true"
      },
      "node_decision": "no",
      "deciders": [
        {
          "decider": "replica_after_primary_active",
          "decision": "NO",
          "explanation": "primary shard for this replica is not yet active"
        },
        {
          "decider": "disk_threshold",
          "decision": "NO",
          "explanation": "the node is above the low watermark cluster setting [cluster.routing.allocation.disk.watermark.low=85%], using more disk space than the maximum allowed [85.0%], actual free: [8.19771138621111%]"
        },
        {
          "decider": "throttling",
          "decision": "NO",
          "explanation": "primary shard for this replica is not yet active"
        }
      ]
    }
  ]
}
```

<br/>

```
$ curl -XPUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "90%"
  }
}'
```

<br/>

Я так понял, есть 2 варианта:

**Solutions**

1. Add more nodes to your cluster, so that replicas can be assigned on other nodes. (preferred way)

2. Reduce the replica shards to 0, this can cause data-loss and performance issues. (if at all, you don't have the option to add data-nodes and you want the green state for your cluster).

   You can update the replica counts using cluster update API.

https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html#indices-update-settings

<br/>

```
// Не работает!
// UPDATE
// PUT
$ curl \
    --data '{
  "template": "*",
  "settings": {
    "number_of_replicas": 0
  }
}' \
    --header "Content-Type: application/json" \
    --request PUT \
    --url localhost:9200/_template/everything_template \
    | jq
```

<br/>

```
// Не работает!
// UPDATE
// PUT
$ curl \
    --data '{
  "template": "*",
  "settings": {
    "number_of_replicas": 0
  }
}' \
    --header "Content-Type: application/json" \
    --request PUT \
    --url localhost:9200/_settings \
    | jq
```

<br/>

```
// Не работает!
// UPDATE
// PUT
$ curl \
    --data '{
     "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
  }' \
    --header "Content-Type: application/json" \
    --request PUT \
    --url localhost:9200/_cluster/settings \
    | jq
```

<br/>

```
$ curl -XPOST 'localhost:9200/_cluster/reroute?retry_failed' | jq
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
$ curl localhost:8080
Kibana server is not ready yet
```

<br/>

```
// UPDATE
// PUT
$ curl \
    --data '{
  "template": ".kibana_7.14.0_001",
  "settings": {
    "number_of_replicas": 0
  }
}' \
    --header "Content-Type: application/json" \
    --request PUT \
    --url localhost:9200/_template/everything_template \
    | jq
```

<br/>

```
// UPDATE
// PUT
$ curl \
    --data '{
  "template": ".kibana_task_manager_7.14.0_001",
  "settings": {
    "number_of_replicas": 0
  }
}' \
    --header "Content-Type: application/json" \
    --request PUT \
    --url localhost:9200/_template/everything_template \
    | jq
```

<br/>

```
$ curl -XDELETE 'localhost:9200/.kibana_7.14.0_001/'
$ curl -XDELETE 'localhost:9200/.kibana_task_manager_7.14.0_001/'
```

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

https://www.datadoghq.com/blog/elasticsearch-unassigned-shards/

<br/>

```
// + Как-то нужно вырубить
"ingest.geoip.downloader.enabled: false"
```
