---
layout: page
title: Services - ClusterIP, NodePort, LoadBalancer, Ingress
description: Services - ClusterIP, NodePort, LoadBalancer, Ingress
keywords: devops, linux, kubernetes, Services - ClusterIP, NodePort, LoadBalancer, Ingress
permalink: /devops/containers/kubernetes/basics/services/
---

# Services: ClusterIP, NodePort, LoadBalancer, Ingress

<br/>

### Kubernetes - Services Explained

<div align="center">

    <iframe width="853" height="480" src="https://www.youtube.com/embed/5lzUpDtmWgM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

</div>

![kubernetes Services](/img/devops/containers/kubernetes/basics/services/services.png 'kubernetes Services'){: .center-image }

<br/>

### ClusterIP

![kubernetes ClusterIP](/img/devops/containers/kubernetes/basics/services/clusterIP.png 'kubernetes ClusterIP'){: .center-image }

<br/>

```
apiVersion: v1
kind: Service
metadata:
  name: my-internal-service
spec:
  selector:
    app: my-app
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```

<br/>

### NodePort

![kubernetes NodePort](/img/devops/containers/kubernetes/basics/services/NodePort.png 'kubernetes NodePort'){: .center-image }

<br/>

**Запускаю deployment**

```
$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quarkus-demo-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: quarkus-demo
  template:
    metadata:
      labels:
        app: quarkus-demo
        env: dev
    spec:
      containers:
      - name: quarkus-demo
        image: quay.io/burrsutter/quarkus-demo:1.0.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
EOF
```

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  selector:
    app: quarkus-demo
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
EOF
```

<br/>

    $ kubectl get services
    NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    nodeport-service   NodePort   10.106.68.183   <none>        80:32321/TCP   17s

<br/>

    $ IP=$(minikube --profile my-profile ip)
    $ echo ${IP}
    $ PORT=$(kubectl get service nodeport-service -o jsonpath="{.spec.ports[*].nodePort}")
    $ echo ${PORT}
    $ while true; do curl $IP:$PORT; sleep .5; done

<br/>

    $ kubectl delete services nodeport-service
    $ kubectl delete deployment quarkus-demo-deployment

<br/>

### LoadBalancer

![kubernetes LoadBalancer](/img/devops/containers/kubernetes/basics/services/LoadBalancer.png 'kubernetes LoadBalancer'){: .center-image }

```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mypython-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mypython
  template:
    metadata:
      labels:
        app: mypython
    spec:
      containers:
      - name: mypython
        image: docker.io/burrsutter/mypython:1.0.0
        ports:
        - containerPort: 8000
EOF
```

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mygo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mygo
  template:
    metadata:
      labels:
        app: mygo
    spec:
      containers:
      - name: mygo
        image: quay.io/burrsutter/mygo:1.0.0
        ports:
        - containerPort: 8000
EOF
```

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mynode-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mynode
  template:
    metadata:
      labels:
        app: mynode
    spec:
      containers:
      - name: mynode
        image: quay.io/burrsutter/mynode:1.0.0
        ports:
        - containerPort: 8000
EOF
```

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
  labels:
    app: mystuff
spec:
  ports:
  - name: http
    port: 8000
  selector:
    inservice: mypods
  type: LoadBalancer
EOF
```

<br/>

    $ kubectl label pod -l app=mypython inservice=mypods
    $ kubectl label pod -l app=mynode inservice=mypods
    $ kubectl label pod -l app=mygo inservice=mypods

<br/>

    $ kubectl get deployments
    NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
    mygo-deployment       1/1     1            1           117s
    mynode-deployment     1/1     1            1           2m5s
    mypython-deployment   1/1     1            1           10m

<br/>

    $ IP=$(minikube --profile my-profile ip)
    $ echo ${IP}
    $ PORT=$(kubectl get service loadbalancer-service -o jsonpath="{.spec.ports[*].nodePort}")
    $ echo ${PORT}

<br/>

    $ while true; do curl $IP:$PORT; sleep .5; done
    Python Hello on mypython-deployment-6874f84d85-kh4g7
    Go Hello on mygo-deployment-6d944c5c69-qx2s6
    Node Hello on mynode-deployment-fb5457c5-hmf67 0
    Node Hello on mynode-deployment-fb5457c5-hmf67 1
    Go Hello on mygo-deployment-6d944c5c69-qx2s6
    Python Hello on mypython-deployment-6874f84d85-kh4g7
    Python Hello on mypython-deployment-6874f84d85-kh4g7
    Python Hello on mypython-deployment-6874f84d85-kh4g7
    Go Hello on mygo-deployment-6d944c5c69-qx2s6

<br/>

    $ kubectl describe service loadbalancer-service

<br/>

    $ kubectl describe service loadbalancer-service
    Name:                     loadbalancer-service
    Namespace:                demo
    Labels:                   app=mystuff
    Annotations:              Selector:  inservice=mypods
    Type:                     LoadBalancer
    IP:                       10.97.67.253
    Port:                     http  8000/TCP
    TargetPort:               8000/TCP
    NodePort:                 http  30460/TCP
    Endpoints:                172.17.0.5:8000,172.17.0.6:8000,172.17.0.7:8000
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:                   <none>

<br/>

    $ kubectl get pods -o wide
    NAME                                   READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
    mygo-deployment-6d944c5c69-qx2s6       1/1     Running   0          18m   172.17.0.6   my-profile   <none>           <none>
    mynode-deployment-fb5457c5-hmf67       1/1     Running   0          12m   172.17.0.7   my-profile   <none>           <none>
    mypython-deployment-6874f84d85-kh4g7   1/1     Running   0          27m   172.17.0.5   my-profile   <none>           <none>

<!--
<br/>

```
kind: Service
apiVersion: v1
metadata:
    name: loadbalancer-service
spec:
    selector:
        app: app-lb
    ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
    clusterIP: <internalIP>
    loadBalancerIP: <externalIP>
    type: LoadBalancer
```

-->

<br/>

### Ingress

![kubernetes Ingress](/img/devops/containers/kubernetes/basics/services/Ingress.png 'kubernetes Ingress'){: .center-image }

<br/>

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
spec:
  backend:
    serviceName: other
    servicePort: 8080
  rules:
  - host: foo.mydomain.com
    http:
      paths:
      - backend:
          serviceName: foo
          servicePort: 8080
  - host: mydomain.com
    http:
      paths:
      - path: /bar/*
        backend:
          serviceName: bar
          servicePort: 8080
```

<br/>

**Скапитализжено:**

https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0
