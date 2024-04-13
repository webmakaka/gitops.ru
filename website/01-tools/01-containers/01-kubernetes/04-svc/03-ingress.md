---
layout: page
title: Создание службы Ingress и ClusterIP
description: Создание службы Ingress и ClusterIP
keywords: devops, containers, kubernetes, minikube, ingress, clusterip, Создание службы Ingress и ClusterIP
permalink: /tools/containers/kubernetes/svc/ingress/
---

# Создание службы Ingress и ClusterIP

Делаю:  
21.11.2021

<br/>

Блин, все меняется от версии к версии, не успеваю конфиги переписывать!
Последний раз когда копал, оказалось, что работает со следующими аннотациями, что у меня в примере, а с теми, что в примерах на оф. сайте с версией v1.22.1 не работает. (При обращение по адресу nodejs-cats-app.example.com page-not-found).
Какие аннотации обязательные, какие нет, сейчас сказать не могу.

<br/>

Deployment уже создан как <a href="/tools/containers/kubernetes/svc/nodeport/">здесь</a>

<br/>

**Создаю сервис ClusterIP**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nodejs-cats-app-cluster-ip
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: nodejs-cats-app
EOF
```

<br/>

### Включение функционала Ingress в Minikube

<br/>

```
$ minikube --profile marley-minikube addons list
- ingress: disabled

// Включение функционала Ingress в Minikube
$ minikube addons --profile marley-minikube enable ingress

$ kubectl get po --all-namespaces | grep ingress-nginx-controller
ingress-nginx          ingress-nginx-controller-69bdbc4d57-zxn77    1/1     Running     0             45m

$ kubectl get pods -n ingress-nginx
```

<br/>

```
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create--1-xk4ww     0/1     Completed   0          76m
ingress-nginx-admission-patch--1-skvq6      0/1     Completed   1          76m
ingress-nginx-controller-69bdbc4d57-zxn77   1/1     Running     0          76m
```

<br/>

### Запускаем приложение

<br/>

```
$ kubectl get service nodejs-cats-app-cluster-ip
NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nodejs-cats-app-cluster-ip   ClusterIP   10.101.76.157   <none>        80/TCP    21s
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nodejs-cats-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/default-backend: ingress-nginx-controller
    kubernetes.io/ingress.class: nginx
    ## tells ingress to check for regex in the config file
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
  - host: nodejs-cats-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nodejs-cats-app-cluster-ip
            port:
              number: 80
EOF
```

<!-- <br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nodejs-cats-app-ingress
spec:
  defaultBackend:
    service:
      name: nodejs-cats-app-cluster-ip
      port:
        number: 80
EOF
```
-->

<br/>

```
$ kubectl get ingresses
NAME                      HOSTS                         ADDRESS   PORTS   AGE
nodejs-cats-app-ingress   nodejs-cats-app.example.com             80      23s
```

<br/>

```
// Для minikube
$ minikube --profile marley-minikube ip
192.168.49.2

// В других случая прописать IP из ADDRESS
```

<br/>

```
$ sudo vi /etc/hosts

192.168.49.2 nodejs-cats-app.example.com
```

<br/>

**Подключаемся по адресу:**

http://nodejs-cats-app.example.com

<br/>

**OK!**

<br/>

```
// Удалить ingress
$ kubectl delete ingress nodejs-cats-app-ingress

// И остальное
$ kubectl delete svc nodejs-cats-app-cluster-ip
$ kubectl delete deployment nodejs-cats-app
```

<br/>

### Debug

```
$ kubectl describe ingress nodejs-cats-app-ingress
```

<br/>

### Дополнительно

https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

<br/>

### Тоже самое но с использованием nip.io

<br/>

nip.io позволит не заморачиваться на добавлении всяких DNS, hosts etc.

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nodejs-cats-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/default-backend: ingress-nginx-controller
    kubernetes.io/ingress.class: nginx
    ## tells ingress to check for regex in the config file
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
  - host: 192.168.49.2.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nodejs-cats-app-cluster-ip
            port:
              number: 80
EOF
```

<br/>

```
$ kubectl get ingresses
NAME                      CLASS    HOSTS                 ADDRESS   PORTS   AGE
nodejs-cats-app-ingress   <none>   192.168.49.2.nip.io             80      8s
```

<br/>

```
// [OK!]
http://192.168.49.2.nip.io/
```
