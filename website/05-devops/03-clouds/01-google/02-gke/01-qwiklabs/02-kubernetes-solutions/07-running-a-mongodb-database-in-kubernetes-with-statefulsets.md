---
layout: page
title: Running a MongoDB Database in Kubernetes with StatefulSets
description: Running a MongoDB Database in Kubernetes with StatefulSets
keywords: Running a MongoDB Database in Kubernetes with StatefulSets
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-solutions/running-a-mongodb-database-in-kubernetes-with-statefulsets/
---

# [GSP022] Running a MongoDB Database in Kubernetes with StatefulSets

<br/>

Делаю:  
29.05.2019

https://www.qwiklabs.com/focuses/640?parent=catalog

<br/>

### Create a new Cluster

    $ gcloud config set compute/zone us-central1-f

    $ gcloud container clusters create hello-world

<br/>

### Plan

-   Download the MongoDB replica set/sidecar (or utility container in our cluster).
-   Instantiate a StorageClass.
-   Instantiate a headless service.
-   Instantiate a StatefulSet.

<br/>

###

    $ git clone https://github.com/thesandlord/mongo-k8s-sidecar.git

    $ cd ./mongo-k8s-sidecar/example/StatefulSet/

<br/>

### Create the StorageClass

A StorageClass tells Kubernetes what kind of storage you want to use for database nodes. On the Google Cloud Platform, you have a couple of storage choices: SSDs and hard disks.

If you take a look inside the StatefulSet directory (you can do this by running the ls command), you will see SSD and HDD configuration files for both Azure and GCP. Run the following command to take a look at the googlecloud_ssd.yaml file

    $ kubectl apply -f googlecloud_ssd.yaml

<br/>

### Deploying the Headless Service and StatefulSet

    $ cat mongo-statefulset.yaml

**Headless service: overview**

The first section of mongo-statefulset.yaml refers to a headless service. In Kubernetes terms, a service describes policies or rules for accessing specific pods. In brief, a headless service is one that doesn't prescribe load balancing. When combined with StatefulSets, this will give us individual DNSs to access our pods, and in turn a way to connect to all of our MongoDB nodes individually. In the yaml file, you can make sure that the service is headless by verifying that the clusterIP field is set to None.

**StatefulSet: overview**

The StatefulSet configuration is the second section of mongo-statefulset.yaml. This is the bread and butter of the application: it's the workload that runs MongoDB and what orchestrates your Kubernetes resources. Referencing the yaml file, we see that the first section describes the StatefulSet object. Then, we move into the Metadata section, where labels and the number of replicas are specified.

Next comes the pod spec. The terminationGracePeriodSeconds is used to gracefully shutdown the pod when you scale down the number of replicas. Then the configurations for the two containers are shown. The first one runs MongoDB with command line flags that configure the replica set name. It also mounts the persistent storage volume to /data/db: the location where MongoDB saves its data. The second container runs the sidecar. This sidecar container will configure the MongoDB replica set automatically. As mentioned earlier, a "sidecar" is a helper container that helps the main container run its jobs and tasks.

Finally, there is the volumeClaimTemplates. This is what talks to the StorageClass we created before to provision the volume. It provisions a 100 GB disk for each MongoDB replica.

    $ kubectl apply -f mongo-statefulset.yaml

<br/>

### Connect to the MongoDB Replica Set

    $ kubectl get statefulset
    $ kubectl get pods
    NAME      READY   STATUS    RESTARTS   AGE
    mongo-0   2/2     Running   0          2m22s
    mongo-1   2/2     Running   0          90s
    mongo-2   2/2     Running   0          61s

<br/>

    $ kubectl exec -ti mongo-0 mongo

    // Instantiate the replica set with a default configuration
    > rs.initiate()

    // Print the replica set configuratio
    > rs.conf()

    > exit

<br/>

### Scaling the MongoDB replica set

    $ kubectl scale --replicas=5 statefulset mongo
    $ kubectl get pods
    NAME      READY   STATUS    RESTARTS   AGE
    mongo-0   2/2     Running   0          4m49s
    mongo-1   2/2     Running   0          3m57s
    mongo-2   2/2     Running   0          3m28s
    mongo-3   2/2     Running   0          87s
    mongo-4   2/2     Running   0          36s

<br/>

### Using the MongoDB replica set

Each pod in a StatefulSet backed by a Headless Service will have a stable DNS name. The template follows this format: <pod-name>.<service-name>

This means the DNS names for the MongoDB replica set are:

    mongo-0.mongo
    mongo-1.mongo
    mongo-2.mongo

You can use these names directly in the connection string URI of your app.

Using a database is outside the scope of this lab, however for this case, the connection string URI would be:

    "mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo:27017/dbname_?"

<br/>

### Clean up

    $ kubectl delete statefulset mongo
    $ kubectl delete svc mongo
    $ kubectl delete pvc -l role=mongo
    $ gcloud container clusters delete "hello-world"
