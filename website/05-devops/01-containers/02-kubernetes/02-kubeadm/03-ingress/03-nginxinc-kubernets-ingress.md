---
layout: page
title: Запускаем приложение с использованием NginxInc Kubernetes Ingress
description: Запускаем приложение с использованием NginxInc Kubernetes Ingress
keywords: devops, linux, kubernetes, Запускаем приложение с использованием NginxInc Kubernetes Ingress
permalink: /devops/containers/kubernetes/kubeadm/ingress/nginxinc-kubernets-ingress/
---

# Запускаем приложение с использованием NginxInc Kubernetes Ingress

Делаю  
04.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=YzaYqxW0wGs&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0

<br/>

![kubernetes ingress](/img/devops/containers/kubernetes/kubeadm/ingress/ingress.png 'kubernetes ingress'){: .center-image }

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

Установили и настроили HAProxy как <a href="/devops/containers/kubernetes/kubeadm/ingress/haproxy/">здесь</a>.

<br/>

### [Создаем NginxInc Kubernetes Ingress контроллер](/devops/containers/kubernetes/kubeadm/ingress/nginxinc-kubernets-ingress-install/)

<br/>

### Запускаем приложение

<br/>

    $ kubectl create -f https://raw.githubusercontent.com/justmeandopensource/kubernetes/master/yamls/ingress-demo/nginx-deploy-blue.yaml

    $ kubectl create -f https://raw.githubusercontent.com/justmeandopensource/kubernetes/master/yamls/ingress-demo/nginx-deploy-green.yaml

    $ kubectl create -f https://raw.githubusercontent.com/justmeandopensource/kubernetes/master/yamls/ingress-demo/ingress-resource-2.yaml

    $ kubectl expose deploy nginx-deploy-blue --port 80

    $ kubectl expose deploy nginx-deploy-green --port 80

    $ kubectl describe ing ingress-resource-2

    $ kubectl get pods
    NAME                                 READY   STATUS    RESTARTS   AGE
    nginx-deploy-blue-7cc7d854dc-9hzwf   1/1     Running   0          3m5s
    nginx-deploy-green-fbbd6d8d8-2nkzb   1/1     Running   0          3m

<br/>

### Настройка на клиенте

<br/>

    # vi /etc/hosts

    192.168.0.5 blue.nginx.example.com
    192.168.0.5 green.nginx.example.com

<br/>

    $ curl blue.nginx.example.com
    <h1>I am <font color=blue>BLUE</font></h1>

    $ curl green.nginx.example.com
    <h1>I am <font color=green>GREEN</font></h1>

<br/>

### Удаляем все это добро:

    $ kubectl delete ing ingress-resource-2

    $ kubectl delete deployment nginx-deploy-blue
    $ kubectl delete deployment nginx-deploy-green

    $ kubectl delete service nginx-deploy-blue
    $ kubectl delete service nginx-deploy-green

<br/>

## Попробуем запустить свое приложение:

    $ mkdir ~/tmp/cats-app/ && cd ~/tmp/cats-app/

<br/>

    # vi nodejs-cats-app-deployment.yaml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nodejs-cats-app-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nodejs-cats-app
    spec:
      containers:
      - name: nodejs-cats-app
        image: webmakaka/cats-app
```

<br/>

    $ vi nodejs-cats-app-svc-nodeport.yaml

```
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
```

<br/>

    # vi ingress-resource-nodejs-cats-app.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nodejs-cats-app-ingress
spec:
  rules:
  - host: nodejs-cats-app.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nodejs-cats-app-nodeport
          servicePort: 80
```

<br/>

    $ kubectl create -f nodejs-cats-app-deployment.yaml
    $ kubectl create -f nodejs-cats-app-svc-nodeport.yaml
    $ kubectl create -f ingress-resource-nodejs-cats-app.yaml

<!--
 # kubectl expose deploy nodejs-cats-app-deployment --port 80

 -->

<br/>

    $ kubectl get pods
    NAME                                          READY   STATUS    RESTARTS   AGE
    nodejs-cats-app-deployment-5d67fbc67d-2bmxz   1/1     Running   0          44m
    nodejs-cats-app-deployment-5d67fbc67d-4jkz2   1/1     Running   0          34m
    nodejs-cats-app-deployment-5d67fbc67d-ghtc5   1/1     Running   0          44m

<br/>

    $ kubectl get deployments
    NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
    nodejs-cats-app-deployment   3/3     3            3           48m

<br/>

    $ kubectl get ing
    NAME                      HOSTS                         ADDRESS   PORTS   AGE
    nodejs-cats-app-ingress   nodejs-cats-app.example.com             80      48m

<br/>

    $ kubectl get service
    NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP        66m
    nodejs-cats-app-nodeport   NodePort    10.110.183.157   <none>        80:30123/TCP   49m

<br/>

### На клиенте:

    # vi /etc/hosts

    192.168.0.5 nodejs-cats-app.example.com

<br/>

    $ curl nodejs-cats-app.example.com
    OK

<br/>

### Удалить

    # kubectl delete deployment nodejs-cats-app-deployment
    # kubectl delete ing nodejs-cats-app-ingress
    # kubectl delete service nodejs-cats-app-deployment
    # kubectl delete svc nodejs-cats-app-nodeport
