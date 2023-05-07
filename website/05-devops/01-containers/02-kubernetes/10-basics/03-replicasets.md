---
layout: page
title: Kubernetes ReplicaSets
description: Kubernetes ReplicaSets
keywords: devops, linux, kubernetes, Kubernetes ReplicaSets
permalink: /devops/containers/kubernetes/basics/replicasets/
---

# Kubernetes ReplicaSets - считается устаревшей. Рекомендуется использовать Deployments

Делаю: 21.04.2020

https://github.com/burrsutter/9stepsawesome/blob/master/2_building_running.adoc

<br/>

### Запускаем ReplicaSets

```
$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: rs-quarkus-demo
spec:
    replicas: 3
    selector:
       matchLabels:
          app: quarkus-demo
    template:
       metadata:
          labels:
             app: quarkus-demo
             env: dev
       spec:
          containers:
          - name: quarkus-demo
            image: quay.io/burrsutter/quarkus-demo:1.0.0
EOF
```

<br/>

    // $ kubectl get pods --show-labels
    $ kubectl get pods
    NAME                    READY   STATUS    RESTARTS   AGE
    rs-quarkus-demo-62z29   1/1     Running   0          7s
    rs-quarkus-demo-d67vd   1/1     Running   0          7s
    rs-quarkus-demo-pgqds   1/1     Running   0          7s

<br/>

    $ kubectl delete pod rs-quarkus-demo-62z29

<br/>

    // $ kubectl get replicasets
    $ kubectl get rs
    NAME              DESIRED   CURRENT   READY   AGE
    rs-quarkus-demo   3         3         3       38s

<br/>

    $ kubectl delete replicaset rs-quarkus-demo
