---
layout: page
title: KubernetesInc Ingress Nginx
description: KubernetesInc Ingress Nginx
keywords: devops, linux, kubernetes, KubernetesInc Ingress Nginx
permalink: /devops/containers/kubernetes/kubeadm/ingress/kubernetesinc-ingress-nginx/
---

# KubernetesInc Ingress Nginx

Делаю  
24.04.2019

# Пример из видеокурса Learn DevOps The Complete Kubernetes Course

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

    В общем, если использовать последнюю версию ingress controller (mandatory.yaml), что предлагают на github, то не работает. Я по быстрому не разобрался в чем причина. Может потом, когда будет более актуально. Также выслушаю предложения в чате.

<br/>

    $ cd ~/tmp
    $ curl -LJO https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/nginx-ingress-controller.yml

    $ vi nginx-ingress-controller.yml

меняю версию nginx-ingress-controller:0.17.1 на nginx-ingress-controller:0.24.1

    $ kubectl create -f nginx-ingress-controller.yml

<!--

    // ingress controller
    $ kubectl create -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

-->

<br/>

    $ kubectl create -f https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/ingress.yml

<br/>

    $ kubectl create -f https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/echoservice.yml

    $ kubectl create -f https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/helloworld-v1.yml

    $ kubectl create -f https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/helloworld-v2.yml

    $ kubectl get pods

    $ curl 192.168.0.11

    $ curl http://192.168.0.11 -H 'Host:helloworld-v1.example.com'
    Hello World!

<br/>

Устанавливю haproxy как <a href="/devops/containers/kubernetes/kubeadm/ingress/haproxy/">здесь</a>

<br/>

    $ echo "192.168.0.5 helloworld-v1.example.com" | sudo tee -a /etc/hosts

    $ echo "192.168.0.5 helloworld-v2.example.com" | sudo tee -a /etc/hosts

<br/>

    $ curl helloworld-v1.example.com
    $ curl helloworld-v2.example.com
    OK

<br/>

## Попробуем запустить свое приложение:

    $ kubectl delete -f https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/ingress.yml

    $ kubectl delete -f https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/helloworld-v1.yml

    $ kubectl delete -f https://raw.githubusercontent.com/wardviaene/kubernetes-course/master/ingress/helloworld-v2.yml

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nodejs-cats-app-rules
spec:
  rules:
  - host: nodejs-cats-app.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nodejs-cats-app
          servicePort: 80
EOF
```

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nodejs-cats-app-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nodejs-cats-app
    spec:
      containers:
      - name: nodejs-cats-app
        image: webmakaka/cats-app:latest
---
apiVersion: v1
kind: Service
metadata:
  name: nodejs-cats-app
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30303
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: nodejs-cats-app
EOF
```

<br/>

    $ curl http://192.168.0.11 -H 'Host:nodejs-cats-app.example.com'
    OK

<br/>

### Еще один пример. На этот раз из видеокурса Packtpub - Kubernetes Recipes [Video].

Качать, а тем более покупать его я никому не рекомендую!

    $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

<br/>

    $ kubectl get pods --all-namespaces | grep ingress-nginx
    ingress-nginx   nginx-ingress-controller-5694ccb578-2jqw2   1/1     Running   0          97s

<br/>

    $ mkdir -p ~/tmp/ingress && cd ~/tmp/ingress

<br/>

    # vi web-01.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: web-01
  labels:
    app: nginx

spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

<br/>

    $ kubectl create -f web-01.yaml

<br/>

    $ kubectl expose pod web-01 --type="ClusterIP" --port 80

<br/>

    $ kubectl get services
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   3h10m
    web-01       ClusterIP   10.108.124.242   <none>        80/TCP    12s

<br/>

    # vi ingress.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
  - name: http-new
    port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30090
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```

<br/>

    $ kubectl create -f ingress.yaml

<br/>

    $ kubectl get service --all-namespaces
    NAMESPACE       NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
    default         kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP                  3h15m
    default         web-01          ClusterIP   10.108.124.242   <none>        80/TCP                   4m52s
    ingress-nginx   ingress-nginx   NodePort    10.96.176.174    <none>        80:30090/TCP             21s
    kube-system     kube-dns        ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   3h15m

<br/>

### Now we will define the rules for ingress

<br/>

    # vi ingress-rules.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-test
  annotations:
    kubernetes.io/ingress.class: nginx
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: kube-master.labs.local
    http:
      paths:
        - path: /
          backend:
            serviceName: web-01
            servicePort: 80
```

<br/>

    $ kubectl create -f ingress-rules.yaml

<br/>

    $ kubectl get ingress
    NAME           HOSTS                    ADDRESS   PORTS   AGE
    ingress-test   kube-master.labs.local             80      91s

<br/>

    $ sudo su -
    # vi /etc/hosts

    192.168.0.11 kube-master.labs.local

<br/>

192.168.0.11 - одна из нод кластера.

<br/>

    $ curl http://192.168.0.11:30090 -H 'Host:kube-master.labs.local'
