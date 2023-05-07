---
layout: page
title: Set Up Network and HTTP Load Balancers
description: Set Up Network and HTTP Load Balancers
keywords: Set Up Network and HTTP Load Balancers
permalink: /devops/clouds/google/gce/qwiklabs/set-up-network-and-http-load-balancers/
---

# [GSP007] Set Up Network and HTTP Load Balancers

<br/>

Делаю!  
23.05.2019

<br/>

### Set the default region and zone for all resources

    $ gcloud config set compute/zone us-central1-a
    $ gcloud config set compute/region us-central1

### Create multiple web server instances

```
$ cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

<br/>

    // Create an instance template, which uses the startup script:
    $ gcloud compute instance-templates create nginx-template \
         --metadata-from-file startup-script=startup.sh

<br/>

    // Create a target pool. A target pool allows a single access point to all the instances in a group and is necessary for load balancing in the future steps.
    $ gcloud compute target-pools create nginx-pool


    // Create a managed instance group using the instance template:
    $ gcloud compute instance-groups managed create nginx-group \
         --base-instance-name nginx \
         --size 2 \
         --template nginx-template \
         --target-pool nginx-pool

<br/>

    $ gcloud compute instances list
    NAME        ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
    nginx-fz6j  us-central1-a  n1-standard-1               10.128.0.2   173.255.112.6   STAGING
    nginx-m8rf  us-central1-a  n1-standard-1               10.128.0.3   104.198.72.238  STAGING

<br/>

    // Now configure a firewall so that you can connect to the machines on port 80 via the EXTERNAL_IP addresses:
    $ gcloud compute firewall-rules create www-firewall --allow tcp:80

<br/>

You should be able to connect to each of the instances via their external IP addresses via http://EXTERNAL_IP/ shown as the result of running the previous command.

<br/>

### Create a Network Load Balancer

    // Create an L3 network load balancer targeting your instance group:
    $ gcloud compute forwarding-rules create nginx-lb \
         --region us-central1 \
         --ports=80 \
         --target-pool nginx-pool

<br/>

    $ gcloud compute forwarding-rules list
    NAME      REGION       IP_ADDRESS     IP_PROTOCOL  TARGET
    nginx-lb  us-central1  35.238.83.178  TCP          us-central1/targetPools/nginx-pool

<br/>

You can then visit the load balancer from the browser http://IP_ADDRESS/ where IP_ADDRESS is the address shown as the result of running the previous command.

<br/>

### Create a HTTP(s) Load Balancer

    // First, create a health check. Health checks verify that the instance is responding to HTTP or HTTPS traffic:
    $ gcloud compute http-health-checks create http-basic-check

    Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-3a84410d949cb8b7/global/httpHealthChecks/http-basic-check].
    NAME              HOST  PORT  REQUEST_PATH
    http-basic-check        80    /

<br/>

    // Define an HTTP service and map a port name to the relevant port for the instance group. Now the load balancing service can forward traffic to the named port:
    $ gcloud compute instance-groups managed \
       set-named-ports nginx-group \
       --named-ports http:80

<br/>

    // Create a backend service:
    $ gcloud compute backend-services create nginx-backend \
      --protocol HTTP --http-health-checks http-basic-check --global

<br/>

    // Add the instance group into the backend service:
    $ gcloud compute backend-services add-backend nginx-backend \
    --instance-group nginx-group \
    --instance-group-zone us-central1-a \
    --global

<br/>

    // Create a default URL map that directs all incoming requests to all your instances:
    $ gcloud compute url-maps create web-map \
    --default-service nginx-backend

<br/>

    // Create a target HTTP proxy to route requests to your URL map:
    $ gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map

<br/>

    // Create a global forwarding rule to handle and route incoming requests. A forwarding rule sends traffic to a specific target HTTP or HTTPS proxy depending on the IP address, IP protocol, and port specified. The global forwarding rule does not support multiple ports.
    $ gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80

<br/>

After creating the global forwarding rule, it can take several minutes for your configuration to propagate.

    $ gcloud compute forwarding-rules list

    NAME               REGION       IP_ADDRESS      IP_PROTOCOL  TARGET
    http-content-rule               35.201.127.153  TCP          http-lb-proxy
    nginx-lb           us-central1  35.238.83.178   TCP          us-central1/targetPools/nginx-pool

Take note of the http-content-rule IP_ADDRESS for the forwarding rule.

From the browser, you should be able to connect to http://IP_ADDRESS/. It may take three to five minutes. If you do not connect, wait a minute then reload the browser.
