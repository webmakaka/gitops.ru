---
layout: page
title: Kubernetes Docker Registry
description: Kubernetes Docker Registry
keywords: linux, kubernetes, Docker Registry
permalink: /tools/containers/kubernetes/tools/registries/standard/v2/
---

# Kubernetes Docker Registry

**Из примера с katacoda (которой уже нет):**  
https://www.katacoda.com/javajon/courses/kubernetes-pipelines/tekton

<br/>

**Требуется попробовать! Спустя столько времени!**

<br/>

```
$ {
    minikube --profile my-profile config set memory 8192
    minikube --profile my-profile config set cpus 4

    minikube --profile my-profile config set vm-driver virtualbox
    // minikube --profile my-profile config set vm-driver docker

    minikube --profile my-profile config set kubernetes-version v1.14.1
    minikube start --profile my-profile
}
```

<br/>

    // Удалить
    // $ minikube --profile my-profile stop && minikube --profile my-profile delete

<br/>

### Инсталляция пакетов с помощью helm

    $ helm repo add stable https://kubernetes-charts.storage.googleapis.com/

    $ helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator

    $ helm repo update

    $ helm install private stable/docker-registry --namespace kube-system

<br/>

### Install Registry Proxies as Node Daemons

    $ helm install registry-proxy incubator/kube-registry-proxy \
    --set registry.host=private-docker-registry.kube-system \
    --set registry.port=5000 \
    --set hostPort=5000 \
    --namespace kube-system

<br/>

Pods can pull images from the registry at http://localhost:5000 and the proxies resolve the requests to https://private-docker-registry.kube-system:5000.

<br/>

### Install Registry UI

https://github.com/Joxit/docker-registry-ui

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: registry-ui-deployment
  labels:
    app: registry-ui
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry-ui
  template:
    metadata:
      labels:
        app: registry-ui
    spec:
      containers:
      - name: reg-ui
        image: joxit/docker-registry-ui:static
        env:
        - name: REGISTRY_URL
          value: "http://private-docker-registry:5000"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: registry-ui
  labels:
    app: registry-ui
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31000
    protocol: TCP
  selector:
    app: registry-ui
EOF
```

<br/>

    $ kubectl get svc -n kube-system | grep private-docker-registry
    private-docker-registry   ClusterIP   10.102.91.197   <none>        5000/TCP                 17m

<br/>

    $ minikube --profile my-profile ip
    192.168.99.130

<br/>

http://192.168.99.130:31000/

<br/>

### Deploy Tekton Controller

    $ kubectl apply --filename https://storage.googleapis.com/tekton-releases/latest/release.yaml


    $ watch kubectl get deployments,pods,services --namespace tekton-pipelines

    $ kubectl get crds
    NAME                                  CREATED AT
    clustertasks.tekton.dev               2020-04-23T14:19:27Z
    conditions.tekton.dev                 2020-04-23T14:19:27Z
    images.caching.internal.knative.dev   2020-04-23T14:19:27Z
    pipelineresources.tekton.dev          2020-04-23T14:19:27Z
    pipelineruns.tekton.dev               2020-04-23T14:19:27Z
    pipelines.tekton.dev                  2020-04-23T14:19:27Z
    taskruns.tekton.dev                   2020-04-23T14:19:27Z
    tasks.tekton.dev                      2020-04-23T14:19:27Z

<br/>

### Tekton CLI installation

    # Get the tar.xz
    $ curl -LO https://github.com/tektoncd/cli/releases/download/v0.8.0/tkn_0.8.0_Linux_x86_64.tar.gz

    # Extract tkn to your PATH (e.g. /usr/local/bin)
    $ sudo tar xvzf tkn_0.8.0_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn

<br/>