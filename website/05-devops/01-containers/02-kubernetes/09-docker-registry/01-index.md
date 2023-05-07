---
layout: page
title: Kubernetes Docker Registry
description: Kubernetes Docker Registry
keywords: linux, kubernetes, Docker Registry
permalink: /devops/containers/kubernetes/docker-registry-2/
---

# Kubernetes Docker Registry

**Из примера с katacoda:**  
https://www.katacoda.com/javajon/courses/kubernetes-pipelines/registries

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

    $ minikube --profile my-profile ssh

<br/>

    $ sudo mkdir -p  /usr/local/bin/
    $ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
    $ export PATH=$PATH:/usr/local/bin/
    $ sudo su -
    /bin/toolbox
    # dnf install -y curl

<br/>

### Инсталляция пакетов с помощью helm

    $ helm repo add stable https://kubernetes-charts.storage.googleapis.com/

    $ helm repo update

    $ helm install private stable/docker-registry --namespace kube-system \
     --set image.tag=2.7.1 \
     --set service.type=NodePort \
     --set service.nodePort=31500

<br/>

    $ kubectl get service --namespace kube-system
    NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
    kube-dns                  ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   4m31s
    private-docker-registry   NodePort    10.102.167.234   <none>        5000:31500/TCP           15s

<br/>

    $ kubectl port-forward --namespace kube-system \
     $(kubectl get po -n kube-system | grep private-docker-registry | \
     awk '{print $1;}') 5000:5000 &

<br/>

    $ export REGISTRY=127.0.0.1:31500

<br/>

    $ kubectl get deployments private-docker-registry --namespace kube-system
    NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
    private-docker-registry   1/1     1            1           3m16s

<br/>

    $ curl $REGISTRY/v2/_catalog

<br/>

### Pull and push a container

    $ docker pull replicated/dockerfilelint
    $ docker tag replicated/dockerfilelint $REGISTRY/dockerfilelint
    $ docker push $REGISTRY/dockerfilelint
    $ curl $REGISTRY/v2/_catalog

<br/>

### Registry Web Interface

```
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

### Push the Container

    $ cd ~/
    $ git clone https://github.com/IBM/MAX-Breast-Cancer-Mitosis-Detector.git && cd MAX-Breast-Cancer-Mitosis-Detector

    $ docker build -t $REGISTRY/max-breast-cancer-mitosis-detector .

    $ docker push $REGISTRY/max-breast-cancer-mitosis-detector

    $ curl $REGISTRY/v2/_catalog
    {"repositories":["dockerfilelint","max-breast-cancer-mitosis-detector"]}

<br/>

### Pull the Container

\$ cd ~ && envsubst < max-breast-cancer-mitosis-detector.yaml > max-breast-cancer-mitosis-detector-modified.yaml

\$ kubectl apply -f max-breast-cancer-mitosis-detector-modified.yaml

\$ kubectl get deployments,pods,services

\$ export APP=http://127.0.0.1:32500/

\$ cd ~/MAX-Breast-Cancer-Mitosis-Detector

$ curl -F image=@samples/true.png -XPOST "${APP}model/predict"

$ curl -F image=@samples/false.png -XPOST "${APP}model/predict"
