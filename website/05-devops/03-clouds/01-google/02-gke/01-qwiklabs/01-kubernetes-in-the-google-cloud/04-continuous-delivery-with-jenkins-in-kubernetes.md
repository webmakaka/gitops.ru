---
layout: page
title: Continuous Delivery with Jenkins in Kubernetes Engine
description: Continuous Delivery with Jenkins in Kubernetes Engine
keywords: Continuous Delivery with Jenkins in Kubernetes Engine
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/
---

# [GSP051] Continuous Delivery with Jenkins in Kubernetes Engine

<br/>

https://www.qwiklabs.com/quests/29

<br/>

Делаю!  
23.05.2019

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-cloud.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

<br/>

### Clone Repository

    $ gcloud config set compute/zone us-central1-f

    $ git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git

    $ cd continuous-deployment-on-kubernetes

<br/>

### Provisioning Jenkins

    $ gcloud container clusters create jenkins-cd \
    --num-nodes 2 \
    --machine-type n1-standard-2 \
    --scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"

<br/>

    $ gcloud container clusters list

    $ gcloud container clusters get-credentials jenkins-cd

    $ kubectl cluster-info

<br/>

### Install Helm

    $ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz

    $ tar zxfv helm-v2.9.1-linux-amd64.tar.gz

    $ cp linux-amd64/helm .

    $ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)

    $ kubectl create serviceaccount tiller --namespace kube-system

    $ kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

    $ ./helm init --service-account=tiller
    $ ./helm update


    // Подзависало на этом шагу
    $ ./helm version

<br/>

### Configure and Install Jenkins

    $ ./helm install -n cd stable/jenkins -f jenkins/values.yaml --version 0.16.6 --wait

    // минуты 2 поднимается
    $ kubectl get pods

    $ export POD_NAME=$(kubectl get pods -l "component=cd-jenkins-master" -o jsonpath="{.items[0].metadata.name}")

    $ kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &

    $ kubectl get svc
    NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
    cd-jenkins         ClusterIP   10.11.245.33   <none>        8080/TCP    6m40s
    cd-jenkins-agent   ClusterIP   10.11.240.32   <none>        50000/TCP   6m40s
    kubernetes         ClusterIP   10.11.240.1    <none>        443/TCP     12m

<br/>

### Connect to Jenkins

    // Получить пароль для логина
    $ printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

<br/>

web preview --> preview on port 8080

<br/>

Открылось окно. Залогинился.

admin/пароль

<br/>

Нужно зайти в настройки и обновить плагины. Иначе ничего не заработает!

<br/>

### Understanding the Application

The application mimics a microservice by supporting two operation modes.

-   In backend mode: gceme listens on port 8080 and returns Compute Engine instance metadata in JSON format.
-   In frontend mode: gceme queries the backend gceme service and renders the resulting JSON in the user interface.

<br/>

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-cloud-app.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

<br/>

### Deploying the Application

You will deploy the application into two different environments:

-   Production: The live site that your users access.
-   Canary: A smaller-capacity site that receives only a percentage of your user traffic. Use this environment to validate your software with live traffic before it's released to all of your users.

    \$ cd sample-app

    $ kubectl create ns production
    $ kubectl apply -f k8s/production -n production
    $ kubectl apply -f k8s/canary -n production
    $ kubectl apply -f k8s/services -n production

<br/>

    $ kubectl scale deployment gceme-frontend-production -n production --replicas 4

<br/>

    $ kubectl get pods -n production -l app=gceme -l role=frontend
    NAME                                         READY   STATUS    RESTARTS   AGE
    gceme-frontend-canary-84cc88cccf-7fqg4       1/1     Running   0          58s
    gceme-frontend-production-5df96c664d-7x82l   1/1     Running   0          16s
    gceme-frontend-production-5df96c664d-flm5q   1/1     Running   0          16s
    gceme-frontend-production-5df96c664d-jcw4v   1/1     Running   0          16s
    gceme-frontend-production-5df96c664d-x9lbb   1/1     Running   0          72s

<br/>

    $ kubectl get pods -n production -l app=gceme -l role=backend
    NAME                                       READY   STATUS    RESTARTS   AGE
    gceme-backend-canary-688b9c69d9-h2vjr      1/1     Running   0          88s
    gceme-backend-production-d6559978d-5pcx5   1/1     Running   0          102s

<br/>

    $ kubectl get service gceme-frontend -n production
    NAME             TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
    gceme-frontend   LoadBalancer   10.11.246.46   35.225.165.249   80:32240/TCP   99s

<br/>

    $ export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)

<br/>

    $ curl http://$FRONTEND_SERVICE_IP/version
    1.0.0

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/app1.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

<br/>

<br/>

### Creating the Jenkins Pipeline

Let's create a copy of the gceme sample app and push it to a Cloud Source Repository:

    $ gcloud alpha source repos create default

<br/>

    $ git init
    $ git config credential.helper gcloud.sh
    $ git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default


    // да любые
    $ git config --global user.email "[EMAIL_ADDRESS]"
    $ git config --global user.name "[USERNAME]"

    $ git add .
    $ git commit -m "Initial commit"
    $ git push origin master

<br/>

### Adding your service account credentials

Step 1: In the Jenkins user interface, click Credentials in the left navigation.

Step 2: Click Jenkins

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-pic-1.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

Step 3: Click Global credentials (unrestricted).

Step 4: Click Add Credentials in the left navigation.

Step 5: Select Google Service Account from metadata from the Kind drop-down and click OK.

The global credentials has been added. The name of the credential is the GCP Project ID found in the CONNECTION DETAILS section of the lab.

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-pic-2.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

<br/>

### Creating the Jenkins job

Step 1: Click Jenkins > New Item in the left navigation:

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-pic-3.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

Step 2: Name the project sample-app, then choose the Multibranch Pipeline option and click OK.

Step 3: On the next page, in the Branch Sources section, click Add Source and select git.

Step 4: Paste the HTTPS clone URL of your sample-app repo in Cloud Source Repositories into the Project Repository field. Replace [PROJECT_ID] with your GCP Project ID:

https://source.developers.google.com/p/qwiklabs-gcp-08db4b7004575b72/r/default

Step 5: From the Credentials drop-down, select the name of the credentials you created when adding your service account in the previous steps.

Step 6: Under Scan Multibranch Pipeline Triggers section, check the Periodically if not otherwise run box and set the Interval value to 1 minute.

Step 7: Your job configuration should look like this:

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-pic-4.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-pic-5.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-pic-6.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

Step 8: Click Save leaving all other options with their defaults

Job завершается ошибкой. Вроде как это нормально.

<br/>

### Creating the Development Environment

Creating a development branch

    $ git checkout -b new-feature

    $ gcloud config get-value project

    $ vi Jenkinsfile

Вместо REPLACE_WITH_YOUR_PROJECT_ID вставить реальный PROJECT_ID.

<br/>

    $ vi html.go

<br/>

    <div class="card blue">

Меняем на

    <div class="card orange">

<br/>

    $ vi main.go

<br/>

    const version string = "1.0.0"

меняем на

    const version string = "2.0.0"

<br/>

### Kick off Deployment

    $ git add Jenkinsfile html.go main.go
    $ git commit -m "Version 2.0.0"
    $ git push origin new-feature

<br/>

    $ kubectl proxy &
    $ curl http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version
    2.0.0

<br/>

### Deploying a Canary Release

    $ git checkout -b canary
    $ git push origin canary

    $ export FRONTEND_SERVICE_IP=$(kubectl get -o \
    jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)

    $ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
    1.0.0

### Deploying to production

    $ git checkout master
    $ git merge canary
    $ git push origin master

Ждем пока jenkins сделает свое дело.

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/jenkins-final.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }

    $ export FRONTEND_SERVICE_IP=$(kubectl get -o \
    jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)

    $ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
    2.0.0

    $ kubectl get service gceme-frontend -n production
    NAME             TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
    gceme-frontend   LoadBalancer   10.11.240.107   35.232.28.72   80:32013/TCP   35m

![Continuous Delivery with Jenkins in Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/continuous-delivery-with-jenkins-in-kubernetes/app2.png 'Continuous Delivery with Jenkins in Kubernetes Engine'){: .center-image }
