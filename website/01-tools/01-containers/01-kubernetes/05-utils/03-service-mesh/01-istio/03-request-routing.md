---
layout: page
title: Istio Request Routing
description: Istio Request Routing
keywords: linux, kubernetes, Istio, Request Routing
permalink: /tools/containers/kubernetes/utils/service-mesh/istio/request-routing/
---

# Istio Request Routing

Поднята виртуальная машина с minikube <a href="/tools/containers/kubernetes/utils/service-mesh/istio/setup/">следующим образом</a>.

<br/>

Делаю:  
08.11.2021

https://www.youtube.com/watch?v=a0Mu0hQ9zzI

<br/>

**Original src:**  
https://github.com/carnage-sh/cloud-for-fun/tree/master/blog/istio-routing

Работаем со скриптами из каталога cloud-for-fun/blog/istio-routing

<br/>

```
$ export INGRESS_HOST=$(kubectl \
 --namespace istio-system \
 get service istio-ingressgateway \
 --output jsonpath="{.status.loadBalancer.ingress[0].ip}")

$ echo ${INGRESS_HOST}
```

<br/>

**helloworld-v1.yaml**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: helloworld-v1
  labels:
    app: helloworld
spec:
  ports:
  - port: 5000
    name: http
  selector:
    app: helloworld
    version: v1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-v1
  labels:
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
        version: v1
    spec:
      containers:
      - name: helloworld
        image: istio/examples-helloworld-v1
        resources:
          requests:
            cpu: "100m"
        imagePullPolicy: IfNotPresent #Always
        ports:
        - containerPort: 5000
EOF
```

<br/>

**helloworld-gateway-v1.yaml**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: helloworld-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: helloworld
spec:
  hosts:
  - "*"
  gateways:
  - helloworld-gateway
  http:
  - match:
    - uri:
        exact: /
    rewrite:
      uri: "/hello"
    route:
    - destination:
        host: helloworld-v1
        port:
          number: 5000
EOF
```

<br/>

**helloworld-v2.yaml**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: helloworld-v2
  labels:
    app: helloworld
spec:
  ports:
  - port: 5000
    name: http
  selector:
    app: helloworld
    version: v2
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-v2
  labels:
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
        version: v2
    spec:
      containers:
      - name: helloworld
        image: istio/examples-helloworld-v2
        resources:
          requests:
            cpu: "100m"
        imagePullPolicy: IfNotPresent #Always
        ports:
        - containerPort: 5000
EOF
```

<br/>

**helloworld-gateway-v2.yaml**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: helloworld
spec:
  hosts:
  - "*"
  gateways:
  - helloworld-gateway
  http:
  - match:
    - uri:
        exact: /
      headers:
        x-user:
          exact: "gregory"
    rewrite:
      uri: "/hello"
    route:
    - destination:
        host: helloworld-v2
        port:
          number: 5000
  - match:
    - uri:
        exact: /
    rewrite:
      uri: "/hello"
    route:
    - destination:
        host: helloworld-v1
        port:
          number: 5000
EOF
```

<br/>

```
$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
helloworld-v1-7476658d4f-45bq4   2/2     Running   0          21s
helloworld-v2-786b58884c-mfmrl   2/2     Running   0          13s


```

<!--

<br/>

    // Или посмотреть EXTERNAL-IP
    $ kubectl get service -n istio-system istio-ingressgateway

-->

<br/>

    $ curl $INGRESS_HOST
    Hello version: v1, instance: helloworld-v1-7695cb4556-9dnsx

<br/>

    $ curl $INGRESS_HOST -H 'x-user: gregory'
    Hello version: v2, instance: helloworld-v2-58b576ddf4-49zds

<br/>

В общем для "gregory" выделенный сервис.

<br/>

```
$ kubectl get vs
NAME         GATEWAYS                 HOSTS   AGE
helloworld   ["helloworld-gateway"]   ["*"]   94s
```

<br/>

```
$ kubectl get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
helloworld-v1   ClusterIP   10.101.16.86     <none>        5000/TCP   6m10s
helloworld-v2   ClusterIP   10.110.180.253   <none>        5000/TCP   3m53s
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP    7m56s
```
