---
layout: page
title: Orchestrating the Cloud with Kubernetes
description: Orchestrating the Cloud with Kubernetes
keywords: Orchestrating the Cloud with Kubernetes
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/orchestrating-the-cloud-with-kubernetes/
---

# [GSP021] Orchestrating the Cloud with Kubernetes

<a href="https://github.com/kelseyhightower/app">App</a> is hosted on GitHub and provides an example 12-Factor application. During this lab you will be working with the following Docker images:

<a href="https://hub.docker.com/r/kelseyhightower/monolith">kelseyhightower/monolith</a> - Monolith includes auth and hello services.
<a href="https://hub.docker.com/r/kelseyhightower/auth">kelseyhightower/auth</a> - Auth microservice. Generates JWT tokens for authenticated users.
<a href="https://hub.docker.com/r/kelseyhightower/hello">kelseyhightower/hello</a> - Hello microservice. Greets authenticated users.
ngnix - Frontend to the auth and hello services.

<br/>

    $ gcloud config set compute/zone us-central1-b
    $ gcloud container clusters create io

<br/>

    $ kubectl run nginx --image=nginx:1.10.0
    $ kubectl expose deployment nginx --port 80 --type LoadBalancer

    $ kubectl get services

<br/>

**Creating a Service**

<br/>

    $ git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git

    $ cd orchestrate-with-kubernetes/kubernetes

    $ kubectl create -f pods/monolith.yaml
    $ kubectl get pods

    $ kubectl describe pods monolith

<br/>

    $ kubectl port-forward monolith 10080:80

Сессия 2

    $ curl http://127.0.0.1:10080
    {"message":"Hello"}

    $ curl http://127.0.0.1:10080/secure
    authorization failed

    $ curl -u user http://127.0.0.1:10080/login
    Enter host password for user 'user': [password]

    $ TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
    Enter host password for user 'user':[password]

    $ curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
    {"message":"Hello"}

    $ kubectl logs -f monolith

Сессия 3

    $ curl http://127.0.0.1:10080

<br/>

    $ kubectl exec monolith --stdin --tty -c monolith /bin/sh
    # ping -c 3 google.com
    # exit

<br>

**Services**

    * ClusterIP (internal) -- the default type means that this Service is only visible inside of the cluster,
    * NodePort gives each node in the cluster an externally accessible IP and
    * LoadBalancer adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

<br>

    $ cd ~/orchestrate-with-kubernetes/kubernetes

    $ kubectl create secret generic tls-certs --from-file tls/
    $ kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
    $ kubectl create -f pods/secure-monolith.yaml

    $ kubectl create -f services/monolith.yaml

    $ gcloud compute firewall-rules create allow-monolith-nodeport \
    --allow=tcp:31000

    NAME                     NETWORK  DIRECTION  PRIORITY  ALLOW      DENY  DISABLED
    allow-monolith-nodeport  default  INGRESS    1000      tcp:31000        False

<br/>

    $ gcloud compute instances list
    NAME                               ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP      STATUS
    gke-io-default-pool-f40e6c02-961s  us-central1-b  n1-standard-1               10.128.0.4   34.66.156.29     RUNNING
    gke-io-default-pool-f40e6c02-b6rw  us-central1-b  n1-standard-1               10.128.0.2   104.197.229.212  RUNNING
    gke-io-default-pool-f40e6c02-jw06  us-central1-b  n1-standard-1               10.128.0.3   35.232.234.44    RUNNING

<br/>

    $ curl -k https://<EXTERNAL_IP>:31000

<br/>

**Adding Labels to Pods**

    $ kubectl get pods -l "app=monolith"
    NAME              READY   STATUS    RESTARTS   AGE
    monolith          1/1     Running   0          22m
    secure-monolith   2/2     Running   0          8m29s

<br/>

    $ kubectl get pods -l "app=monolith,secure=enabled"
    No resources found.

<br/>

    $ kubectl label pods secure-monolith 'secure=enabled'

    $ kubectl get pods secure-monolith --show-labels
    NAME              READY   STATUS    RESTARTS   AGE     LABELS
    secure-monolith   2/2     Running   0          9m56s   app=monolith,secure=enabled

<br/>

    $ kubectl describe services monolith | grep Endpoints
    Endpoints:                10.4.1.4:443

<br/>

    $ gcloud compute instances list
    NAME                               ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP      STATUS
    gke-io-default-pool-f40e6c02-961s  us-central1-b  n1-standard-1               10.128.0.4   34.66.156.29     RUNNING
    gke-io-default-pool-f40e6c02-b6rw  us-central1-b  n1-standard-1               10.128.0.2   104.197.229.212  RUNNING
    gke-io-default-pool-f40e6c02-jw06  us-central1-b  n1-standard-1               10.128.0.3   35.232.234.44    RUNNING

<br/>

    $ curl -k https://10.4.1.4:31000

<br/>

**Deploying Applications with Kubernetes**

    auth - Generates JWT tokens for authenticated users.
    hello - Greet authenticated users.
    frontend - Routes traffic to the auth and hello services.

<br/>

    $ kubectl create -f deployments/auth.yaml
    $ kubectl create -f services/auth.yaml

    $ kubectl create -f deployments/hello.yaml
    $ kubectl create -f services/hello.yaml

    $ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
    $ kubectl create -f deployments/frontend.yaml
    $ kubectl create -f services/frontend.yaml

<br/>

    $ kubectl get services frontend
    NAME       TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)         AGE
    frontend   LoadBalancer   10.7.250.249   35.188.84.62   443:32527/TCP   39s

    $ curl -k https://35.188.84.62
    {"message":"Hello"}
