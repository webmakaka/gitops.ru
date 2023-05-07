---
layout: page
title: Kubernetes Deployments
description: Kubernetes Deployments
keywords: devops, linux, kubernetes, Kubernetes Deployments
permalink: /devops/containers/kubernetes/basics/deployments/
---

# Kubernetes Deployments

Делаю: 21.04.2020

https://github.com/burrsutter/9stepsawesome/blob/master/2_building_running.adoc

<br/>

### Запускаем Deployments

```
$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quarkus-demo-deployment
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
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
EOF
```

<br/>

    $ kubectl get pods --show-labels
    NAME                                       READY   STATUS    RESTARTS   AGE     LABELS
    quarkus-demo-deployment-5979886fb7-4xbpb   1/1     Running   0          2m29s   app=quarkus-demo,env=dev,pod-template-hash=5979886fb7
    quarkus-demo-deployment-5979886fb7-nrptt   1/1     Running   0          2m29s   app=quarkus-demo,env=dev,pod-template-hash=5979886fb7
    quarkus-demo-deployment-5979886fb7-ntrpd   1/1     Running   0          2m29s   app=quarkus-demo,env=dev,pod-template-hash=5979886fb7

<br/>

    $ kubectl get deployments
    NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
    quarkus-demo-deployment   3/3     3            3           3m24s

<br/>

    $ kubectl get replicaset
    NAME                                 DESIRED   CURRENT   READY   AGE
    quarkus-demo-deployment-5979886fb7   3         3         3       3m52s

<br/>

    $ kubectl scale deploy quarkus-demo-deployment --replicas=6

<br/>

    $ kubectl delete deploy quarkus-demo-deployment
