---
layout: page
title: Distributed Load Testing Using Kubernetes
description: Distributed Load Testing Using Kubernetes
keywords: Distributed Load Testing Using Kubernetes
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-solutions/distributed-load-testing-using-kubernetes/
---

# [GSP182] Distributed Load Testing Using Kubernetes

<br/>

Делаю:  
26.05.2019

https://www.qwiklabs.com/focuses/967?parent=catalog

<br/>

### Overview

In this lab you will learn how to use Kubernetes Engine to deploy a distributed load testing framework. The framework uses multiple containers to create load testing traffic for a simple REST-based API. Although this solution tests a simple web application, the same pattern can be used to create more complex load testing scenarios such as gaming or Internet-of-Things (IoT) applications. This solution discusses the general architecture of a container-based load testing framework.

System under test
For this lab the system under test is a small web application deployed to Google App Engine. The application exposes basic REST-style endpoints to capture incoming HTTP POST requests (incoming data is not persisted).

Example workloads
The application that you'll deploy is modeled after the backend service component found in many Internet-of-Things (IoT) deployments. Devices first register with the service and then begin reporting metrics or sensor readings, while also periodically re-registering with the service.

Common backend service component interaction looks like this:

![Distributed Load Testing Using Kubernetes](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/distributed-load-testing-using-kubernetes/pic1.png 'Distributed Load Testing Using Kubernetes'){: .center-image }

To model this interaction, you'll use Locust, a distributed, Python-based load testing tool that is capable of distributing requests across multiple target paths. For example, Locust can distribute requests to the /login and /metrics target paths.

The workload is based on the interaction described above and is modeled as a set of Tasks in Locust. To approximate real-world clients, each Locust task is weighted. For example, registration happens once per thousand total client requests.

**Container-based computing**

-   The Locust container image is a Docker image that contains the Locust software.

-   A container cluster consists of at least one cluster master and multiple worker machines called nodes. These master and node machines run the Kubernetes cluster orchestration system. For more information about clusters, see the Kubernetes Engine documentation

-   A pod is one or more containers deployed together on one host, and the smallest compute unit that can be defined, deployed, and managed. Some pods contain only a single container. For example, in this lab, each of the Locust containers runs in its own pod.

-   A Deployment controller provides declarative updates for Pods and ReplicaSets. This lab has two deployments: one for locust-master and other for locust-worker.

-   Services

A particular pod can disappear for a variety of reasons, including node failure or intentional node disruption for updates or maintenance. This means that the IP address of a pod does not provide a reliable interface for that pod. A more reliable approach would use an abstract representation of that interface that never changes, even if the underlying pod disappears and is replaced by a new pod with a different IP address. A Kubernetes Engine service provides this type of abstract interface by defining a logical set of pods and a policy for accessing them.

In this lab there are several services that represent pods or sets of pods. For example, there is a service for the DNS server pod, another service for the Locust master pod, and a service that represents all 10 Locust worker pods.

The following diagram shows the contents of the master and worker nodes:

![Distributed Load Testing Using Kubernetes](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/distributed-load-testing-using-kubernetes/pic2.png 'Distributed Load Testing Using Kubernetes'){: .center-image }

**What you'll do**

-   Create a system under test i.e. a small web application deployed to Google App Engine.
-   Use Kubernetes Engine to deploy a distributed load testing framework.
-   Create load testing traffic for a simple REST-based API.

<br/>

### Set project and zone

    $ PROJECT=$(gcloud config get-value project)
    $ REGION=us-central1
    $ ZONE=${REGION}-a
    $ CLUSTER=gke-load-test
    $ TARGET=${PROJECT}.appspot.com
    $ gcloud config set compute/region $REGION
    $ gcloud config set compute/zone $ZONE

<br/>

### Get the sample code and build a Docker image for the application

    $ git clone https://github.com/GoogleCloudPlatform/distributed-load-testing-using-kubernetes.git

    $ cd distributed-load-testing-using-kubernetes/

    $ gcloud builds submit --tag gcr.io/$PROJECT/locust-tasks:latest docker-image/.

<br/>

### Deploy Web Application

The sample-webapp folder contains a simple Google App Engine Python application as the "system under test". To deploy the application to your project use the gcloud app deploy command:

    $ gcloud app deploy sample-webapp/app.yaml

Please enter your numeric choice: 13 [Enter]

Do you want to continue (Y/n)? Y [Enter]

# Deploy Kubernetes Cluster

    $ gcloud container clusters create $CLUSTER \
      --zone $ZONE \
      --num-nodes=5

<br/>

### Load testing master

    $ sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-master-controller.yaml

    $ sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-worker-controller.yaml

    $ sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-master-controller.yaml

    $ sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-worker-controller.yaml

<br/>

    $ kubectl apply -f kubernetes-config/locust-master-controller.yaml
    $ kubectl apply -f kubernetes-config/locust-master-service.yaml

    $ kubectl get svc locust-master

<br/>

### Load testing workers

    $ kubectl apply -f kubernetes-config/locust-worker-controller.yaml
    $ kubectl scale deployment/locust-worker --replicas=20

    $ kubectl get pods

![Distributed Load Testing Using Kubernetes](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/distributed-load-testing-using-kubernetes/pic3.png 'Distributed Load Testing Using Kubernetes'){: .center-image }

<br/>

### Execute Tests

    $ EXTERNAL_IP=$(kubectl get svc locust-master -o yaml | grep ip | awk -F": " '{print $NF}')

    $ echo http://$EXTERNAL_IP:8089

![Distributed Load Testing Using Kubernetes](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/distributed-load-testing-using-kubernetes/screen-1.png 'Distributed Load Testing Using Kubernetes'){: .center-image }

<br/><br/>

![Distributed Load Testing Using Kubernetes](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/distributed-load-testing-using-kubernetes/screen-2.png 'Distributed Load Testing Using Kubernetes'){: .center-image }

<br/>

As time progress and users are spawned, you will see statistics begin to aggregate for simulation metrics, such as the number of requests and requests per second.

To stop the simulation, click Stop and the test will terminate. The complete results can be downloaded into a spreadsheet.
