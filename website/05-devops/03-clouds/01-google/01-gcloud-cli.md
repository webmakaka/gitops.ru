---
layout: page
title: Инсталляция GCloud CLI
description: Инсталляция GCloud CLI
keywords: Инсталляция GCloud CLI
permalink: /devops/clouds/google/gcloud-cli/
---

# Инсталляция GCloud CLI

Делаю!  
21.05.2019

http://cloud.google.com/sdk/

<br/>

### Ubuntu 18.04

// UPD (2023)  
// https://cloud.google.com/storage/docs/gsutil_install#deb

<br/>

    $ export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"

    $ echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

    $ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

    $ sudo apt-get update && sudo apt-get install google-cloud-sdk

<br/>

### Подключение

    Google Cloud Platform --> IAM & admin --> Service Account --> Create service account

    Service account name: skd-service-account
    Service account description: Account for GCP SDK

    Create

<br/>

    Role: Owner

<br/>

Create Key --> JSON

Ключ сохранился на локальный компьютер

<br/>

    $ gcloud auth activate-service-account --key-file=/mnt/dsk1/qwiklabs-gcp-d8a61bdf964e52e3-9775b0a34e00.json

<br/>

    $ gcloud init

<br/>

    Pick configuration to use:
    [1] Re-initialize this configuration [default] with new settings
    [2] Create a new configuration
    Please enter your numeric choice:  1

<br/>

    Choose the account you would like to use to perform operations for
    this configuration:
    [1] skd-service-account@qwiklabs-gcp-d8a61bdf964e52e3.iam.gserviceaccount.com
    [2] Log in with a new account
    Please enter your numeric choice:  1

<br/>

    Pick cloud project to use:
    [1] qwiklabs-gcp-d8a61bdf964e52e3
    [2] Create a new project
    Please enter numeric choice or text value (must exactly match list
    item):  1

<br/>

### Проверка

    $ gcloud auth list
    $ gcloud config list
    $ gcloud info

<br/>

### Некоторые команды

    $ gcloud compute regions list
    $ gcloud compute zones list
    $ gcloud config set compute/zone europe-west1-c

<br/>

    $ gcloud components list

    // Если kubectl not installed
    $ gcloud components install kubectl

<br/>

    $ gcloud compute instances list
    $ gcloud container clusters list

<br/>

    // Очистить список аккаунтов из auth list
    $ gcloud auth revoke --all

    // Подключиться к cloud-shell (Чтобы не рабать в браузерной командной строке)oud-shell ssh

<!-- gcloud config configurations delete



https://cloud.google.com/blog/products/gcp/introducing-the-ability-to-connect-to-cloud-shell-from-any-terminal

-->
