---
layout: page
title: Running Dedicated Game Servers in Google Kubernetes Engine
description: Running Dedicated Game Servers in Google Kubernetes Engine
keywords: Running Dedicated Game Servers in Google Kubernetes Engine
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-solutions/running-dedicated-game-servers-in-google-kubernetes-engine/
---

# [GSP133] Running Dedicated Game Servers in Google Kubernetes Engine

<br/>

Делаю:  
27.05.2019

https://www.qwiklabs.com/focuses/617?parent=catalog

<br/>

![Running Dedicated Game Servers in Google Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/running-dedicated-game-servers-in-google-kubernetes-engine/app.png 'Running Dedicated Game Servers in Google Kubernetes Engine'){: .center-image }

<br/>

### Creating a dedicated game server binaries container image

Google Cloud Console --> Compute Engine --> VM Instances --> Create

Identity and API access --> Allow full access to all Cloud APIs --> Create

Click the SSH button. The remaining tasks for this lab will be carried out in the SSH console of this VM.

<br/>

### Install kubectl and docker on your VM

    $ sudo apt-get update
    $ sudo apt-get -y install kubectl

    $ sudo apt-get -y install \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg2 \
      software-properties-common

    $ curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -

    $ sudo apt-key fingerprint 0EBFCD88

    $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")    $(lsb_release -cs) stable"

    $ sudo apt-get update
    $ sudo apt-get -y install docker-ce

    $ sudo usermod -aG docker $USER
    $ logout

Exit from the SSH session then reconnect by clicking the SSH button for your VM instance.

    $ docker run hello-world

<br/>

### Download the Sample Game Server Demo

    $ git clone https://github.com/GoogleCloudPlatform/gke-dedicated-game-server.git

<br/>

## Creating a dedicated game server binaries container image

    $ export PROJECT_ID=$(gcloud config get-value project)
    $ export GCR_REGION=us
    $ printf "$GCR_REGION \n$PROJECT_ID\n"

<br/>

### Generate a new container image

    $ cd gke-dedicated-game-server/openarena
    $ docker build -t ${GCR_REGION}.gcr.io/${PROJECT_ID}/openarena:0.8.8 .
    $ gcloud docker -- push ${GCR_REGION}.gcr.io/${PROJECT_ID}/openarena:0.8.8

<br/>

### Generate an assets disk

In most games binaries are orders of magnitude smaller than assets. Because of this fact, it makes sense to create a container image that only contains binaries; assets can be put on a persistent disk and attached to multiple VM instances that run the DGS container. This architecture saves money and eliminates the need to distribute assets to all VM instances.

Create an OpenArena asset disk by following these steps:

    $ region=us-east1
    $ zone_1=${region}-b
    $ gcloud config set compute/region ${region}

<br/>

    // Create a small Compute Engine VM instance using gcloud.
    $ gcloud compute instances create openarena-asset-builder \
      --machine-type f1-micro \
      --image-family debian-9 \
      --image-project debian-cloud \
      --zone ${zone_1}

Create and attach an appropriately-sized persistent disk. The persistent disk must be separate from the boot disk, and should be configured to remain undeleted when the virtual machine is removed. Kubernetes persistentVolume functionality works best with persistent disks initialized according to the Compute Engine documentation consisting of a single ext4 file system without a partition table.

    $ gcloud compute disks create openarena-assets \
      --size=50GB --type=pd-ssd\
      --description="OpenArena data disk. Mount read-only at /usr/share/games/openarena/baseoa/" \
      --zone ${zone_1}

Wait until the openarena-asset-builder instance has fully started up, then attach the persistent disk.

    $ gcloud compute instances attach-disk openarena-asset-builder \
      --disk openarena-assets --zone ${zone_1}

Once attached, you can SSH into the openarena-asset-builder VM instance and format the new persistent disk.

<br/>

### Connect to the Asset Builder VM Instance using SSH

![Running Dedicated Game Servers in Google Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/running-dedicated-game-servers-in-google-kubernetes-engine/screen-01.png 'Running Dedicated Game Servers in Google Kubernetes Engine'){: .center-image }

Создали новую виртуальную машину, к которой подключаемся по ssh

Compute Engine --> VM Instances --> openarena-asset-builder --> ssh

<br/>

### Format and Configure the Assets Disk

    $ sudo lsblk

    $ sudo mkfs.ext4 -m 0 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb

    $ sudo mkdir -p /usr/share/games/openarena/baseoa/

    $ sudo mount -o discard,defaults /dev/sdb \
        /usr/share/games/openarena/baseoa/

    $ sudo apt-get update
    $ sudo apt-get -y install openarena-server

    $ sudo gsutil cp gs://qwiklabs-assets/single-match.cfg /usr/share/games/openarena/baseoa/single-match.cfg

Once the installation has been completed unmount the persistent volume and shut down the instance:

    $ sudo umount -f -l /usr/share/games/openarena/baseoa/
    $ sudo shutdown -h now

The SSH console will now stop responding, this is expected.

The persistent disk is ready to be used as a persistentVolume in Kubernetes and the instance can be safely deleted.

<br/>

### Main VM

Return to your main lab VM instance SSH console and verify value of zone_1 variable which you set earlier

    $ echo $zone_1

If it is not set, i.e. it returns none, you can set it via following command:

    $ region=us-east1
    $ zone_1=${region}-b

<br/>

Now, delete the openarena-asset-builder VM.

    // Удаляем ненужную более виртуальную машину
    $ gcloud compute instances delete openarena-asset-builder --zone ${zone_1}

Enter Y when prompted to confirm the deletion.

<br/>

## Setting up a Kubernetes cluster

### Creating a Kubernetes cluster on Container Engine

    $ gcloud compute networks create game

<br/>

    $ gcloud compute firewall-rules create openarena-dgs --network game \
      --allow udp:27961-28061

<br/>

    $ gcloud container clusters create openarena-cluster \
      --num-nodes 3 \
      --network game \
      --machine-type=n1-standard-2 \
      --zone=${zone_1}

<br/>

    $ gcloud container clusters get-credentials openarena-cluster --zone ${zone_1}

<br/>

### Configuring the assets disk in Kubernetes

    $ cd ~/gke-dedicated-game-server/openarena/
    $ kubectl apply -f k8s/asset-volume.yaml
    $ kubectl apply -f k8s/asset-volumeclaim.yaml

    $ kubectl get persistentVolume
    $ kubectl get persistentVolumeClaim

<br/>

### Setting up the scaling manager

    $ cd ../scaling-manager
    $ ./build-and-push.sh

<br/>

### Configure the Openarena Scaling Manager Deployment File

    $ gcloud compute instance-groups managed list

Copy the base instance name. (gke-openarena-cluster-default-pool-8eb88aad)

<!--

    $ export PROJECT_ID=$(gcloud config get-value project)
    $ export GCR_REGION=us

-->

    // Нужно оставить только последний ID.
    $ export GKE_BASE_INSTANCE_NAME=8eb88aad
    $ export GCP_ZONE=us-east1-b

    $ printf "$GCR_REGION \n$PROJECT_ID \n$GKE_BASE_INSTANCE_NAME \n$GCP_ZONE \n"

<br/>

    $ sed -i "s/\[GCR_REGION\]/$GCR_REGION/g" k8s/openarena-scaling-manager-deployment.yaml

    $ sed -i "s/\[PROJECT_ID\]/$PROJECT_ID/g" k8s/openarena-scaling-manager-deployment.yaml

    $ sed -i "s/\[ZONE\]/$GCP_ZONE/g" k8s/openarena-scaling-manager-deployment.yaml

    $ sed -i "s/\gke-openarena-cluster-default-pool-\[REPLACE_ME\]/$GKE_BASE_INSTANCE_NAME/g" k8s/openarena-scaling-manager-deployment.yaml

<br/>

    $ kubectl apply -f k8s/openarena-scaling-manager-deployment.yaml
    $ kubectl get pods

## Testing the setup

### Requesting a new DGS instance

    $ cd ~/gke-dedicated-game-server

    $ sed -i "s/\[GCR_REGION\]/$GCR_REGION/g" openarena/k8s/openarena-pod.yaml
    $ sed -i "s/\[PROJECT_ID\]/$PROJECT_ID/g" openarena/k8s/openarena-pod.yaml

<br/>

<!--

    // Try this only if the pod errors out at launch.
    $ sed -i "s/\/usr\/share\/games\/openarena\/baseoa/\/usr\/lib\/openarena-server\/baseoa/g"  openarena/k8s/openarena-pod.yaml

-->

    // Apply the new pod configuration by running:
    $ kubectl apply -f openarena/k8s/openarena-pod.yaml

<br/>

    $ kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    openarena.dgs                      1/1     Running   0          33s
    scaling-manager-798947bd4c-gfm8r   3/3     Running   1          2m4s

<br/>

## Connecting to the DGS

    $ export NODE_NAME=$(kubectl get pod openarena.dgs \
        -o jsonpath="{.spec.nodeName}")

    $ export DGS_IP=$(gcloud compute instances list \
        --filter="name=( ${NODE_NAME} )" \
        --format='table[no-heading](EXTERNAL_IP)')

<br/>

    $ printf "Node Name: $NODE_NAME \nNode IP  : $DGS_IP \nPort         : 27961\n"

<br/>

    $ printf " launch client with: \nopenarena +connect $DGS_IP +set net_port 27961\n"

<br/>

### На клиетне Ubuntu 18

// Есть и другие операционки
https://openarena.fandom.com/wiki/Manual/Install

    $ sudo apt-get install -y openarena
    $ openarena

Multiplayer

![Running Dedicated Game Servers in Google Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/running-dedicated-game-servers-in-google-kubernetes-engine/screen-02.png 'Running Dedicated Game Servers in Google Kubernetes Engine'){: .center-image }

Connect.

К серверу подключился, но меня из игры выбрасывало. Наверное нужны особые настройки под железо.

Потом под перешел в состояние completed. Я его пересоздал. После возникли проблемы с scaling-manager.

    $ kubectl log scaling-manager-798947bd4c-gfm8r
    log is DEPRECATED and will be removed in a future version. Use logs instead.
    Error from server (BadRequest): a container name must be specified for pod scaling-manager-798947bd4c-gfm8r, choose one of: [scaling-manager node-stopper kubectl-proxy]

Играть не собираюсь, разбираться не стал.

<br/>

## Testing the scaling manager

Since the scaling manager scales the number of VM instances in the Kubernetes cluster based on the number of DGS pods, testing it requires requesting a number of pods over a period of time and checking that the number of nodes scale appropriately.

For testing purposes, a script is provided in the solutions repository which adds four DGS pods per minute for 5 minutes:

    $ ./scaling-manager/tests/test-loader.sh

In your own environment, make sure to set the match length in the server configuration file to an appropriate limit so that the pods eventually exit and you can see the nodes scale back down! For this lab, the server configuration file places a five minute time limit on the match, and can be used as an example. It's located in the repo at openarena/single-match.cfg, and is used by default.

Google Cloud Platform --> Kubernetes Engine --> Workloads.

![Running Dedicated Game Servers in Google Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/running-dedicated-game-servers-in-google-kubernetes-engine/screen-03.png 'Running Dedicated Game Servers in Google Kubernetes Engine'){: .center-image }

You will see a sequence of workloads startup called openarena-dgs.s, openarena-dgs.2 up to openarena-dgs.15. Because there are a limited the number of vCPUs in this cluster, many of the test containers will initially show an error state with a status of "Unschedulable". As the startup load in each container reduces, some of the later game server containers will successfully start up. All containers will start after about 10 minutes.

If you explore any of the failed instances you will see that the reason for the failure is a lack of CPU resources. If you explore any of the running instances you will see that the game server has initialized on those instances.
