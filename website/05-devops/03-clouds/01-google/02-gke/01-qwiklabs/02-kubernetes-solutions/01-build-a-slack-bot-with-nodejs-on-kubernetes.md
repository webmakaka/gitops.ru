---
layout: page
title: Build a Slack Bot with Node.js on Kubernetes
description: Build a Slack Bot with Node.js on Kubernetes
keywords: Build a Slack Bot with Node.js on Kubernetes
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-solutions/build-a-slack-bot-with-nodejs-on-kubernetes/
---

# [GSP024] Build a Slack Bot with Node.js on Kubernetes

<br/>

Делаю:  
24.05.2019

https://www.qwiklabs.com/focuses/635?parent=catalog

<br/>

### Create a Slack bot user

**Create a new Slack app**

https://api.slack.com/apps

Click the Create New App button.

Name the app "Kittenbot".

Choose the Slack workspace where you want it installed.

Click Create App.

<br/>

**Add a new bot user to the app**

In the left pane, under Features, select Bot Users.

Click the Add a Bot User button.

Your default name will be "kittenbot", use this.

This lab uses the Realtime Messaging (RTM) API, so keep the Always Show My Bot as Online option Off. The bot user will show as online only when there is a connection from the bot.

Click Add Bot User.

<br/>

**Get the bot user OAuth access token**

-   Select OAuth & Permissions in the left pane.
-   Click Install App to Workplace. Click Authorize to confirm.
-   Click the Copy button to copy the Bot User OAuth Access Token text into your clipboard.

<br/>

### Create a Kubernetes cluster on Kubernetes Engine

    $ gcloud container clusters create my-cluster \
      --num-nodes=2 \
      --zone=us-central1-f \
      --machine-type n1-standard-1

    $ gcloud compute instances list

<br/>

### Get the sample code

    $ git clone https://github.com/googlecodelabs/cloud-slack-bot.git
    $ cd cloud-slack-bot/step-4-update

    $ vi slack-codelab-deployment.yaml

Прописать PROJECT_ID

    $ export PROJECT_ID=qwiklabs-gcp-23b76630c5841036

    $ docker build -t gcr.io/${PROJECT_ID}/slack-codelab:v2 .
    $ gcloud docker -- push gcr.io/${PROJECT_ID}/slack-codelab:v2


    // Bot User OAuth Access Token
    $ vi slack-token


    $ kubectl create secret generic slack-token --from-file=./slack-token

    $ kubectl apply -f slack-codelab-deployment.yaml

![Build a Slack Bot with Node.js on Kubernetes](/img/devops/clouds/google/gke/qwiklabs/kubernetes-solutions/build-a-slack-bot-with-nodejs-on-kubernetes/google-clouds-slack-integration.png 'Build a Slack Bot with Node.js on Kubernetes'){: .center-image }

<br/>

### Extra Credit: Create an incoming webhook to Slack

Go to the Slack apps management page. https://api.slack.com/apps

Click on your app Kittenbot, and then in the left panel, click Incoming Webhooks.

Toggle Activate Incoming Webhooks to On.

At the top, there's a message "You've changed the permissions scopes ...". Click click here.

From the Post to dropdown, select the Slack channel #general for messages to post to.

Click Authorize.

Copy the webhook URL and save it to your computer. You'll come back to this in a later step.

<br/>

Выполнить в командной строке (точный вариант в форме на сайте):

    $ curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' https://hooks.slack.com/services/${KEY}

И появится сообщение в general чате.
