---
layout: page
title: Managing Deployments Using Kubernetes Engine
description: Managing Deployments Using Kubernetes Engine
keywords: Managing Deployments Using Kubernetes Engine
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/managing-deployments-using-kubernetes-engine/
---

# [GSP053] Managing Deployments Using Kubernetes Engine

    $ gcloud config set compute/zone us-central1-a

    $ git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git

    $ cd orchestrate-with-kubernetes/kubernetes

    $ gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"

<br/>

    $ vi deployments/auth.yaml

```
containers:
- name: auth
  image: kelseyhightower/auth:1.0.0
```

    $ kubectl create -f deployments/auth.yaml
    $ kubectl create -f services/auth.yaml
    $ kubectl create -f deployments/hello.yaml
    $ kubectl create -f services/hello.yaml

<br/>

    $ kubectl create secret generic tls-certs --from-file tls/
    $ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
    $ kubectl create -f deployments/frontend.yaml
    $ kubectl create -f services/frontend.yaml

<br/>

    $ kubectl get services frontend

<br/>

    $ curl -ks https://<EXTERNAL-IP>

Или лучше:

    $ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
    {"message":"Hello"}

<br/>

**Scale a Deployment**

    $ kubectl explain deployment.spec.replicas

    $ kubectl scale deployment hello --replicas=5

<br/>

    $ kubectl get pods | grep hello- | wc -l
    5

<br/>

    $ kubectl scale deployment hello --replicas=3

<br/>

**Rolling update**

![Managing Deployments Using Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/managing-deployments-using-kubernetes-engine/pic1.png 'Managing Deployments Using Kubernetes Engine'){: .center-image }

    $ kubectl edit deployment hello

```
containers:
- name: hello
  image: kelseyhightower/hello:2.0.0
```

<br/>

    $ kubectl get replicaset
    NAME                  DESIRED   CURRENT   READY   AGE
    auth-6bb8dcd7bd       1         1         1       11m
    frontend-77f46bf858   1         1         1       10m
    hello-5cbf94fc49      0         0         0       11m
    hello-677685c76       3         3         3       20s

<br/>

    $ kubectl rollout history deployment/hello
    deployment.extensions/hello
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>

<br/>

**Pause a rolling update**

    $ kubectl rollout pause deployment/hello

    $ kubectl rollout status deployment/hello
    deployment "hello" successfully rolled out

    $ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
    auth-6bb8dcd7bd-fpttf           kelseyhightower/auth:1.0.0
    frontend-77f46bf858-p77hp               nginx:1.9.14
    hello-677685c76-lr92g           kelseyhightower/hello:2.0.0
    hello-677685c76-tj25x           kelseyhightower/hello:2.0.0
    hello-677685c76-vwd2l           kelseyhightower/hello:2.0.0

<br/>

**Resume a rolling update**

    $ kubectl rollout resume deployment/hello
    $ kubectl rollout status deployment/hello

<br/>

**Rollback an update**

    $ kubectl rollout undo deployment/hello
    $ kubectl rollout history deployment/hello

    $ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
    auth-6bb8dcd7bd-fpttf           kelseyhightower/auth:1.0.0
    frontend-77f46bf858-p77hp               nginx:1.9.14
    hello-5cbf94fc49-2j62n          kelseyhightower/hello:1.0.0
    hello-5cbf94fc49-sfzs8          kelseyhightower/hello:1.0.0
    hello-5cbf94fc49-tcm6w          kelseyhightower/hello:1.0.0

<br/>

### Canary deployments

When you want to test a new deployment in production with a subset of your users, use a canary deployment. Canary deployments allow you to release a change to a small subset of your users to mitigate risk associated with new releases.

**Create a canary deployment**

![Managing Deployments Using Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/managing-deployments-using-kubernetes-engine/pic2.png 'Managing Deployments Using Kubernetes Engine'){: .center-image }

<br/>

    $ vi deployments/hello-canary.yaml

version: 1.0.0

```
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        track: canary
        # Use ver 1.0.0 so it matches version on service selector
        version: 1.0.0
```

    $ kubectl create -f deployments/hello-canary.yaml

<br/>

    $ kubectl get deployments
    NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    auth           1         1         1            1           20m
    frontend       1         1         1            1           19m
    hello          3         3         3            3           20m
    hello-canary   1         1         1            1           18s

<br/>

**Verify the canary deployment**

    $ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
    {"version":"1.0.0"}

В общем если повторять, то с вероятностью 25% можно получить версию 2.0.0

<br/>

**Blue-green deployments**

Rolling updates are ideal because they allow you to deploy an application slowly with minimal overhead, minimal performance impact, and minimal downtime. There are instances where it is beneficial to modify the load balancers to point to that new version only after it has been fully deployed. In this case, blue-green deployments are the way to go.

Kubernetes achieves this by creating two separate deployments; one for the old "blue" version and one for the new "green" version. Use your existing hello deployment for the "blue" version. The deployments will be accessed via a Service which will act as the router. Once the new "green" version is up and running, you'll switch over to using that version by updating the Service.

<br/>

![Managing Deployments Using Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/managing-deployments-using-kubernetes-engine/pic3.png 'Managing Deployments Using Kubernetes Engine'){: .center-image }

<br/>

    $ kubectl apply -f services/hello-blue.yaml
    $ kubectl create -f deployments/hello-green.yaml

    $ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

<br/>

    $ kubectl apply -f services/hello-green.yaml

Теперь всегда будет использоваться версия 2.0.0

    $ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

<br/>

**Blue-Green Rollback**

    $ kubectl apply -f services/hello-blue.yaml

А теперь все время 1.0.0

    $ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

<br/>

    $ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
    auth-6bb8dcd7bd-fpttf           kelseyhightower/auth:1.0.0
    frontend-77f46bf858-p77hp               nginx:1.9.14
    hello-5cbf94fc49-2j62n          kelseyhightower/hello:1.0.0
    hello-5cbf94fc49-sfzs8          kelseyhightower/hello:1.0.0
    hello-5cbf94fc49-tcm6w          kelseyhightower/hello:1.0.0
    hello-canary-67ddbf5d7c-ldm6h           kelseyhightower/hello:2.0.0
    hello-green-5bf7fc86ff-hdzx8            kelseyhightower/hello:2.0.0
    hello-green-5bf7fc86ff-mpwmn            kelseyhightower/hello:2.0.0
    hello-green-5bf7fc86ff-sww8j            kelseyhightower/hello:2.0.0
