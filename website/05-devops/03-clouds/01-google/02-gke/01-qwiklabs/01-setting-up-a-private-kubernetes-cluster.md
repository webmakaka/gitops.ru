---
layout: page
title: Setting up a Private Kubernetes Cluster
description: Setting up a Private Kubernetes Cluster
keywords: Setting up a Private Kubernetes Cluster
permalink: /devops/clouds/google/gke/qwiklabs/setting-up-a-private-kubernetes-cluster/
---

# [GSP178] Setting up a Private Kubernetes Cluster

<br/>

Делаю:  
02.06.2019

<br/>

In Kubernetes Engine, a private cluster is a cluster that makes your master inaccessible from the public internet. In a private cluster, nodes do not have public IP addresses, only private addresses, so your workloads run in an isolated environment. Nodes and masters communicate with each other using VPC peering.

In the Kubernetes Engine API, address ranges are expressed as Classless Inter-Domain Routing (CIDR) blocks.

<br/>

### Creating a private cluster

    $ gcloud config set compute/zone us-central1-a

When you create a private cluster, you must specify a /28 CIDR range for the VMs that run the Kubernetes master components and you need to enable IP aliases.

Next you'll create a cluster named private-cluster, and specify a CIDR range of 172.16.0.16/28 for the masters. When you enable IP aliases, you let Kubernetes Engine automatically create a subnetwork for you.

You'll create the private cluster by using the --private-cluster, --master-ipv4-cidr, and --enable-ip-alias flags.

<br/>

    $ gcloud beta container clusters create private-cluster \
        --private-cluster \
        --master-ipv4-cidr 172.16.0.16/28 \
        --enable-ip-alias \
        --create-subnetwork ""

<br/>

    $ gcloud compute networks subnets list --network default
    ***
    gke-private-cluster-subnet-7135c381  us-central1              default  10.33.40.0/22

    // $ gcloud compute networks subnets describe [SUBNET_NAME] --region us-central1
    $ gcloud compute networks subnets describe gke-private-cluster-subnet-7135c381 --region us-central1

<br/>

### Enabling master authorized networks

    $ gcloud compute instances create source-instance --zone us-central1-a --scopes 'https://www.googleapis.com/auth/cloud-platform'

    $ gcloud compute instances describe source-instance --zone us-central1-a | grep natIP

    natIP: 104.198.63.250

    [MY_EXTERNAL_RANGE] - natIP/32

    // $ gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks [MY_EXTERNAL_RANGE]

    $ gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks 104.198.63.250/32


    // Enter through the passphrase questions.
    $ gcloud compute ssh source-instance --zone us-central1-a

В виртуалке:

    $ sudo apt-get install -y kubectl

<!--
$ gcloud components install kubectl
-->

    $ gcloud container clusters get-credentials private-cluster --zone us-central1-a

    $ kubectl get nodes --output wide
    NAME                                             STATUS   ROLES    AGE     VERSION          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
    gke-private-cluster-default-pool-7bbe0b39-6gz4   Ready    <none>   8m25s   v1.12.7-gke.10   10.33.40.3                  Container-Optimized OS from Google   4.14.106+        docker://17.3.2
    gke-private-cluster-default-pool-7bbe0b39-hrxb   Ready    <none>   8m26s   v1.12.7-gke.10   10.33.40.4                  Container-Optimized OS from Google   4.14.106+        docker://17.3.2
    gke-private-cluster-default-pool-7bbe0b39-rjpd   Ready    <none>   8m24s   v1.12.7-gke.10   10.33.40.2                  Container-Optimized OS from Google   4.14.106+        docker://17.3.2

    $ exit

<br/>

### Clean Up

    $ gcloud container clusters delete private-cluster --zone us-central1-a

<br/>

## Creating a private cluster that uses a custom subnetwork (Optional)

In the previous section Kubernetes Engine automatically created a subnetwork for you. In this section, you'll create your own custom subnetwork, and then create a private cluster. Your subnetwork has a primary address range and two secondary address ranges.

Create a subnetwork and secondary ranges:

    $ gcloud compute networks subnets create my-subnet \
        --network default \
        --range 10.0.4.0/22 \
        --enable-private-ip-google-access \
        --region us-central1 \
        --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14

<br/>

Create a private cluster that uses your subnetwork:

    $ gcloud beta container clusters create private-cluster2 \
        --private-cluster \
        --enable-ip-alias \
        --master-ipv4-cidr 172.16.0.32/28 \
        --subnetwork my-subnet \
        --services-secondary-range-name my-svc-range \
        --cluster-secondary-range-name my-pod-range

<br/>

Authorize your external address range, replacing [MY_EXTERNAL_RANGE] with the CIDR range of the external addresses from the previous output:

    // $ gcloud container clusters update private-cluster2 \
        --enable-master-authorized-networks \
        --master-authorized-networks [MY_EXTERNAL_RANGE]

    $ gcloud container clusters update private-cluster2 \
        --enable-master-authorized-networks \
        --master-authorized-networks 104.198.63.250/32


    $ gcloud compute ssh source-instance --zone us-central1-a

    $ gcloud container clusters get-credentials private-cluster2 --zone us-central1-a

    $ kubectl get nodes --output yaml | grep -A4 addresses
          addresses:
          - address: 10.0.4.4
            type: InternalIP
          - address: ""
            type: ExternalIP
      --
          addresses:
          - address: 10.0.4.2
            type: InternalIP
          - address: ""
            type: ExternalIP
      --
          addresses:
          - address: 10.0.4.3
            type: InternalIP
          - address: ""
            type: ExternalIP

<br/>

At this point, the only IP addresses that have access to the master are the addresses in these ranges:

-   The primary range of your subnetwork. This is the range used for nodes. In this example, the range for nodes is 10.0.4.0/22.

-   The secondary range of your subnetwork that is used for pods. In this example, the range for pods is 10.4.0.0/14.
