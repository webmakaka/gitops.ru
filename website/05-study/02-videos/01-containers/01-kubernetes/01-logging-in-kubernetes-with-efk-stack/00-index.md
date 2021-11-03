---
layout: page
title: Logging in Kubernetes with EFK Stack | The Complete Guide
description: Logging in Kubernetes with EFK Stack | The Complete Guide
keywords: containers, kubernetes, linode, elastic, fluentd, kibana
permalink: /study/videos/containers/kubernetes/logging-in-kubernetes-with-efk-stack/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack | The Complete Guide [ENG, 2021]

<br/>

Делаю:  
04.11.2021

<br/>

**node app:**  
https://gitlab.com/nanuchi/node-app

<br/>

**java app**
https://gitlab.com/nanuchi/java-app

<br/>

**Промо на облачный kubernetes от linode на $100**  
https://gitlab.com/nanuchi/efk-course-commands/-/tree/master

<br/>

**Еще какие-то полезные ссылки**  
https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/links.md

<br/>

**Set up elastic stack in kubernetes cluster**  
https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/commands.md

<br/>

### Подключение к бесплатному облаку от Google

https://shell.cloud.google.com/

<br/>

**Инсталлим google-cloud-sdk**

https://cloud.google.com/sdk/docs/install

<br/>

```
$ gcloud auth login
$ gcloud cloud-shell ssh
```

<br/>

1. Инсталляция [MiniKube](/containers/kubernetes/setup/minikube/) (Ingress и остальное можно не устанавливать)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/containers/kubernetes/tools/kubectl/)

<br/>

### Подготавливаем образы и выкладываем на hub.docker.com

<br/>

Для меня образы в публичном регистри - норм.

<br/>

```
$ docker login
```

<br/>

**Приватные репо не нужны:**

<br/>

**node-app**

```
$ cd ~/tmp
$ git clone https://gitlab.com/nanuchi/node-app.git
$ cd node-app
$ docker build -t node-app .
$ docker tag node-app webmakaka/node-1.0:latest
$ docker push webmakaka/node-1.0
```

<br/>

**java-app**

```
$ cd ~/tmp
$ git clone https://gitlab.com/nanuchi/java-app.git
$ cd java-app
$ ./gradlew build
$ docker build -t java-app .
$ docker tag java-app webmakaka/java-1.0:latest
$ docker push webmakaka/java-1.0
```

<br/>

### Create docker-registry secret for dockerHub (Пропускаю)

Не нужно выполнять, если image хранятся в публичном registry.

<br/>

```
$ export DOCKER_REGISTRY_SERVER=docker.io
$ export DOCKER_USER=your dockerID, same as for `docker login`
$ export DOCKER_EMAIL=your dockerhub email, same as for `docker login`
$ export DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login`

$ kubectl create secret docker-registry myregistrysecret \
--docker-server=${DOCKER_REGISTRY_SERVER} \
--docker-username=${DOCKER_USER} \
--docker-password=${DOCKER_PASSWORD} \
--docker-email=${DOCKER_EMAIL}

$ kubectl get secret
```

<br/>

### Deploy

Оригинальные лежат в репо проектов.

<br/>

**node-app**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
          image: webmakaka/node-1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 3000
EOF
```

<br/>

**java-app**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
          image: webmakaka/java-1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
EOF
```

<br/>

```
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
java-app-85b44765bb-rqlwk   1/1     Running   0          6s
node-app-6c87fddb75-wn285   1/1     Running   0          12s
```

<br/>

**Инсталляция [helm](/containers/kubernetes/tools/helm/setup/)**

<br/>

**Инсталляция [Elastic Search, Kibana, Fluentd](/containers/kubernetes/tools/helm/)**

<br/>

**kibana-ingress**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: kibana-ingress
spec:
  rules:
    - host: nb-172-104-255-70.frankfurt.nodebalancer.linode.com
      http:
        paths:
          - path: /
            backend:
              serviceName: kibana-kibana
              servicePort: 5601
EOF
```
