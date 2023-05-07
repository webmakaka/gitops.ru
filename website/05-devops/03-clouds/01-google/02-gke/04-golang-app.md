---
layout: page
title: Запуск приложения в GKE (Load Balancer)
description: Запуск приложения в GKE (Load Balancer)
keywords: Запуск приложения в GKE (Load Balancer)
permalink: /devops/clouds/google/gke/google/golang-app/
---

# Запуск приложения в GKE (Load Balancer)

<br/>

### Подготавливаем kubernetes кластер

```
$ gcloud container clusters create echo-cluster \
    --num-nodes 2 \
    --machine-type n1-standard-2 \
    --zone us-central1-a
```

<br/>

### Скопировать приложение

    $ mkdir project && cd project
    $ gsutil cp gs://qwiklabs-gcp-03-8922069ccfc7/echo-web.tar.gz .
    $ tar -xvzpf echo-web.tar.gz ./

<br>

### Создаем имидж и отправляем его в GCR

    $ export PROJECT_ID=$(gcloud config get-value project)
    $ echo ${PROJECT_ID}

    $ docker build -t gcr.io/${PROJECT_ID}/echo-app:v1 .
    $ gcloud docker -- push gcr.io/${PROJECT_ID}/echo-app:v1

<!--

  $ gcloud container clusters get-credentials echo-cluster

-->

<br/>

### Запускаем приложение в kubernetes кластере

    $ vi echo-web-app-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-web
spec:
  replicas: 3
  selector:
    matchLabels:
      component: echo-web-app
  template:
    metadata:
      labels:
        component: echo-web-app
    spec:
      containers:
        - name: echo-web-app
          image: gcr.io/qwiklabs-gcp-03-8922069ccfc7/echo-app:v1
          ports:
            - containerPort: 8000
```

<br/>

    $ kubectl apply -f echo-web-app-deployment.yaml

<br/>

    $ kubectl expose deployment echo-web --port 80 --target-port 8000 --type="LoadBalancer"

<br/>

    $ kubectl get services

<br/>

http://35.222.220.63:80/
