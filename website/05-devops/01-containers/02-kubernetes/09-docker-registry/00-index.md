---
layout: page
title: Kubernetes Docker Registry
description: Kubernetes Docker Registry
keywords: linux, kubernetes, Docker Registry
permalink: /devops/containers/kubernetes/docker-registry/
---

# Kubernetes Docker Registry

**Из примера с katacoda:**  
https://www.katacoda.com/javajon/courses/kubernetes-pipelines/tekton

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

### Clone Example Node.js App

    $ cd ~/tmp/
    $ git clone https://github.com/javajon/node-js-tekton

    $ cd node-js-tekton

<br/>

### Declare Service Account

    $ kubectl apply -f pipeline/service-account.yaml

<br/>

    $ kubectl get ServiceAccounts
    NAME              SECRETS   AGE
    default           1         52m
    service-account   1         16s

### Declare Pipeline Resources

**pipeline/git-resource.yaml**

```
$ cat << EOF | kubectl apply -f -
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: git
spec:
  type: git
  params:
    - name: revision
      value: master
    - name: url
      value: https://github.com/javajon/node-js-tekton
EOF
```

<br/>

    $ tkn resources list
    NAME   TYPE   DETAILS
    git    git    url: https://github.com/javajon/node-js-tekton
    marley@workstation:~/projects/dev/devops/voting-tekton$

<br/>

### Declare Pipeline Tasks

For our pipeline, we have defined two tasks.

-   task-build-src clones the source, builds the Node.js based container, and pushed the image to a registry.
-   task-deploy pulls the container image from the private registry and runs it on this Kubernetes cluster.

<br/>

    $ kubectl apply -f pipeline/task-build-src.yaml
    $ kubectl apply -f pipeline/task-deploy.yaml

<br/>

    $ tkn tasks list
    NAME                      AGE
    build-image-from-source   1 minute ago
    deploy-application        2 seconds ago

<br/>

### Declare Pipeline

    $ kubectl apply -f pipeline/pipeline.yaml

    $ tkn pipelines list
    NAME                   AGE              LAST RUN   STARTED   DURATION   STATUS
    application-pipeline   13 seconds ago   ---        ---       ---        ---

<br/>

### Declare Runner for Pipeline

**pipeline/pipeline-run.yaml**

```
$ cat << EOF | kubectl apply -f -
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: application-pipeline-run
spec:
  pipelineRef:
    name: application-pipeline
  resources:
    - name: git-source
      resourceRef:
        name: git
  params:
    - name: pathToContext
      value: "src"
    - name: pathToYamlFile
      value: "deploy.yaml"
    - name: "imageUrl"
      value: "private-docker-registry.kube-system:5000/app"
    - name: "imageTag"
      value: "0.0.1"
  serviceAccountName: service-account
EOF
```

<br/>

    $ tkn pipelineruns list
    NAME                       STARTED          DURATION   STATUS
    application-pipeline-run   56 seconds ago   ---        Running

<br/>

```
$ tkn pipelineruns describe application-pipeline-run
Name:              application-pipeline-run
Namespace:         default
Pipeline Ref:      application-pipeline
Service Account:   service-account

🌡️  Status

STARTED        DURATION   STATUS
1 minute ago   1 minute   Succeeded

📦 Resources

 NAME           RESOURCE REF
 ∙ git-source   git

⚓ Params

 NAME               VALUE
 ∙ pathToContext    src
 ∙ pathToYamlFile   deploy.yaml
 ∙ imageUrl         private-docker-registry.kube-system:5000/app
 ∙ imageTag         0.0.1

🗂  Taskruns

 NAME                                                       TASK NAME                 STARTED          DURATION     STATUS
 ∙ application-pipeline-run-deploy-application-qd79d        deploy-application        36 seconds ago   18 seconds   Succeeded
 ∙ application-pipeline-run-build-image-from-source-txgvx   build-image-from-source   1 minute ago     52 seconds   Succeeded
```

<br/>

### Access Application

```
$ kubectl get deployments,pods,services
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/app   3/3     3            3           62s

NAME                                                                  READY   STATUS      RESTARTS   AGE
pod/app-79fd75857b-9smdc                                              1/1     Running     0          62s
pod/app-79fd75857b-jq7xx                                              1/1     Running     0          62s
pod/app-79fd75857b-rk4x8                                              1/1     Running     0          62s
pod/application-pipeline-run-build-image-from-source-txgvx-po-mgvwm   0/3     Completed   0          2m13s
pod/application-pipeline-run-deploy-application-qd79d-pod-2wg2w       0/3     Completed   0          80s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/app          NodePort    10.96.65.174   <none>        8080:32000/TCP   62s
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          12m
```

<br/>

Приложение доступно:

http://192.168.99.130:32000/
