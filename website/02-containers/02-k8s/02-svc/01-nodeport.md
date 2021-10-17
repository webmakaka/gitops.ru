---
layout: page
title: Создание службы Nodeport
description: Создание службы Nodeport
keywords: devops, containers, kubernetes, minikube, Создание службы Nodeport
permalink: /containers/k8s/svc/nodeport/
---

# Создание службы Nodeport

Делаю:  
12.09.2021

<br/>

**Nodeport используют в разработке. В остальных случаях, рекомендуют использовать LoadBalancer.**

<br/>

**Запуск minikube**

Как <a href="/containers/k8s/setup/minikube/">здесь</a>

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-cats-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nodejs-cats-app
  template:
    metadata:
      labels:
        app: nodejs-cats-app
        env: dev
    spec:
      containers:
      - name: nodejs-cats-app
        image: webmakaka/cats-app
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
EOF
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nodejs-cats-app-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30123
  selector:
    app: nodejs-cats-app
EOF
```

<br/>

    port: 80 - хз для чего задаем. Наверняка можно заменить, чем-то более осмысленным.
    targetPort: 8080 - порт на котором работает приложение внутри pod.
    nodePort: 30123 - то к какому порту обращаться на этот под.

<br/>

    $ kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    nodejs-cats-app-774f89d47b-2tbrj   1/1     Running   0          61s
    nodejs-cats-app-774f89d47b-8hjrv   1/1     Running   0          61s
    nodejs-cats-app-774f89d47b-lwc85   1/1     Running   0          61s

<br/>

    $ kubectl get svc
    NAME                       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    nodejs-cats-app-nodeport   NodePort   10.109.227.32   <none>        80:30123/TCP   22s

<br/>

    // Если не используется профиль, удалить
    // Если не используется namespace, таке можно убрать -n default
    $ echo $(minikube --profile marley-minikube service nodejs-cats-app-nodeport -n default --url)

<br/>

    http://192.168.99.113:30123

<br/>

При обращении по адресу запустилось приложение.

<br/>

    // Если понадобится удалить
    $ kubectl delete svc nodejs-cats-app-nodeport
    $ kubectl delete deployment nodejs-cats-app
