---
layout: page
title: Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine
description: Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine
keywords: Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/
---

# [GSP114] Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine

<br/>

https://www.qwiklabs.com/focuses/552?parent=catalog

<br/>

Делаю!  
03.10.2019

<br/>

### Introduction

This hands-on lab shows you how to create a continuous delivery pipeline using Google Kubernetes Engine, Google Cloud Source Repositories, Google Cloud Container Builder, and Spinnaker. After you create a sample application, you configure these services to automatically build, test, and deploy it. When you modify the application code, the changes trigger the continuous delivery pipeline to automatically rebuild, retest, and redeploy the new version.

<br/>

### Objectives

-   Set up your environment by launching Google Cloud Shell, creating a Kubernetes Engine cluster, and configuring your identity and user management scheme.
-   Download a sample application, create a Git repository then upload it to a Google Cloud Source Repository.
-   Deploy Spinnaker to Kubernetes Engine using Helm.
-   Build your Docker image.
-   Create triggers to create Docker images when your application changes.
-   Configure a Spinnaker pipeline to reliably and continuously deploy your application to Kubernetes Engine.
-   Deploy a code change, triggering the pipeline, and watch it roll out to production.

<br/>

### Pipeline architecture

To continuously deliver application updates to your users, you need an automated process that reliably builds, tests, and updates your software. Code changes should automatically flow through a pipeline that includes artifact creation, unit testing, functional testing, and production rollout. In some cases, you want a code update to apply to only a subset of your users, so that it is exercised realistically before you push it to your entire user base. If one of these canary releases proves unsatisfactory, your automated procedure must be able to quickly roll back the software changes.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic1.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

With Kubernetes Engine and Spinnaker you can create a robust continuous delivery flow that helps to ensure your software is shipped as quickly as it is developed and validated. Although rapid iteration is your end goal, you must first ensure that each application revision passes through a gamut of automated validations before becoming a candidate for production rollout. When a given change has been vetted through automation, you can also validate the application manually and conduct further pre-release testing.

After your team decides the application is ready for production, one of your team members can approve it for production deployment.

<br/>

### Application delivery pipeline

In this lab you build the continuous delivery pipeline shown in the following diagram.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic2.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

<br/>

## Set up your environment

    $ gcloud config set compute/zone us-central1-f

    $ gcloud container clusters create spinnaker-tutorial \
    --machine-type=n1-standard-2

This deployment takes between five and ten minutes to complete. You may see a warning about default scopes that you can safely ignore as it has no impact on this lab. Wait for the deployment to complete before proceeding.

When completed you see a report detailing the name, location, version, ip-address, machine-type, node version, number of nodes and status of the cluster that indicates the cluster is running.

<br/>

### Configure identity and access management

Create a Cloud Identity Access Management (Cloud IAM) service account to delegate permissions to Spinnaker, allowing it to store data in Cloud Storage. Spinnaker stores its pipeline data in Cloud Storage to ensure reliability and resiliency. If your Spinnaker deployment unexpectedly fails, you can create an identical deployment in minutes with access to the same pipeline data as the original.

Upload your startup script to a Cloud Storage bucket by following these steps:

    // Create the service account:
    $ gcloud iam service-accounts create spinnaker-account \
      --display-name spinnaker-account

<br/>

    // Store the service account email address and your current project ID in environment variables for use in later commands:
    $ export SA_EMAIL=$(gcloud iam service-accounts list \
        --filter="displayName:spinnaker-account" \
        --format='value(email)')

    $ export PROJECT=$(gcloud info --format='value(config.project)')

<br/>

    // Bind the storage.admin role to your service account:
    $ gcloud projects add-iam-policy-binding $PROJECT \
        --role roles/storage.admin \
        --member serviceAccount:$SA_EMAIL

<br/>

    // Download the service account key. In a later step, you will install Spinnaker and upload this key to Kubernetes Engine:
    $ gcloud iam service-accounts keys create spinnaker-sa.json \
     --iam-account $SA_EMAIL

<br/>

## Set up Cloud Pub/Sub to trigger Spinnaker pipelines

    // Create the Cloud Pub/Sub topic for notifications from Container Registry.
    $ gcloud pubsub topics create projects/$PROJECT/topics/gcr

    // Create a subscription that Spinnaker can read from to receive notifications of images being pushed.
    $ gcloud pubsub subscriptions create gcr-triggers \
    --topic projects/${PROJECT}/topics/gcr

    // Give Spinnaker's service account permissions to read from the gcr-triggers subscription.

    $ export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')

    $ gcloud beta pubsub subscriptions add-iam-policy-binding gcr-triggers \
    --role roles/pubsub.subscriber --member serviceAccount:$SA_EMAIL

<br/>

## Deploying Spinnaker using Helm

In this section you use Helm to deploy Spinnaker from the Charts repository. Helm is a package manager you can use to configure and deploy Kubernetes applications.

<br/>

### Install Helm

    $ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.10.0-linux-amd64.tar.gz

    $ tar zxfv helm-v2.10.0-linux-amd64.tar.gz
    $ cp linux-amd64/helm .

    // Grant Tiller, the server side of Helm, the cluster-admin role in your cluster:
    $ kubectl create clusterrolebinding user-admin-binding \
    --clusterrole=cluster-admin --user=$(gcloud config get-value account)

    $ kubectl create serviceaccount tiller \
    --namespace kube-system

    $ kubectl create clusterrolebinding tiller-admin-binding \
    --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

    // Grant Spinnaker the cluster-admin role so it can deploy resources across all namespaces:

    $ kubectl create clusterrolebinding --clusterrole=cluster-admin \
    --serviceaccount=default:default spinnaker-admin

    // Initialize Helm to install Tiller, the server side of Helm, in your cluster:
    $ ./helm init --service-account=tiller

    $ ./helm repo update

    // sure that Helm is properly installed by running the following command. If Helm is correctly installed, v2.10.0 outputs for both client and server.

    $ ./helm version

<br/>

### Configure Spinnaker

    // Still in Cloud Shell, create a bucket for Spinnaker to store its pipeline configuration:
    $ export PROJECT=$(gcloud info \
    --format='value(config.project)')

    $ export BUCKET=$PROJECT-spinnaker-config

    $ gsutil mb -c regional -l us-central1 gs://$BUCKET


    // Run the following command to create a spinnaker-config.yaml file, which describes how Helm should install Spinnaker:

    $ export SA_JSON=$(cat spinnaker-sa.json)
    $ export PROJECT=$(gcloud info --format='value(config.project)')
    $ export BUCKET=$PROJECT-spinnaker-config

```
$ cat > spinnaker-config.yaml <<EOF
gcs:
  enabled: true
  bucket: $BUCKET
  project: $PROJECT
  jsonKey: '$SA_JSON'

dockerRegistries:
- name: gcr
  address: https://gcr.io
  username: _json_key
  password: '$SA_JSON'
  email: 1234@5678.com

# Disable minio as the default storage backend
minio:
  enabled: false

# Configure Spinnaker to enable GCP services
halyard:
  spinnakerVersion: 1.10.2
  image:
    tag: 1.12.0
  additionalScripts:
    create: true
    data:
      enable_gcs_artifacts.sh: |-
        \$HAL_COMMAND config artifact gcs account add gcs-$PROJECT --json-path /opt/gcs/key.json
        \$HAL_COMMAND config artifact gcs enable
      enable_pubsub_triggers.sh: |-
        \$HAL_COMMAND config pubsub google enable
        \$HAL_COMMAND config pubsub google subscription add gcr-triggers \
          --subscription-name gcr-triggers \
          --json-path /opt/gcs/key.json \
          --project $PROJECT \
          --message-format GCR
EOF
```

<br/>

### Deploy the Spinnaker chart

    // Use the Helm command-line interface to deploy the chart with your configuration set:
    $ ./helm install -n cd stable/spinnaker -f spinnaker-config.yaml \
    --timeout 600 --version 1.1.6 --wait

This command typically takes five to ten minutes to complete.

<br/>

After the command completes, run the following command to set up port forwarding to Spinnaker from Cloud Shell:

    $ export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
        -o jsonpath="{.items[0].metadata.name}")

    $ kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &

Note: This command can take several minutes to complete. Be sure to wait until you see that it has succeeded before proceeding.

To open the Spinnaker user interface, click the Web Preview icon at the top of the Cloud Shell window and select Preview on port 8080.

The welcome screen opens, followed by the Spinnaker user interface:

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic3.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

Leave this tab open, this is where you'll access the Spinnaker UI.

<br/>

## Building the Docker image

    $ wget https://gke-spinnaker.storage.googleapis.com/sample-app-v2.tgz
    $ tar xzfv sample-app-v2.tgz
    $ cd sample-app

Set the username and email address for your Git commits in this repository. Replace [USERNAME] with a username you create:

    $ git config --global user.email "$(gcloud config get-value core/account)"
    $ git config --global user.name "[USERNAME]"

    $ git init
    $ git add .
    $ git commit -m "Initial commit"

    // Create a repository to host your code:
    $ gcloud source repos create sample-app

    $ git config credential.helper gcloud.sh

    $ export PROJECT=$(gcloud info --format='value(config.project)')
    $ git remote add origin https://source.developers.google.com/p/$PROJECT/r/sample-app

    $ git push origin master

Check that you can see your source code in the Console by clicking Navigation Menu > Source Repositories.

Click sample-app.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic4.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

<br/>

### Configure your build triggers

Configure Container Builder to build and push your Docker images every time you push Git tags to your source repository. Container Builder automatically checks out your source code, builds the Docker image from the Dockerfile in your repository, and pushes that image to Google Cloud Container Registry.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic5.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

In the Cloud Platform Console, click Navigation menu > Cloud Build > Triggers.

Click Create trigger.

Select your newly created sample-app repository then click Continue.

Set the following trigger settings:

Name:sample-app-tags

Trigger type: Tag

Tag (regex): v.\*

Build configuration: Cloud Build configuration file (yaml or json)

cloudbuild.yaml location: /cloudbuild.yaml

Click Create trigger.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic6.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

From now on, whenever you push a Git tag prefixed with the letter "v" to your source code repository, Container Builder automatically builds and pushes your application as a Docker image to Container Registry.

<br/>

### Prepare your Kubernetes Manifests for use in Spinnaker

Spinnaker needs access to your Kubernetes manifests in order to deploy them to your clusters. This section creates a Cloud Storage bucket that will be populated with your manifests during the CI process in Cloud Build. After your manifests are in Cloud Storage, Spinnaker can download and apply them during your pipeline's execution.

    // Create the bucket
    $ export PROJECT=$(gcloud info --format='value(config.project)')
    $ gsutil mb -l us-central1 gs://$PROJECT-kubernetes-manifests

    // Enable versioning on the bucket so that you have a history of your manifests:
    $ gsutil versioning set on gs://$PROJECT-kubernetes-manifests

    // Set the correct project ID in your kubernetes deployment manifests:
    $ sed -i s/PROJECT/$PROJECT/g k8s/deployments/*

    // Commit the changes to the repository:
    $ git commit -a -m "Set project ID"

<br/>

### Build your image

Push your first image using the following steps:

    // In Cloud Shell, still in the sample-app directory, create a Git tag:
    $ git tag v1.0.0

    // Push the tag:
    $ git push --tags

Go to the GCP Console. Still in Cloud Build, click History in the left pane to check that the build has been triggered. If not, verify that the trigger was configured properly in the previous section.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic7.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

Stay on this page and wait for the build to complete before going on to the next section.

<br/>

## Configuring your deployment pipelines

Now that your images are building automatically, you need to deploy them to the Kubernetes cluster.

You deploy to a scaled-down environment for integration testing. After the integration tests pass, you must manually approve the changes to deploy the code to production services.

<br/>

### Install the spin CLI for managing Spinnaker

spin is a command-line utility for managing Spinnaker's applications and pipelines.

    $ curl -LO https://storage.googleapis.com/spinnaker-artifacts/spin/1.6.0/linux/amd64/spin

    $ chmod +x spin

<br/>

### Create the deployment pipeline

Use spin to create an app called sample in Spinnaker. Set the owner email address for the app in Spinnaker:

    $ ./spin application save --application-name sample \
      --owner-email "$(gcloud config get-value core/account)" \
      --cloud-providers kubernetes \
      --gate-endpoint http://localhost:8080/gate

Ignore the Could not read configuration file... output message.

Next, you create the continuous delivery pipeline. In this tutorial, the pipeline is configured to detect when a Docker image with a tag prefixed with "v" has arrived in your Container Registry.

From your sample-app source code directory, run the following command to upload an example pipeline to your Spinnaker instance:

    $ export PROJECT=$(gcloud info --format='value(config.project)')

    $ sed s/PROJECT/$PROJECT/g spinnaker/pipeline-deploy.json > pipeline.json

    $ ./spin pipeline save --gate-endpoint http://localhost:8080/gate -f pipeline.json

Ignore the Could not read configuration file... output message.

<br/>

### Manually Trigger and View your pipeline execution

The configuration you just created uses notifications of newly tagged images being pushed to trigger a Spinnaker pipeline. In a previous step, you pushed a tag to the Cloud Source Repositories which triggered Cloud Build to build and push your image to Container Registry. To verify the pipeline, manually trigger it.

In the Spinnaker UI and click Applications at the top of the screen to see your list of managed applications. sample is your application. If you don't see sample, try refreshing the Spinnaker Applications tab.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic8.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

Click sample to view your application deployment.
Click Pipelines at the top to view your applications pipeline status.
Click Start Manual Execution to trigger the pipeline this first time.

Click Run.

Click Details to see more information about the pipeline's progress.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic9.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

The progress bar shows the status of the deployment pipeline and its steps.

Steps in blue are currently running, green ones have completed successfully, and red ones have failed.

Click a stage to see details about it.
After 3 to 5 minutes the integration test phase completes and the pipeline requires manual approval to continue the deployment.

Hover over the yellow "person" icon and click Continue.

Your rollout continues to the production frontend and backend deployments. It completes after a few minutes.

To view the app, select Infrastructure > Load Balancers in the top of the Spinnaker UI.

Scroll down the list of load balancers and click Default, under service sample-frontend-production.

Scroll down the details pane on the right and copy your app's IP address by clicking the clipboard button on the Ingress IP. The ingress IP link from the Spinnaker UI may use HTTPS by default, while the application is configured to use HTTP.

Paste the address into a new browser tab to view the production version of the application.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic10.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

You have now manually triggered the pipeline to build, test, and deploy your application.

<br/>

## Triggering your pipeline from code changes

Now test the pipeline end to end by making a code change, pushing a Git tag, and watching the pipeline run in response. By pushing a Git tag that starts with "v", you trigger Container Builder to build a new Docker image and push it to Container Registry. Spinnaker detects that the new image tag begins with "v" and triggers a pipeline to deploy the image to canaries, run tests, and roll out the same image to all pods in the deployment.

From your sample-app directory, change the color of the app from orange to blue:

    $ sed -i 's/orange/blue/g' cmd/gke-info/common-service.go

Tag your change and push it to the source code repository:

    $ git commit -a -m "Change color to blue"
    $ git tag v1.0.1
    $ git push --tags

In the Console, in Cloud Build > History, wait a couple of minutes for the new build to appear. You may need to refresh your page. Wait for the new build to complete, before going to the next step.

Return to the Spinnaker UI and click Pipelines to watch the pipeline start to deploy the image. The automatically triggered pipeline will take a few minutes to appear. You may need to refresh your page.

![Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-pipelines-with-spinnaker-and-kubernetes-engine/pic11.png 'Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine'){: .center-image }

<br/>

## Observe the canary deployments

When the deployment is paused, waiting to roll out to production, return to the web page displaying your running application and start refreshing the tab that contains your app. Four of your backends are running the previous version of your app, while only one backend is running the canary. You should see the new, blue version of your app appear about every fifth time you refresh.

After testing completes, return to the Spinnaker tab and approve the deployment by clicking Continue.

When the pipeline completes, your app looks like the following screenshot. Note that the color has changed to blue because of your code change, and that the Version field now reads production.

You have now successfully rolled out your app to your entire production environment!

Optionally, you can roll back this change by reverting your previous commit. Rolling back adds a new tag (v1.0.2), and pushes the tag back through the same pipeline you used to deploy v1.0.1:

    $ git revert v1.0.1
    $ git tag v1.0.2
    $ git push --tags

When the build and then the pipeline completes, verify the roll back by clicking Infrastructure > Load Balancers, then click the service sample-frontend-canary Default and copy the Ingress IP address into a new tab.

Now your app is back to orange and you can see the canary version number.

pic
