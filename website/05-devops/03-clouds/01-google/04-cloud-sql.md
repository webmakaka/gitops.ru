---
layout: page
title: Cloud SQL
description: Cloud SQL
keywords: Cloud SQL
permalink: /devops/clouds/google/cloud-sql/
---

# Cloud SQL

    // create a Cloud SQL instance
    $ gcloud sql instances create flights \
    --tier=db-n1-standard-1 --activation-policy=ALWAYS

    $ gcloud sql users set-password root --host % --instance flights \
    --password Passw0rd

    // Now create an environment variable with the IP address of the Cloud Shell:
    $ export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32


    // Whitelist the Cloud Shell instance for management access to your SQL instance.
    $ gcloud sql instances patch flights --authorized-networks $ADDRESS


    // Get the IP address of your Cloud SQL instance
    $ MYSQLIP=$(gcloud sql instances describe \
    flights --format="value(ipAddresses.ipAddress)")

    $ echo $MYSQLIP
    35.232.157.0

    // Create the flights table using the create_table.sql file.
    $ mysql --host=$MYSQLIP --user=root \
          --password --verbose < create_table.sql

<br/>

    $ git clone \
      https://github.com/GoogleCloudPlatform/data-science-on-gcp/

    $ cd data-science-on-gcp/03_sqlstudio


    // Connect to the mysql command line interface:
    $ mysql --host=$MYSQLIP --user=root  --password

    use bts;
    describe flights;

    select DISTINCT(FL_DATE) from flights;

    exit

<br/>

### Add data to Cloud SQL instance

    $ counter=0
    $ for FILE in 201501.csv 201502.csv; do
      gsutil cp gs://$BUCKET/flights/raw/$FILE \
                flights.csv-${counter}
      counter=$((counter+1))
    done

<br/>

    $ mysqlimport --local --host=$MYSQLIP --user=root --password \
    --ignore-lines=1 --fields-terminated-by=',' bts flights.csv-*

    $ mysql --host=$MYSQLIP --user=root  --password

<br/>

### Build the initial data model

    use bts;
    select DISTINCT(FL_DATE) from flights;
