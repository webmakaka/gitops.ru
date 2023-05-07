---
layout: page
title: Traefik ingress
description: Traefik ingress
keywords: devops, linux, kubernetes, Traefik ingress
permalink: /devops/containers/kubernetes/kubeadm/ingress/traefik-ingress/
---

# Traefik ingress

Делаю  
04.04.2019

По материалам:  
https://www.youtube.com/watch?v=A_PjjCM1eLA

<br/>
Рисунок индуса:
<br/>

![kubernetes ingress](/img/devops/containers/kubernetes/kubeadm/ingress/ingress.png 'kubernetes ingress'){: .center-image }

<br/>

Подготовили кластер, окружение и haproxy как <a href="/devops/containers/kubernetes/kubeadm/ingress/nginxinc-kubernets-ingress/">здесь</a>.

<br/>

### Создаем traefik ingress контроллер

Ссылки с сайта Traefik на github были битые.

<br/>

https://docs.traefik.io/v1.6/user-guide/kubernetes/

<br/>

    $ mkdir /tmp/traefik && cd /tmp/traefik

<br/>

    $ vi traefik-rbac.yaml

```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system

```

<br/>

    $ kubectl apply -f traefik-rbac.yaml

<br/>

    $ kubectl describe clusterrole traefik-ingress-controller -n kube-system
    Name:         traefik-ingress-controller
    Labels:       <none>
    Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                    {"apiVersion":"rbac.authorization.k8s.io/v1beta1","kind":"ClusterRole","metadata":{"annotations":{},"name":"traefik-ingress-controller"},"...
    PolicyRule:
    Resources             Non-Resource URLs  Resource Names  Verbs
    ---------             -----------------  --------------  -----
    endpoints             []                 []              [get list watch]
    secrets               []                 []              [get list watch]
    services              []                 []              [get list watch]
    ingresses.extensions  []                 []              [get list watch]

<br/>

    $ kubectl get ns
    NAME              STATUS   AGE
    default           Active   23m
    kube-node-lease   Active   24m
    kube-public       Active   24m
    kube-system       Active   24m

<br/>

    $ vi traefik-ds.yaml

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin

```

<br/>

    $ kubectl apply -f traefik-ds.yaml

<br/>

    $ kubectl get all -n kube-system | grep traefik

    pod/traefik-ingress-controller-g84gh     1/1     Running   0          15s
    pod/traefik-ingress-controller-zgqd6     1/1     Running   0          15s

    service/traefik-ingress-service   ClusterIP   10.106.161.165   <none>        80/TCP,8080/TCP          15s

    daemonset.apps/traefik-ingress-controller   2         2         2       2            2           <none>                          15s

<br/>

### Запускаем приложение

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/ingress-demo/nginx-deploy-main.yaml

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/ingress-demo/nginx-deploy-green.yaml

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/ingress-demo/nginx-deploy-blue.yaml

<br/>

    $ kubectl get svc
    NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   28m

<br/>

    $ kubectl expose deploy nginx-deploy-main --port 80
    $ kubectl expose deploy nginx-deploy-blue --port 80
    $ kubectl expose deploy nginx-deploy-green --port 80

<br/>

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/ingress-demo/ingress-resource-2.yaml

<br/>

    $ kubectl get ing
    NAME                 HOSTS                                                              ADDRESS   PORTS   AGE
    ingress-resource-2   nginx.example.com,blue.nginx.example.com,green.nginx.example.com             80      5s

<br/>

    $ kubectl describe ing ingress-resource-2
    Name:             ingress-resource-2
    Namespace:        default
    Address:
    Default backend:  default-http-backend:80 (<none>)
    Rules:
    Host                     Path  Backends
    ----                     ----  --------
    nginx.example.com
                                nginx-deploy-main:80 (<none>)
    blue.nginx.example.com
                                nginx-deploy-blue:80 (<none>)
    green.nginx.example.com
                                nginx-deploy-green:80 ()
    Annotations:
    Events:  <none>

<br/>

    $ kubectl get pods -o wide
    NAME                                 READY   STATUS    RESTARTS   AGE   IP           NODE        NOMINATED NODE   READINESS GATES
    nginx-deploy-blue-7cc7d854dc-njg7q   1/1     Running   0          20m   10.244.2.4   node2.k8s   <none>           <none>
    nginx-deploy-green-fbbd6d8d8-2szhx   1/1     Running   0          32m   10.244.2.3   node2.k8s   <none>           <none>
    nginx-deploy-main-77f4b995c8-rwwjz   1/1     Running   0          7s    10.244.1.8   node1.k8s   <none>           <none>

<br/>

### На локальном хосту

    # vi /etc/hosts

    192.168.0.10 nginx.example.com
    192.168.0.10 blue.nginx.example.com
    192.168.0.10 green.nginx.example.com

<br/>

    $ curl nginx.example.com
    $ curl blue.nginx.example.com
    $ curl green.nginx.example.com
    OK

<br/>

### Возможно, полезные материалы

Kubernetes Ingress Explained  
https://www.youtube.com/watch?v=izWCkcJAzBw
