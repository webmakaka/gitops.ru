---
layout: page
title: Yandex Clouds - Kubernetes
description: Yandex Clouds - Kubernetes
keywords: Deploy, Clouds, Yandex, Kubernetes
permalink: /tools/clouds/yandex/kubernetes/
---

# Yandex Clouds - Kubernetes

**Надо будет посмотреть:**  
https://www.youtube.com/watch?v=52k_cFxRZF4

<br/>

https://practicum.yandex.ru/trainer/ycloud/lesson/166f2f98-e773-4e2a-949e-9c8f61459b24/

<br/>

YANDEX CLOUD UI -> Managed Service for Kubernetes -> Создать

<br/>

Для Kubernetes необходим сервисный аккаунт для ресурсов и узлов.

Сервисный аккаунт для ресурсов — это аккаунт, под которым сервису Kubernetes будут выделяться ресурсы в нашем облаке.

Сервисный аккаунт для узлов необходим уже созданным узлам самого кластера Kubernetes для доступа к другим ресурсам. Например, чтобы получить Docker-образы из Container Registry.

Этим аккаунтам нужны разные права, и поэтому у них бывают разные роли. В общем случае вы можете использовать один и тот же сервисный аккаунт. Выберите аккаунт, который создали на первом курсе, или заведите новый.

<br/>

Управление узлами -> Создание группы узлов

<br/>

$ vi my-nginx.yaml

<br/>

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: cr.yandex/crpv7tlcpgb30qpgkiij/ubuntu-nginx:latest
```

<br/>

```
$ kubectl apply -f my-nginx.yaml
```

<br/>

$ vi load-balancer.yaml

<br/>

```
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
spec:
  selector:
     app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

<br/>

```
$ kubectl apply -f load-balancer.yaml
```

<br/>

YANDEX CLOUD UI -> Network Load Balancer
