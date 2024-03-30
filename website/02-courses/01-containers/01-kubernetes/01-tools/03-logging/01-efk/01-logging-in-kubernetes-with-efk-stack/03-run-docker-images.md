---
layout: page
title: Logging in Kubernetes with EFK Stack | Запускаем image на kubernetes
description: Logging in Kubernetes with EFK Stack | Запускаем image на kubernetes
keywords: courses, containers, kubernetes, tools, logging, elastic, fluentd, kibana, Запускаем image на kubernetes
permalink: /courses/containers/kubernetes/tools/logging/efk/logging-in-kubernetes-with-efk-stack/run-docker-images/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack | The Complete Guide [ENG, 2021]: Запускаем image на kubernetes

<br/>

Делаю:  
2024.03.30

<br/>

### Create docker-registry secret for dockerHub

<br/>

```
$ export DOCKER_REGISTRY_SERVER=docker.io
$ export DOCKER_USER=<YOUR_DOCKERHUB_LOGIN>
$ export DOCKER_EMAIL=<YOUR_DOCKERHUB_LOGIN>
$ export DOCKER_PASSWORD=<YOUR_DOCKERHUB_PASSWORD>

$ kubectl create secret docker-registry myregistrysecret \
    --docker-server=${DOCKER_REGISTRY_SERVER} \
    --docker-username=${DOCKER_USER} \
    --docker-password=${DOCKER_PASSWORD} \
    --docker-email=${DOCKER_EMAIL}

$ kubectl get secret
```

<br/>

### Deploy

<br/>

**node-app**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
  labels:
    app: node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
        - name: node-app
          image: ${DOCKER_HUB_LOGIN}/node-1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 3000
EOF
```

<br/>

**java-app**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
  labels:
    app: java-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      containers:
        - name: java-app
          image: ${DOCKER_HUB_LOGIN}/java-1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
      imagePullSecrets:
        - name: myregistrysecret
EOF
```

<br/>

```
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
java-app-85b44765bb-rqlwk   1/1     Running   0          6s
node-app-6c87fddb75-wn285   1/1     Running   0          12s
```
