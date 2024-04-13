---
layout: page
title: Logging in Kubernetes with EFK Stack | The Complete Guide
description: Logging in Kubernetes with EFK Stack | The Complete Guide
keywords: courses, containers, kubernetes, tools, logging, elastic, fluentd, kibana
permalink: /courses/containers/kubernetes/tools/logging/efk/logging-in-kubernetes-with-efk-stack/fluentd/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack | The Complete Guide [ENG, 2021]

<br/>

Делаю:  
2024.04.13

<br/>

# Настройка Fluentd

```
$ kubectl --namespace logging get configmaps
NAME                    DATA   AGE
fluentd-aggregator-cm   4      7m24s
fluentd-forwarder-cm    4      7m24s
kube-root-ca.crt        1      8m12s
```

<br/>

```
$ kubectl --namespace logging get pods
$ kubectl --namespace logging logs fluentd-zn6bx
```

<br/>

```
$ kubectl --namespace logging logs fluentd-zn6bx | grep node-app
2021-11-05 12:59:47 +0000 [info]: #0 following tail of /var/log/tools/containers/node-app-6c87fddb75-5nbrr_default_node-app-2a757252113980b62c449d735845abb36aea92546273c8740fa07e151f9ddc2f.log

$ kubectl --namespace logging logs fluentd-zn6bx | grep java-app
2021-11-05 12:59:47 +0000 [info]: #0 following tail of /var/log/tools/containers/java-app-85b44765bb-pb8d2_default_java-app-2e037cfd5231ac26fc642b1254953cd5be078c11b796f4653252f8b953573a7a.log
```

<br/>

```
$ kubectl --namespace logging describe configmap fluentd-forwarder-cm > ~/tmp/fluentd-forwarder-cm.backup.txt
$ kubectl --namespace logging edit configmap fluentd-forwarder-cm
```

<br/>

**Отключаем неинтересные для задач курса инструкции**

```
    # Throw the healthcheck to the standard output instead of forwarding it
    <match fluent.**>
        @type null
    </match>
```

<br/>

**Отслеживаем логи только наших приложений -app**

```
    # Get the logs from the containers running in the node
    <source>
      @type tail
      path /var/log/tools/containers/*-app*.log
      pos_file /opt/bitnami/fluentd/logs/buffers/fluentd-docker.pos
      tag kubernetes.*
      read_from_head true
      format json
      time_format %Y-%m-%dT%H:%M:%S.%NZ
    </source>
```

<br/>

**Чтобы логи отображались более наглядно**

```
    <filter **>
      @type parser
      key_name log
      <parse>
        @type multi_format
        <pattern>
          format json
          time_key time
          keep_time_key true
        </pattern>
      </parse>
    </filter>
```

<br/>

**Определяем что отправлять аггрегаторам -app**

```
    # Forward all logs to the aggregators
    <match kubernetes.var.log.containers.**java-app**.log>
      @type elasticsearch
      include_tag_key true
      host "elasticsearch-master.logging.svc.cluster.local"
      port "9200"
      index_name "java-app-logs"
      <buffer>
        @type file
        path /opt/bitnami/fluentd/logs/buffers/java-logs.buffer
        flush_thread_count 2
        flush_interval 5s
      </buffer>
    </match>

    <match kubernetes.var.log.containers.**node-app**.log>
      @type elasticsearch
      include_tag_key true
      host "elasticsearch-master.logging.svc.cluster.local"
      port "9200"
      index_name "node-app-logs"
      <buffer>
        @type file
        path /opt/bitnami/fluentd/logs/buffers/node-logs.buffer
        flush_thread_count 2
        flush_interval 5s
      </buffer>
    </match>
```

<br/>

```yaml
***
data:
    fluentd.conf: |

        # Ignore fluentd own events
        <match fluent.**>
            @type null
        </match>

        # HTTP input for the liveness and readiness probes
        <source>
            @type http
            port 9880
        </source>

        # Throw the healthcheck to the standard output instead of forwarding it
        <match fluentd.healthcheck>
            @type null
        </match>

        # Get the logs from the containers running in the node
        <source>
          @type tail
          path /var/log/tools/containers/*-app*.log
          pos_file /opt/bitnami/fluentd/logs/buffers/fluentd-docker.pos
          tag kubernetes.*
          read_from_head true
          format json
          time_format %Y-%m-%dT%H:%M:%S.%NZ
        </source>

        <filter **>
          @type parser
          key_name log
          <parse>
            @type multi_format
            <pattern>
              format json
              time_key time
              keep_time_key true
            </pattern>
          </parse>
        </filter>

        # enrich with kubernetes metadata
        <filter kubernetes.**>
            @type kubernetes_metadata
        </filter>


        <match kubernetes.var.log.containers.**java-app**.log>
          @type elasticsearch
          include_tag_key true
          host "elasticsearch-master.logging.svc.cluster.local"
          port "9200"
          index_name "java-app-logs"
          <buffer>
            @type file
            path /opt/bitnami/fluentd/logs/buffers/java-logs.buffer
            flush_thread_count 2
            flush_interval 5s
          </buffer>
        </match>

        <match kubernetes.var.log.containers.**node-app**.log>
          @type elasticsearch
          include_tag_key true
          host "elasticsearch-master.logging.svc.cluster.local"
          port "9200"
          index_name "node-app-logs"
          <buffer>
            @type file
            path /opt/bitnami/fluentd/logs/buffers/node-logs.buffer
            flush_thread_count 2
            flush_interval 5s
          </buffer>
        </match>
```

<br/>

```
$ kubectl --namespace logging \
  rollout restart daemonset/fluentd
```

<br/>

Enterprise Search ->

Menu -> Analitics -> discover

Data -> Index Management

должны быть: node-app-logs и java-app-logs в состоянии green

<br/>

Kibana -> Index Patterns -> Create Index pattern

Index pattern name -> **_*app*_** -> Next step

<br/>

Fime field -> time

Create index pattern

<br/>

Kibana -> Discovery
