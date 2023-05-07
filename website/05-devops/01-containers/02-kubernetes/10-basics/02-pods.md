---
layout: page
title: Kubernetes Pods
description: Kubernetes Pods
keywords: devops, linux, kubernetes, Kubernetes Pods
permalink: /devops/containers/kubernetes/basics/pods/
---

# Kubernetes Pods

Делаю: 21.04.2020

https://github.com/burrsutter/9stepsawesome/blob/master/2_building_running.adoc

<br/>

### Запускаем Pod

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: quarkus-demo
spec:
  containers:
  - name: quarkus-demo
    image: quay.io/burrsutter/quarkus-demo:1.0.0
EOF
```

<br/>

    $ kubectl get events

<br/>

    $ kubectl get pods
    NAME           READY   STATUS    RESTARTS   AGE
    quarkus-demo   1/1     Running   0          72s

<br/>

    $ kubectl describe pod quarkus-demo

<br/>

    $ kubectl get pod quarkus-demo -o yaml
    $ kubectl get pod quarkus-demo -o json

<br/>

    $ kubectl delete pod quarkus-demo
