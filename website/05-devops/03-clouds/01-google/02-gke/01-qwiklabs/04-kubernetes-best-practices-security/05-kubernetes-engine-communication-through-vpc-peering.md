---
layout: page
title: Kubernetes Engine Communication Through VPC Peering
description: Kubernetes Engine Communication Through VPC Peering
keywords: Kubernetes Engine Communication Through VPC Peering
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-best-practices-security/kubernetes-engine-communication-through-vpc-peering/
---

# [GSP476] Kubernetes Engine Communication Through VPC Peering

<br/>

Делаю:  
09.06.2019

https://www.qwiklabs.com/focuses/5540?parent=catalog

<br/>

### Architecture

The execution of this code in the GCP environment creates two custom GCP networks connected via VPC peering. Each network will have two subnets - one in the us-central1 region and the other in the us-east1 region. Each of the subnets hosts a Kubernetes Engine cluster which has nginx pods and services to expose those pods across other clusters.

![Kubernetes Engine Communication Through VPC Peering](/img/devops/clouds/google/gke/qwiklabs/kubernetes-best-practices-security/kubernetes-engine-communication-through-vpc-peering/pic1.png 'Kubernetes Engine Communication Through VPC Peering'){: .center-image }

<br>

### Lab setup

    $ git clone https://github.com/GoogleCloudPlatform/gke-networking-demos.git
    $ cd gke-networking-demos
    $ cd gke-to-gke-peering
    $ ./install.sh

    $ gcloud compute instances list
    NAME                                                 ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
    gke-cluster-deployment-c-default-pool-fb67005b-m4sn  us-central1-b  n1-standard-1               10.2.0.2     35.226.103.83   RUNNING
    gke-cluster-deployment-c-default-pool-8c916bfd-8b8q  us-central1-c  n1-standard-1               10.12.0.2    35.225.122.88   RUNNING
    gke-cluster-deployment-c-default-pool-85307135-2rtd  us-east1-c     n1-standard-1               10.11.0.2    35.185.120.193  RUNNING
    gke-cluster-deployment-c-default-pool-eba8fd20-mw4v  us-east1-d     n1-standard-1               10.1.0.2     34.74.163.157   RUNNING

<br/>

    $ kubectl get nodes
    NAME                                                  STATUS   ROLES    AGE     VERSION
    gke-cluster-deployment-c-default-pool-8c916bfd-8b8q   Ready    <none>   9m43s   v1.12.8-gke.6

<br/>

    $ kubectl get pods
    NAME                        READY   STATUS    RESTARTS   AGE
    my-nginx-75766448f7-w6pg6   1/1     Running   0          7m32s
    my-nginx-75766448f7-z7zxd   1/1     Running   0          7m32s

<br/>

    $ kubectl get svc
    NAME                TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                        AGE
    kubernetes          ClusterIP      10.238.0.1      <none>           443/TCP                        11m
    my-nginx            ClusterIP      10.238.5.108    <none>           80/TCP                         8m2s
    my-nginx-lb         LoadBalancer   10.238.1.19     35.232.246.213   8080:32364/TCP,443:31161/TCP   8m
    my-nginx-nodeport   NodePort       10.238.12.216   <none>           8080:32577/TCP,443:30012/TCP   8m1s

<br/>

http://35.232.246.213:8080/

<br/>

### Validation

To make sure that there are no errors in the install script execution, go to the GCP Console.

1.  Verify that the CIDR ranges of subnet-us-west1 and subnet-us-east1 matches the specification.
2.  Click on Compute Engine > VM instances and verify that the node IP addresses are drawn from the subnet's CIDR.
3.  Click on Kubernetes Engine > Clusters to verify the 4 created clusters. Click on the cluster hyperlink and verify that "Service address range" matches the specified cluster-ipv4-cidr.
4.  Still on the Kubernetes Engine page, click on Workloads and verify that the status is OK for nginx pods.
5.  Now click on Services. Verify that the cluster ip nodeport, internal load balancer (ILB) and load balancer (LB) are created for cluster1.
6.  Verify that the cluster ip nodeport, LB and ingress services are created for cluster2.
7.  Verify that cluster IP address of all the services for a cluster are drawn from service-ipv4-cidr.
8.  Access the endpoint for URL for external load balancer to view the nginx pods.
    Still in the gke-to-gke-peering directory, run the validation script:

        $ ./validate.sh

<br/>

### Verify the pod-to-service communication

Next you will run a pod-to-service validation script that does the following:

1. Clusters in the same region communicate through the internal load balancer.
2. Clusters across the different regions communicate through the global load balancer.
3. All the services created to expose pods in a cluster are accessible to pods within that cluster.
4. Refer to validate-pod-to-service-communication.sh script to view the commands to verify pod to service communication.

Change to the project root directory, gke-networking-demos:

    $ cd ..

Run the following,

    $ ./validate-pod-to-service-communication.sh

This script demonstrates how the pods in cluster1 can access the local Kubernetes Engine services and the other Kubernetes Engine Internal/External load balancer services from the same or different regions.

<br/>

### Tear Down

    $ cd gke-to-gke-peering
    $ ./cleanup.sh
