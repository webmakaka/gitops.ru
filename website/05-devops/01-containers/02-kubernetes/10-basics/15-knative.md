---
layout: page
title: Kubernetes KNative
description: Kubernetes KNative
keywords: devops, linux, kubernetes, Custom Resource Definitions, KNative
permalink: /devops/containers/kubernetes/basics/knative/
---

# Kubernetes KNative

**Примеры из курса "11 Steps to Awesome with Kubernetes, Istio, and Knative LiveLessons"**

<br/>

Делаю:
23.04.2020

<br/>

// banch 0.7
https://github.com/redhat-developer-demos/knative-tutorial

<br/>

https://github.com/burrsutter/scripts-knative

<br/>

https://github.com/burrsutter/sidebyside

<br/>

С v1.16.9 были какие-то проблемы. Не заработало.  
v1.14.0 норм.

<br/>

```
$ {
minikube --profile my-profile config set memory 8192
minikube --profile my-profile config set cpus 4
minikube --profile my-profile config set disk-size 50g

minikube --profile my-profile config set vm-driver virtualbox
// minikube --profile my-profile config set vm-driver docker

// minikube --profile my-profile config set kubernetes-version v1.16.9
minikube --profile my-profile config set kubernetes-version v1.14.0


minikube start --profile my-profile  --extra-config='apiserver.enable-admission-plugins=LimitRanger,NamespaceExists,NamespaceLifecycle,ResourceQuota,ServiceAccount,DefaultStorageClass,MutatingAdmissionWebhook' \
  --insecure-registry='10.0.0.0/24'

}
```

<br/>

    $ minikube addons enable registry

<br/>

## Learn Knative serverless capabilities for Kubernetes

**По следующей доке:**  
https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-basics/0.7.x/01-setup.html

<br/>

### Install Istio

    $ kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v0.7.1/third_party/istio-1.1.7/istio-crds.yaml

    $ kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v0.7.1/third_party/istio-1.1.7/istio-lean.yaml

<br/>

Если нужно конвертануть:

    $ cd ~/tmp
    $ wget https://raw.githubusercontent.com/knative/serving/v0.7.1/third_party/istio-1.1.7/istio-lean.yaml

    $ kubectl convert -f istio-lean.yaml > istio-lean1.yaml

    $ kubectl apply -f istio-lean1.yaml

<br/>

    $ kubectl get pods -n istio-system
    NAME                                     READY   STATUS      RESTARTS   AGE
    cluster-local-gateway-644b4689c9-zpqbj   1/1     Running     0          11m
    istio-ingressgateway-65648c4585-rtcpv    2/2     Running     0          11m
    istio-init-crd-10-qqggl                  0/1     Completed   0          20m
    istio-init-crd-11-6786q                  0/1     Completed   0          20m
    istio-pilot-5bdbcdd9bc-84glc             1/1     Running     0          11m

<br/>

### Install Custom Resource Definitions

    $ kubectl apply --selector knative.dev/crd-install=true \
      --filename https://github.com/knative/serving/releases/download/v0.7.1/serving.yaml \
      --filename https://github.com/knative/eventing/releases/download/v0.7.1/release.yaml

<br/>

    $ kubectl get crds | grep knative
    apiserversources.sources.eventing.knative.dev        2020-04-23T01:44:33Z
    brokers.eventing.knative.dev                         2020-04-23T01:44:33Z
    certificates.networking.internal.knative.dev         2020-04-23T01:44:33Z
    channels.eventing.knative.dev                        2020-04-23T01:44:33Z
    clusterchannelprovisioners.eventing.knative.dev      2020-04-23T01:44:33Z
    clusteringresses.networking.internal.knative.dev     2020-04-23T01:44:33Z
    configurations.serving.knative.dev                   2020-04-23T01:44:33Z
    containersources.sources.eventing.knative.dev        2020-04-23T01:44:33Z
    cronjobsources.sources.eventing.knative.dev          2020-04-23T01:44:33Z
    eventtypes.eventing.knative.dev                      2020-04-23T01:44:33Z
    images.caching.internal.knative.dev                  2020-04-23T01:44:33Z
    ingresses.networking.internal.knative.dev            2020-04-23T01:44:33Z
    inmemorychannels.messaging.knative.dev               2020-04-23T01:44:33Z
    podautoscalers.autoscaling.internal.knative.dev      2020-04-23T01:44:33Z
    revisions.serving.knative.dev                        2020-04-23T01:44:33Z
    routes.serving.knative.dev                           2020-04-23T01:44:33Z
    sequences.messaging.knative.dev                      2020-04-23T01:44:33Z
    serverlessservices.networking.internal.knative.dev   2020-04-23T01:44:33Z
    services.serving.knative.dev                         2020-04-23T01:44:33Z
    subscriptions.eventing.knative.dev                   2020-04-23T01:44:33Z
    triggers.eventing.knative.dev                        2020-04-23T01:44:33Z

<br/>

    $ kubectl apply --selector networking.knative.dev/certificate-provider!=cert-manager \
      --filename https://github.com/knative/serving/releases/download/v0.7.1/serving.yaml

<br/>

    $ kubectl get pods -n knative-serving
    NAME                                READY   STATUS    RESTARTS   AGE
    activator-7d57675b9b-pf2qp          1/1     Running   0          47s
    autoscaler-f6f5494ff-8xl6r          1/1     Running   0          47s
    controller-866bd9f69d-9b82q         1/1     Running   0          47s
    networking-istio-849bd546ff-snztb   1/1     Running   0          47s
    webhook-7884ddd64f-sn2tx            1/1     Running   0          47s

<br/>

    $ kubectl apply --selector networking.knative.dev/certificate-provider!=cert-manager \
    --filename https://github.com/knative/eventing/releases/download/v0.7.1/release.yaml

<br/>

    $ kubectl get pods -n knative-eventing
    NAME                                           READY   STATUS    RESTARTS   AGE
    eventing-controller-7b9f49d46c-xrfxz           1/1     Running   0          44s
    eventing-webhook-d55865985-zdj6f               1/1     Running   0          43s
    imc-controller-65cc458779-cntbk                1/1     Running   0          43s
    imc-dispatcher-d5c5bf954-j2wm9                 1/1     Running   0          43s
    in-memory-channel-controller-9cd44c656-hwdj5   1/1     Running   0          43s
    in-memory-channel-dispatcher-98d7fcdf6-5fkf5   1/1     Running   0          43s
    sources-controller-84b9cb88cd-6cwh5            1/1     Running   0          44s

<br/>

## Learn Knative Serving

    $ kubectl get crd | grep serving
    configurations.serving.knative.dev                   2020-04-23T00:28:05Z
    revisions.serving.knative.dev                        2020-04-23T00:28:05Z
    routes.serving.knative.dev                           2020-04-23T00:28:05Z
    services.serving.knative.dev                         2020-04-23T00:28:05Z

<br/>

## Prepare and deploy a Knative Service

    $ kubectl create namespace knativetutorial
    $ kubectl config set-context --current --namespace=knativetutorial

```
$ cat <<EOF | kubectl apply -f -
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: greeter
spec:
  template:
    metadata:
      name: greeter-v1
      annotations:
        # disable istio-proxy injection
        sidecar.istio.io/inject: "false"
    spec:
      containers:
      - image: quay.io/rhdevelopers/knative-tutorial-greeter:quarkus
        livenessProbe:
          httpGet:
            path: /healthz
        readinessProbe:
          httpGet:
            path: /healthz
EOF
```

<br/>

    $ kubectl get all | grep in-memory
    clusterchannelprovisioner.eventing.knative.dev/in-memory   True             20m

<br/>

    $ watch kubectl get pods

<br/>

    $ kubectl get deployment

<br/>

    $ kubectl exec -it greeter-v1-deployment-567fd49555-mxfkb /bin/bash

    $ curl localhost:8080
    $ curl localhost:8080
    $ curl localhost:8080
    $ curl localhost:8080
    $ curl localhost:8080

<br/>

    $ kubectl get ksvc
    NAME      URL                                          LATESTCREATED   LATESTREADY   READY   REASON
    greeter   http://greeter.knativetutorial.example.com   greeter-v1      greeter-v1    True

    $ kubectl describe ksvc
    $ kubectl describe ksvc greeter

<br/>

    $ IP_ADDRESS=$(minikube  --profile my-profile ip):$(kubectl get svc istio-ingressgateway --namespace istio-system --output 'jsonpath={.spec.ports[?(@.port==80)].nodePort}')

    $ echo ${IP_ADDRESS}
    192.168.99.124:30667


    $ kubectl get ksvc
    NAME      URL                                          LATESTCREATED   LATESTREADY   READY   REASON
    greeter   http://greeter.knativetutorial.example.com   greeter-v1      greeter-v1    True

<br/>

    $ while true; do curl $IP_ADDRESS -H 'Host:greeter.knativetutorial.example.com'; sleep .3; done

<br/>

Поды поднялись!

<br/>

## Autoscaling HTTP

\$ kubectl delete ksvc greeter

https://github.com/redhat-developer-demos/knative-tutorial/blob/release/0.7.x/04-scaling/knative/service-10.yaml

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: prime-generator
spec:
  template:
    metadata:
      name: prime-generator-v1
      annotations:
        # disable istio-proxy injection
        sidecar.istio.io/inject: "false"
        # Target 10 in-flight-requests per pod.
        autoscaling.knative.dev/target: "10"
    spec:
      containers:
      - image: quay.io/rhdevelopers/prime-generator:v27-quarkus
        livenessProbe:
          httpGet:
            path: /healthz
        readinessProbe:
          httpGet:
            path: /healthz
EOF
```

<br/>

\$ kubectl get deployments
NAME READY UP-TO-DATE AVAILABLE AGE
prime-generator-v1-deployment 0/0 0 0 2m34s

<br/>

Deployment поломалась. Ничего не работает.

<br/>

\$ kubectl get events

<br/>

2m48s Warning InternalError route/prime-generator Operation cannot be fulfilled on clusteringresses.networking.internal.knative.dev "route-bb5458d0-8506-11ea-ad4a-0800277556f1": the object has been modified; please apply your changes to the latest version and try again

<br/>

    $ IP_ADDRESS=$(minikube  --profile my-profile ip):$(kubectl get svc istio-ingressgateway --namespace istio-system --output 'jsonpath={.spec.ports[?(@.port==80)].nodePort}')

    $ echo ${IP_ADDRESS}
    192.168.99.124:30667

<br/>

    $ while true; do curl $IP_ADDRESS -H 'Host:greeter.knativetutorial.example.com'; sleep .3; done

<br/>

\$ kubectl delete ksvc prime-generator

<br/>
