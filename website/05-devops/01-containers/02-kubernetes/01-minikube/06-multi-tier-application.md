---
layout: page
title: Разворачиваем многоуровневое приложение в minikube
description: Разворачиваем многоуровневое приложение в minikubes
keywords: devops, linux, kubernetes,  Разворачиваем многоуровневое приложение в minikubes
permalink: /devops/containers/kubernetes/minikube/multi-tier-application/
---

# Разворачиваем многоуровневое приложение в minikube

Оригинальное название: "Kubernetes - A Multi-Tier Application"

Делаю  
29.03.2019

Прилжение работает!  
Но разворачивается минут 10.

<br/>

https://www.youtube.com/watch?time_continue=1&v=Tywdpr3tWLo

In a typical application, we have different tiers:

-   Backend
-   Frontend
-   Caching, etc.

In this chapter, we will learn to deploy a multi-tier application with Kubernetes and then scale it.

App src:

https://github.com/cloudyuga/rsvpapp

<br/>

    $ mkdir ~/kubernetes-minikube && cd ~/kubernetes-minikube

<br/>

    $ vi rsvp-db.yaml

<br/>

```yaml1
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rsvp-db
  labels:
    appdb: rsvpdb
spec:
  replicas: 1
  selector:
    matchLabels:
      appdb: rsvpdb
  template:
    metadata:
      labels:
        appdb: rsvpdb
    spec:
      containers:
      - name: rsvp-db
        image: mongo:3.3
        ports:
        - containerPort: 27017

```

<br/>

    $ kubectl create -f rsvp-db.yaml

<br/>

### Create the Service for MongoDB

    $ vi rsvp-db-service.yaml

<br/>

```yaml1
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  labels:
    app: rsvpdb
spec:
  ports:
  - port: 27017
    protocol: TCP
  selector:
    appdb: rsvpdb
```

<br/>

    $ kubectl create -f rsvp-db-service.yaml

<br/>

As we did not specify any ServiceType, mongodb will have the default ClusterIP ServiceType. This means that the mongodb Service will not be accessible from the external world.

<br/>

    $ kubectl get deployments
    NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    rsvp-db     1         1         1            1           0s

<br/>

    $ kubectl get services
    NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          9d
    mongodb          ClusterIP   10.96.180.154    <none>        27017/TCP        5s

<br/>

The frontend is created using a Python Flask-based microframework. Its source code is available here. We have created a Docker image called teamcloudyuga/rsvpapp, in which we have imported the application's source code. The application's code is executed when a container created from that image runs. The Dockerfile to create the teamcloudyuga/rsvpapp image is available here.

Next, we will go through the steps of creating the rsvp frontend.

Docker image  
https://raw.githubusercontent.com/cloudyuga/rsvpapp/master/Dockerfile

https://hub.docker.com/r/teamcloudyuga/rsvpapp/

<br/>

### Create the Deployment for the 'rsvp' Frontend

    $ vi rsvp-web.yaml

<br/>

```yaml1
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rsvp
  labels:
    app: rsvp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rsvp
  template:
    metadata:
      labels:
        app: rsvp
    spec:
      containers:
      - name: rsvp-app
        image: teamcloudyuga/rsvpapp
        env:
        - name: MONGODB_HOST
          value: mongodb
        ports:
        - containerPort: 5000
          name: web-port
```

<br/>

    $ kubectl create -f rsvp-web.yaml

<br/>

While creating the Deployment for the frontend, we are passing the name of the MongoDB Service, mongodb, as an environment variable, which is expected by our frontend.

Notice that in the ports section we mentioned the containerPort 5000, and given it the web-port name. We will be using the referenced web-port name while creating the Service for the rsvp application. This is useful, as we can change the underlying containerPort without making any changes to our Service.

<br/>

    $ vi rsvp-web-service.yaml

<br/>

```yaml1
apiVersion: v1
kind: Service
metadata:
  name: rsvp
  labels:
    app: rsvp
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: web-port
    protocol: TCP
  selector:
    app: rsvp

```

<br/>

    $ kubectl create -f rsvp-web-service.yaml

<br/>

You may notice that we have mentioned the targetPort in the ports section, which will forward all the requests coming on port 80 for the ClusterIP to the referenced web-port port (5000) on the connected Pods. We can describe the Service and verify it.

<br/>

    $ kubectl get deployments
    NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    rsvp        1         1         1            1           5m
    rsvp-db     1         1         1            1           11m

<br/>

    $ kubectl get svc
    $ kubectl get services
    NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          9d
    mongodb          ClusterIP   10.96.180.154    <none>        27017/TCP        11m
    rsvp             NodePort    10.103.53.240    <none>        80:31024/TCP     4m

<br/>

    $ minikube ip
    192.168.99.100

<br/>
    
    $ minikube service rsvp

<br/>

### Scale the Frontend

Currently, we have one replica running for the frontend. To scale it to 4 replicas, we can use the following command:

<br/>

    $ kubectl scale deployment rsvp --replicas=3

<br/>

    $ kubectl get deployments
    NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    rsvp        3         3         3            3           10m
    rsvp-db     1         1         1            1           15m

<br/>

    $ kubectl get pods
    NAME                        READY     STATUS    RESTARTS   AGE
    rsvp-876876b6c-8tk6h        1/1       Running   0          16m
    rsvp-876876b6c-mwxlw        1/1       Running   0          16m
    rsvp-876876b6c-xf9j8        1/1       Running   0          27m
    rsvp-db-687d5b488d-6l6d7    1/1       Running   0          32m

<br/>

By F5 server is changing
