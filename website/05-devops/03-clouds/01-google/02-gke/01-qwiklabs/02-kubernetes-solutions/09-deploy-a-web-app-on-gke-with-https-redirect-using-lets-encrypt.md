---
layout: page
title: Deploy a Web App on GKE with HTTPS Redirect using Lets Encrypt
description: Deploy a Web App on GKE with HTTPS Redirect using Lets Encrypt
keywords: Deploy a Web App on GKE with HTTPS Redirect using Lets Encrypt
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-solutions/deploy-a-web-app-on-gke-with-https-redirect-using-lets-encrypt/
---

# [GSP269] Deploy a Web App on GKE with HTTPS Redirect using Lets Encrypt

<br/>

Делаю:  
31.05.2019

https://www.qwiklabs.com/focuses/2771?parent=catalog

<br/>

    $ wget https://storage.googleapis.com/vwebb-codelabs/gke-tls-qwik/gke-tls-lab.tar.gz

    $ tar zxfv gke-tls-lab.tar.gz

    $ cd gke-tls-lab

<br/>

### Configure Cloud Endpoints

    $ gcloud compute addresses create endpoints-ip --region us-central1

    $ gcloud compute addresses list
      endpoints-ip  34.66.27.77

<br/>

    $ export PROJECT_ID=$(gcloud config get-value project)
    $ export ENDPOINTS_IP=34.66.27.77

    $ sed -i "s/\[MY-PROJECT\]/$PROJECT_ID/g" ./openapi.yaml
    $ sed -i "s/\[MY-STATIC-IP\]/$ENDPOINTS_IP/g" ./openapi.yaml

    // Deploy to Cloud Endpoints:
    $ gcloud endpoints services deploy openapi.yaml

<br/>

### Create a Kubernetes Engine Cluster

    $ gcloud container clusters create cl-cluster --zone us-central1-f
    $ gcloud container clusters get-credentials cl-cluster --zone us-central1-f

<br/>

### Set up Role-Based Access Control

To be able to deploy to the cluster, you need the proper permissions.

    $ kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole cluster-admin --user $(gcloud config get-value account)

<br/>

### Install Helm

    $ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh

    $ chmod 700 get_helm.sh

    $ ./get_helm.sh

<br/>

    $ kubectl create serviceaccount -n kube-system tiller

    $ kubectl create clusterrolebinding tiller-binding \
        --clusterrole=cluster-admin \
        --serviceaccount kube-system:tiller

    $ helm init --service-account tiller

    $ helm repo update

<br/>

### Install NGINX Ingress

You will deploy an NGINX ingress using Helm to handle our HTTP to HTTPS redirect when configure our web app for HTTPS to ensure the user always has a secure connection to our app.

    $ helm install stable/nginx-ingress --set controller.service.loadBalancerIP=${ENDPOINTS_IP},rbac.create=true

<br/>

### Deploy "Hello World" App

    $ sed -i "s/\[MY-PROJECT\]/$PROJECT_ID/g" ./configmap.yaml
    $ sed -i "s/\[MY-PROJECT\]/$PROJECT_ID/g" ./ingress.yaml

<br/>

### Deploy web app to cluster

    $ kubectl apply -f configmap.yaml
    $ kubectl apply -f deployment.yaml
    $ kubectl apply -f service.yaml
    $ kubectl apply -f ingress.yaml

    $ echo http://api.endpoints.${PROJECT_ID}.cloud.goog

<br/>

### Set Up HTTPS

    $ helm install --name cert-manager --version v0.3.2 --namespace kube-system stable/cert-manager

    $ export EMAIL=ahmet@example.com

    $ cat letsencrypt-issuer.yaml | sed -e "s/email: ''/email: $EMAIL/g" | kubectl apply -f-

<br/>

### Reconfigure ingress for HTTPS

    $ sed -i "s/\[MY-PROJECT\]/$PROJECT_ID/g" ./ingress-tls.yaml

    $ kubectl apply -f ingress-tls.yaml

    $ kubectl describe ingress esp-ingress

    $ echo http://api.endpoints.${PROJECT_ID}.cloud.goog

Note: It might take 5-10 minutes for the ingress to be properly provisioned.
