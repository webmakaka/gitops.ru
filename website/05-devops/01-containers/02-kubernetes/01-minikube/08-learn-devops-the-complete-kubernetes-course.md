---
layout: page
title: Материал из курса - Learn DevOps - The Complete Kubernetes Course
description: Материал из курса - Learn DevOps - The Complete Kubernetes Course
keywords: devops, linux, kubernetes,  Материал из курса - Learn DevOps - The Complete Kubernetes Course
permalink: /devops/containers/kubernetes/minikube/learn-devops-the-complete-kubernetes-course/
---

# Материал из курса: Learn DevOps: The Complete Kubernetes Course

Делаю:  
25.03.2019

<br/>

Материал из курса:

Learn DevOps: The Complete Kubernetes Course

Выпоняю после каждого шага:

    $ minikube stop
    $ minikube delete
    $ minikube start

<br/>

### Подготовка

    $ minikube start
    $ mkdir ~/kubernetes-test1 && cd ~/kubernetes-test1
    $ git clone https://github.com/wardviaene/kubernetes-course
    $ cd kubernetes-course

<br/>

### port-forward

    $ kubectl create -f first-app/helloworld.yml

    $ kubectl get pods
    $ kubectl describe pod nodehelloworld.example.com
    $ kubectl port-forward nodehelloworld.example.com 8081:3000

    $ curl localhost:8081
    Hello World!

    $ ^C

<br/>

### Service

    $ kubectl expose pod nodehelloworld.example.com  --type=NodePort --name helloworld-service

    $ minikube service helloworld-service --url

    http://192.168.99.106:31141

<br/>

    $ kubectl get service
    $ kubectl describe service helloworld-service

<br/>

### Load Balancer

    $ kubectl create -f first-app/helloworld.yml
    $ kubectl create -f first-app/helloworld-service.yml

    $ minikube service helloworld-service --url
    http://192.168.99.107:31975

    $ curl http://192.168.99.107:31975
    Hello World!

<br/>

### Replication Controller

    $ kubectl create -f replication-controller/helloworld-repl-controller.yml

    $ kubectl get pods

    NAME READY STATUS RESTARTS AGE
    helloworld-controller-swghp 1/1 Running 0 80s
    helloworld-controller-zq8m4 1/1 Running 0 80s

    $ kubectl scale --replicas=4 -f replication-controller/helloworld-repl-controller.yml

    $ kubectl get rc
    NAME                    DESIRED   CURRENT   READY   AGE
    helloworld-controller   4         4         4       4m38s

    $ kubectl delete rc/helloworld-controller

<br/>

### Deployments

    $ kubectl create -f deployment/helloworld.yml

    $ kubectl get deploymentsNAME                    READY   UP-TO-DATE   AVAILABLE   AGE
    helloworld-deployment   3/3     3            3           20s

    $ kubectl get rs
    NAME                              DESIRED   CURRENT   READY   AGE
    helloworld-deployment-969d5cbd5   3         3         3       73s

    $ kubectl get pods
    NAME                                    READY   STATUS    RESTARTS   AGE
    helloworld-deployment-969d5cbd5-65zdw   1/1     Running   0          99s
    helloworld-deployment-969d5cbd5-74mt9   1/1     Running   0          99s
    helloworld-deployment-969d5cbd5-rs2s9   1/1     Running   0          99s


    $ kubectl get pods --show-labels
    NAME                                    READY   STATUS    RESTARTS   AGE    LABELS
    helloworld-deployment-969d5cbd5-65zdw   1/1     Running   0          2m4s   app=helloworld,pod-template-hash=969d5cbd5
    helloworld-deployment-969d5cbd5-74mt9   1/1     Running   0          2m4s   app=helloworld,pod-template-hash=969d5cbd5
    helloworld-deployment-969d5cbd5-rs2s9   1/1     Running   0          2m4s   app=helloworld,pod-template-hash=969d5cbd5


    $ kubectl rollout status deployment/helloworld-deployment

    $ kubectl expose deployment helloworld-deployment --type=NodePort
    service/helloworld-deployment exposed

    $ kubectl get service
    NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    helloworld-deployment   NodePort    10.111.138.173   <none>        3000:30462/TCP   29s
    kubernetes              ClusterIP   10.96.0.1        <none>        443/TCP          10h


    $ minikube service helloworld-deployment --url
    http://192.168.99.108:30462

    $ curl http://192.168.99.108:30462
    Hello World!

<br/>

Поменяем версию

    $ kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2

    $ kubectl rollout status deployment/helloworld-deployment

    $ curl http://192.168.99.108:30462
    Hello World v2!

    $ kubectl rollout history deployment/helloworld-deployment
    deployment.extensions/helloworld-deployment
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>

<br/>

Поменяем обратно

    $ kubectl rollout undo deployment/helloworld-deployment

<br/>

### Services

    $ kubectl create -f first-app/helloworld.yml
    $ kubectl create -f first-app/helloworld-nodeport-service.yml

    $ minikube service helloworld-service --url
    http://192.168.99.109:31001

    $ curl http://192.168.99.109:31001
    Hello World!

    $ kubectl delete svc helloworld-service

<br/>

### Labels

    $ kubectl label nodes minikube hardware=high-spec

    $ kubectl get nodes --show-labels

    $ kubectl create -f deployment/helloworld-nodeselector.yml

    $ kubectl describe pod helloworld-deployment-794c748d5b-lw877

<br/>

### Healthchecks

    $ kubectl create -f deployment/helloworld-healthcheck.yml

<br/>

### Credentials

    $ kubectl create -f deployment/helloworld-secrets.yml

    $ kubectl create -f deployment/helloworld-secrets-volumes.yml

    $ exec helloworld-deployment-blalbalba -i -t -- /bin/bash

    $ cat /etc/creds/username
    $ cat /etc/creds/password

<br/>

### Running Wordpress on Kubernetes (при перезагрузке данные потеряются!)

    $ kubectl create -f wordpress/wordpress-secrets.yml

    $ kubectl create -f wordpress/wordpress-single-deployment-no-volumes.yml

    $ kubectl create -f wordpress/wordpress-service.yml

    $ minikube service wordpress-service --url
    http://192.168.99.111:31001

<br/>

### Service Discovery

    $ kubectl create -f service-discovery/secrets.yml

    $ kubectl create -f service-discovery/database.yml

    $ kubectl create -f service-discovery/database-service.yml

    $ kubectl create -f service-discovery/helloworld-db.yml

    $ kubectl create -f service-discovery/helloworld-db-service.yml

    $ minikube service helloworld-db-service --url

    $ kubectl exec database -i -t -- mysql -u root -p

rootpassword

    mysql> show databases;

    mysql> use helloworld

    mysql> show tables;

    mysql> select * from visits;

    mysql> exit

<br/>

    $ kubectl get svc
    NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    database-service        NodePort    10.107.72.126   <none>        3306:30457/TCP   16m
    helloworld-db-service   NodePort    10.111.22.17    <none>        3000:31937/TCP   14m
    kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP          74m


    $ kubectl run -i --tty busybox --image=busybox --restart=Never -- sh

    # nslookup database-service
    # nslookup helloworld-db-service

<br/>

### ConfigMap

    $ kubectl create configmap nginx-config --from-file=configmap/reverseproxy.conf
    $ kubectl get configmap

    $ kubectl get configmap
    NAME           DATA   AGE
    nginx-config   1      32s


    $ kubectl get configmap nginx-config -o yaml

    $ kubectl create -f configmap/nginx.yml

    $ kubectl create -f configmap/  nginx-service.yml

    $ minikube service helloworld-nginx-service --url
    http://192.168.99.112:31324

<br/>

### Ingress Controller

    $ kubectl create -f ingress/ingress.yml
    $ kubectl create -f ingress/nginx-ingress-controller.yml
    $ kubectl create -f ingress/echoservice.yml
    $ kubectl create -f ingress/helloworld-v1.yml
    $ kubectl create -f ingress/helloworld-v2.yml

    $ minikube ip
    192.168.99.113

    $ kubectl get pods

    $ curl 192.168.99.113
    default backend - 404

    $ curl 192.168.99.113 -H 'Host: helloworld-v1.example.com'
    Hello World!

    $ curl 192.168.99.113 -H 'Host: helloworld-v2.example.com'
    Hello World v2!

    $ kubectl get svc
    NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    echoheaders-default   NodePort    10.96.161.76   <none>        80:30302/TCP   4m30s
    helloworld-v1         NodePort    10.110.25.93   <none>        80:30303/TCP   4m25s
    helloworld-v2         NodePort    10.96.91.129   <none>        80:30304/TCP   4m19s
    kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP        6m12s

<br/>

### Volumes

Используется AWS, я не стал изучать

<br/>

### Namespace quotas

    $ kubectl create -f resourcequotas/resourcequota.yml
    $ kubectl create -f resourcequotas/helloworld-no-quotas.yml

    $ kubectl get deploy --namespace=myspace
    $ kubectl get rs --namespace=myspace

    $ kubectl delete deploy/helloworld-deployment --namespace=myspace

    $ kubectl create -f resourcequotas/helloworld-with-quotas.yml

    $ kubectl get quota --namespace=myspace

    $ kubectl describe quota/compute-quota --namespace=myspace

    $ kubectl delete deploy/helloworld-deployment --namespace=myspace

    $ kubectl create -f resourcequotas/defaults.yml

    $ kubectl describe limits limits --namespace=myspace

    $ kubectl create -f resourcequotas/helloworld-no-quotas.yml

    $ kubectl get pods --namespace=myspace

<!-- <br/>

### Adding Users

    $ minikube ssh

    // хз как установить package manager

    $ openssl genrsa -out edward.pem 2048

    $ openssl req -new -key edward.pem -out edward-ssr.pem -subj "/CN=edward/0=myteam/"

    $ sudo openssl x509 -req -in edward-csr.pem -CA /var/lib/localkube/certs/ca.crt -CAkey /var/lib/localkube/certs/ca.key -CAcreateserial -out edward.crt -days 10000

    не разобрался.
    -->
