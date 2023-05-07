---
layout: page
title: Cloud Storage bucket
description: Cloud Storage bucket
keywords: Cloud Storage bucket
permalink: /devops/clouds/google/cloud-storage-bucket/
---

# Cloud Storage bucket

    $ export PROJECT_ID=$(gcloud info --format='value(config.project)')
    $ export BUCKET_NAME=${PROJECT_ID}-ml

    // Create the storage bucket
    $ gsutil mb gs://${BUCKET_NAME}


    // Скопировать все файлы в Cloud Storage bucket
    $ gsutil -m cp *.csv gs://${BUCKET_NAME}/flights/raw/
