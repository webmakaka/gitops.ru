---
layout: page
title: VPC Network Peering
description: VPC Network Peering
keywords: VPC Network Peering
permalink: /devops/clouds/google/gce/qwiklabs/vpc-network-peering/
---

# [GSP007] VPC Network Peering

<br/>

Делаю!  
31.05.2019

<br/>

https://www.qwiklabs.com/focuses/964?parent=catalog

<br/>

Открываем 2 консоли.
Одна для project1, другая для project2

<br/>

**Project-A:**

    $ gcloud config set project <PROJECT_ID1>

    // Create a custom network:
    $ gcloud compute networks create network-a --subnet-mode custom

    // Create a subnet within this VPC and specify a region and IP range
    $ gcloud compute networks subnets create network-a-central --network network-a \
        --range 10.0.0.0/16 --region us-central1

    // Create a VM instance
    $ gcloud compute instances create vm-a --zone us-central1-a --network network-a --subnet network-a-central

    // Run the following to enable SSH and icmp
    $ gcloud compute firewall-rules create network-a-fw --network network-a --allow tcp:22,icmp

<br/>

**Project-B:**

    $ gcloud config set project <PROJECT_ID2>

    $ gcloud compute networks create network-b --subnet-mode custom

    $ gcloud compute networks subnets create network-b-central --network network-b \
    --range 10.8.0.0/16 --region us-central1

    $ gcloud compute instances create vm-b --zone us-central1-a --network network-b --subnet network-b-central

    $ gcloud compute firewall-rules create network-b-fw --network network-b --allow tcp:22,icmp

<br/>

**Project-A:**

VPC Network > VPC network peering

1. Click Create connection.
2. Click Continue.
3. Type "peer-ab" as the Name for this side of the connection.
4. Under Your VPC network, select the network you want to peer (network-a).
5. Set the Peered VPC network radio buttons to In another project.
6. Paste in the Project ID of the second project.
7. Type in the VPC network name of the other network (network-b).
8. Click Create.

<br/>

**Project-B:**

VPC Network > VPC network peering

1. Click Create connection.
2. Click Continue.
3. Type "peer-ba" as the Name for this side of the connection.
4. Under Your VPC network, select the network you want to peer (network-b).
5. Set the Peering VPC network radio buttons to In another project, unless you wish to peer within the same project.
6. Specify the Project ID of the first project.
7. Specify VPC network name of the other network (network-a).
8. Click Create.

<br/>

    $ gcloud compute routes list --project <FIRST_PROJECT_ID>

<br/>

### Connectivity Test

Project-A

Navigation Menu > Compute Engine > VM instances.

Copy the INTERNAL_IP for vm-a. (35.193.65.173)

Project-B

    // $ ping -c 5 <INTERNAL_IP_OF_VM_A>
    $ ping -c 5 35.193.65.173
