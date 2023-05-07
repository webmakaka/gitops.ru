---
layout: page
title: NGINX Ingress Controller on Google Kubernetes Engine
description: NGINX Ingress Controller on Google Kubernetes Engine
keywords: NGINX Ingress Controller on Google Kubernetes Engine
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-solutions/nginx-ingress-controller-on-google-kubernetes-engine/
---

# [GSP181] NGINX Ingress Controller on Google Kubernetes Engine

<br/>

Делаю:  
25.05.2019

https://www.qwiklabs.com/focuses/872?parent=catalog

<br/>

## Overview

In Kubernetes, Ingress allows external users and client applications access to HTTP services. Ingress consists of two components: an Ingress Resource and an Ingress Controller:

-   Ingress Resource is a collection of rules for the inbound traffic to reach Services. These are Layer 7 (L7) rules that allow hostnames (and optionally paths) to be directed to specific Services in Kubernetes.
-   Ingress Controller acts upon the rules set by the Ingress Resource, typically via an HTTP or L7 load balancer. It is vital that both pieces are properly configured so that traffic can be routed from an outside client to a Kubernetes Service.

NGINX—a high performance web server—is a popular choice for an Ingress Controller because of its robustness and the many features it boasts. For example, it supports:

-   Websockets, which allows you to load balance Websocket applications.
-   SSL Services, which allows you to load balance HTTPS applications.
-   Rewrites, which allows you to rewrite the URI of a request before sending it to the application.
-   Session Persistence (NGINX Plus only), which guarantees that all the requests from the same client are always passed to the same backend container.
-   JWTs (NGINX Plus only), which allows NGINX Plus to authenticate requests by validating JSON Web Tokens (JWTs).

The following diagram illustrates the basic flow of an Ingress Controller in GCP and gives you a rough idea of what you'll be building:

![NGINX Ingress Controller on Google Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/nginx-ingress-controller-on-google-kubernetes-engine/nginx-ingress-1.png 'NGINX Ingress Controller on Google Kubernetes Engine'){: .center-image }

<br/>

## Objectives

In this lab, you will configure a Kubernetes deployment with an Ingress Resource. You will use NGINX as an Ingress Controller, which you will use to route and load balance traffic from external clients to the deployment. More specifically, you will:

-   Deploy a simple Kubernetes web application.

-   Deploy an NGINX Ingress Controller using a stable Helm Chart.

-   Deploy an Ingress Resource for the application that uses NGINX Ingress as the controller.

-   Test NGINX Ingress functionality by accessing the Google Cloud L4 (TCP/UDP) Load Balancer frontend IP and ensure it can access the web application.

<br/>

    $ gcloud config set compute/zone us-central1-a

    $ gcloud container clusters create nginx-tutorial --num-nodes 2

<br/>

### Install Helm

    $ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh

    $ chmod 700 get_helm.sh

    $ ./get_helm.sh

    $ helm init

<br/>

### Installing Tiller

    $ kubectl create serviceaccount --namespace kube-system tiller

    $ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

    $ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

    // Now initialize Helm with your newly-created service account:
    $ helm init --service-account tiller --upgrade

    $ kubectl get deployments -n kube-system

<br/>

## Deploy an application in Kubernetes Engine

    $ kubectl run hello-app --image=gcr.io/google-samples/hello-app:1.0 --port=8080

    $ kubectl expose deployment hello-app

<br/>

### Deploying the NGINX Ingress Controller via Helm

The Kubernetes platform gives administrators flexibility when it comes to Ingress Controllers—you can integrate your own rather than having to work with your provider's built-in offering. The NGINX controller must be exposed for external access. This is done using Service type: LoadBalancer on the NGINX controller service. On Kubernetes Engine, this creates a Google Cloud Network (TCP/IP) Load Balancer with NGINX controller Service as a backend. Google Cloud also creates the appropriate firewall rules within the Service's VPC to allow web HTTP(S) traffic to the load balancer frontend IP address.

<br/>

### NGINX Ingress Controller on Kubernetes Engine

The following flowchart is a visual representation of how an NGINX controller runs on a Kubernetes Engine cluster:

![NGINX Ingress Controller on Google Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/nginx-ingress-controller-on-google-kubernetes-engine/nginx-ingress-2.png 'NGINX Ingress Controller on Google Kubernetes Engine'){: .center-image }

<br/>

### Deploy NGINX Ingress Controller

    $ helm install --name nginx-ingress stable/nginx-ingress --set rbac.create=true

    $ kubectl get service nginx-ingress-controller

### Configure Ingress Resource to use NGINX Ingress Controller

    $ vi ingress-resource.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /hello
        backend:
          serviceName: hello-app
          servicePort: 8080
```

<br/>

    $ kubectl apply -f ingress-resource.yaml
    $ kubectl get ingress ingress-resource

<br/>

### Test Ingress and default backend

    $ kubectl get service nginx-ingress-controller

http://external-ip-of-ingress-controller/hello
