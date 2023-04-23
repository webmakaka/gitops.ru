---
layout: page
title: Создание службы Nodeport
description: Создание службы Nodeport
keywords: devops, containers, kubernetes, minikube, Создание службы Nodeport
permalink: /tools/containers/kubernetes/svc/nodeport/
---

# Создание службы Nodeport

Делаю:  
14.11.2021

<br/>

**Nodeport используют в разработке. В остальных случаях, рекомендуют использовать LoadBalancer.**

<br/>

**Запуск minikube**

Как <a href="/tools/containers/kubernetes/minikube/setup/">здесь</a>

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

```
port: 80 - Внутренний порт для для коммуникаций между контейнерами (Надеюсь я ничего не перепутал!).
targetPort: 8080 - порт на котором работает приложение внутри pod.
nodePort: 30123 - то к какому порту обращаться на этот под.
```

<br/>

```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
nodejs-cats-app-56cc7754f-8kbrq   1/1     Running   0          3m22s
nodejs-cats-app-56cc7754f-vnlrl   1/1     Running   0          3m22s
nodejs-cats-app-56cc7754f-z26sm   1/1     Running   0          3m22s
```

<br/>

```
$ kubectl get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP        10m
nodejs-cats-app-nodeport   NodePort    10.109.225.154   <none>        80:30123/TCP   3m3s

```

<br/>

```
// Если не используется профиль, удалить
// Если не используется namespace, таке можно убрать -n default
$ echo $(minikube --profile ${PROFILE} service nodejs-cats-app-nodeport -n default --url)
```

<br/>

```
http://192.168.49.2:30123
```

<br/>

При обращении по адресу запустилось приложение.

<br/>

### UPD по поводу порт 80 в Service

```
$ kubectl exec -it nodejs-cats-app-56cc7754f-8kbrq -- sh
```

<br/>

```
# apk add curl
```

<br/>

```
# curl nodejs-cats-app-nodeport:80
```

<br/>

Возвращает страницу

<br/>

```
# apk add curl
```

<br/>

### Удалить созданное

<br/>

    // Если понадобится удалить
    $ kubectl delete svc nodejs-cats-app-nodeport
    $ kubectl delete deployment nodejs-cats-app
