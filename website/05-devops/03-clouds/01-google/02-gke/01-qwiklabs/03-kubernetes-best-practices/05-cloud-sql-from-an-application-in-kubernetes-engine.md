---
layout: page
title: Connect to Cloud SQL from an Application in Kubernetes Engine
description: Connect to Cloud SQL from an Application in Kubernetes Engine
keywords: Connect to Cloud SQL from an Application in Kubernetes Engine
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-best-practices/connect-to-cloud-sql-from-an-application-in-kubernetes-engine/
---

# [GSP449] Connect to Cloud SQL from an Application in Kubernetes Engine

<br/>

Делаю:  
04.05.2019

https://www.qwiklabs.com/focuses/5625?parent=catalog

<br/>

### Overview

This lab shows how easy it is to connect an application in Kubernetes Engine to a Cloud SQL instance using the Cloud SQL Proxy container as a sidecar container. You will deploy a Kubernetes Engine cluster and a Cloud SQL Postgres instance and use the Cloud SQL Proxy container to allow communication between them.

<br/>

    $ gcloud config set compute/region us-central1
    $ gcloud config set compute/zone us-central1-a

<br/>

    $ git clone https://github.com/GoogleCloudPlatform/gke-cloud-sql-postgres-demo.git


    $ cd gke-cloud-sql-postgres-demo

<br/>

### Deployment

    // USER_PASSWORD=<password> PG_ADMIN_CONSOLE_PASSWORD=<password> ./create.sh <DATABASE_USER_NAME> <PGADMIN_USERNAME>

    $ USER_PASSWORD=mypassword PG_ADMIN_CONSOLE_PASSWORD=mypassword ./create.sh dbadmin pgadmin

During the deployment, create.sh will run the following scripts:

1. enable_apis.sh - enables the Kubernetes Engine API and Cloud SQL Admin API.
2. postgres_instance.sh - creates the Cloud SQL instance and additional Postgres user. Note that gcloud will timeout when waiting for the creation of a Cloud SQL instance so the script manually polls for its completion instead.
3. service_account.sh - creates the service account for the Cloud SQL Proxy container and creates the credentials file.
4. cluster.sh - Creates the Kubernetes Engine cluster.
5. configs_and_secrets.sh - creates the Kubernetes Engine secrets and configMap containing credentials and connection string for the Cloud SQL instance.
6. pgadmin_deployment.sh - creates the pgAdmin4 pod.

Deployment of the Cloud SQL instnace can take up to 10 minutes.

<br/>

    $ make validate


    // port-forward to the running pod
    $ make expose

Cloud Shell --> Preview on Port 8080

<br/>

    login: pgadmin
    password: mypassword

<br/>

Админка как-то криво установилась. Никаких работающих кнопок нет.
Пробовал 2 раза. Не заработало.

<br/>

    From there, click Add New Server.

    On the General tab give your server a name.

    On the Connection tab set the following:

    Host name/address: 127.0.0.1
    Username: <DATABASE_USER_NAME>(dbadmin)
    Password: <USER_PASSWORD> you created
