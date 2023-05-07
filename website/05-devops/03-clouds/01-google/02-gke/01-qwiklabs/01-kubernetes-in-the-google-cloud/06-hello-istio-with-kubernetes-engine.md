---
layout: page
title: Hello Istio with Kubernetes Engine
description: Hello Istio with Kubernetes Engine
keywords: Hello Istio with Kubernetes Engine
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/hello-istio-with-kubernetes-engine/
---

# [GSP135] Hello Istio with Kubernetes Engine

<br/>

https://google.qwiklabs.com/focuses/616?parent=catalog

<br/>

Делаю!  
04.10.2019

<br/>

### Introduction

Istio is an open source framework for connecting, securing, and managing microservices, including services running on Google Kubernetes Engine (GKE). It lets you create a network of deployed services with load balancing, service-to-service authentication, monitoring, and more, without requiring any changes in service code.

You'll add Istio support to services by deploying a special sidecar proxy to each of your application's Pods. The proxy intercepts all network communication between microservices and is configured and managed using Istio's control plane functionality.

This lab shows you how to install and configure Istio on Kubernetes Engine, deploy an Istio-enabled multi-service application, and dynamically change request routing.

<br/>

### Setup your Kubernetes/GKE cluster

    $ gcloud config set compute/zone us-central1-f

    $ gcloud container clusters create hello-istio \
    --num-nodes 4

    // Grant admin permissions in the cluster to the current gcloud user
    $ kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole=cluster-admin \
    --user=$(gcloud config get-value core/account)

<br/>

### Installing Istio

Now, install Istio. Istio's control plane is installed in its own Kubernetes istio-system namespace, and can manage microservices from all other namespaces. The installation includes Istio core components, tools, and samples.

    $ curl -L https://git.io/getLatestIstio | ISTIO_VERSION=0.5.1 sh -

<br/>

The installation directory contains the following files which we'll use:

-   Installation .yaml files for Kubernetes in install/kubernetes
-   Sample applications in samples/
-   The istioctl client binary in the bin/ directory. Similar to kubectl for Kubernetes, this is the tool used to manage Istio, including network routing and security policies.
-   The istio.VERSION configuration file

<br/>

    $ cd istio-0.5.1/

    $ export PATH=$PWD/bin:$PATH

<br/>

### Installing the core components

Next you'll install Istio's core components and the optional Istio Auth components, which enable mutual TLS authentication between the sidecars:

    $ kubectl apply -f install/kubernetes/istio-auth.yaml

This creates the istio-system namespace along with the required RBAC permissions, and deploys the four primary Istio control plane components:

-   Pilot: Handles configuration and programming of the proxy sidecars.
-   Mixer: Handles policy decisions for your traffic and gathers telemetry.
-   Ingress: Handles incoming requests from outside your cluster.
-   CA: the Certificate Authority.

<br/>

### Verifying the installation

    $ kubectl get svc -n istio-system
    $ kubectl get pods -n istio-system

<br/>

### Deploying an application

Now that Istio is installed and verified, you can deploy one of the sample applications provided with the installation — BookInfo. This is a simple mock bookstore application made up of four microservices - all managed using Istio. Each microservice is written in a different language, to demonstrate how you can use Istio in a multi-language environment, without any changes to code.

The microservices are:

-   productpage: calls the details and reviews microservices to populate the page.
-   details: contains book information.
-   reviews: contains book reviews. It also calls the ratings microservice.
-   ratings: contains book ranking information that accompanies a book review.

There are 3 versions of the reviews microservice:

-   Reviews v1 doesn't call the ratings service.
-   Reviews v2 calls the ratings service, and displays each rating as 1 - 5 black stars.
-   Reviews v3 calls the ratings service, and displays each rating as 1 - 5 red stars.

The end-to-end architecture of the application looks like this:

![Hello Istio with Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/hello-istio-with-kubernetes-engine/pic1.png 'Hello Istio with Kubernetes Engine'){: .center-image }

You will find the source code and all the other files used in this example in your Istio samples/bookinfo <a href="https://github.com/istio/istio/tree/master/samples/bookinfo">samples/bookinfo</a> directory.

Have a look at the .yaml which describes the bookInfo application:

    $ cat samples/bookinfo/kube/bookinfo.yaml

Note how there are Deployments, Services, and an Ingress to deploy the BookInfo application, but there is nothing Istio-specific here at all. If you were to deploy the application as it is, it would work, but it would not have any Istio functionality.

You will use the following Istio command to inject the proxy sidecar along with each Pod that is deployed. istioctl kube-inject takes a Kubernetes YAML file as input, and outputs a version of that YAML which includes the Istio proxy server.

    $ istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml

Look at one of the Deployments. Now it contains a second container, the Istio sidecar, along with all the configuration necessary.

You can take the output from istioctl kube-inject and feed it directly to kubectl to create the objects with their sidecars:

    $ kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)

Finally, confirm that the application has been deployed correctly by running the following commands:

    $ kubectl get services
    $ kubectl get pods

You may need to re-run this command until you see that all of the pods are in Running status.

<br/>

### Use the application

    $ kubectl get ingress

    $ export GATEWAY_URL=<your gateway IP>

    $ curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage

http://<your gateway IP>/productpage

![Hello Istio with Kubernetes Engine](/img/devops/clouds/google/gke/qwiklabs/kubernetes-in-the-google-cloud/hello-istio-with-kubernetes-engine/pic2.png 'Hello Istio with Kubernetes Engine'){: .center-image }

Refresh the page several times. Notice how you see three different versions of reviews! If you refer back to the diagram, you will see there are three different book review services which are being called in a round-robin style - no stars, black stars, and red stars. This is normal Kubernetes balancing behavior.

Istio can also be used to control which users are routed to which version of the services.

<br/>

### Dynamically change request routing

When you were accessing the application, you saw the three versions of reviews. This is because without an explicit default version set, Istio will route requests to all available versions of a service, in a round-robin fashion.

Route rules control how requests are routed within an Istio service mesh. Requests can be routed based on the source and destination, HTTP header fields, and weights associated with individual service versions.

Now you'll use the istioctl command line tool to control routing.

**Static routing**

Let's add a route rule that says all traffic should go to v1 of the reviews service.

First, confirm that there are no route rules installed:

    $ istioctl get routerules

This example of a route rule will route all traffic for a service named reviews to Pods running v1 of that service, as identified by Kubernetes labels.

```

apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: reviews-default
spec:
  destination:
    name: reviews
  precedence: 1
  route:
  - labels:
      version: v1

```

BookInfo includes a sample with rules for all available services. Install it by running:

    $ istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml

Confirm that four rules were created:

    $ istioctl get routerules

Go back to the BookInfo application (http://\$GATEWAY_URL/productpage) in your browser. Refresh a few times. Do you see any stars? You should see the book review with no rating stars because the service reviews:v1 does not have any stars.

<br/>

**Dynamic routing**

As the mesh operates at Layer 7, you can use HTTP attributes (paths or cookies) to decide on how to route a request.

In this example, a rule which routes certain users (in this case, Jason) to a service (v2) based on a cookie, looks like this:

```
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: reviews-test-v2
spec:
  destination:
    name: reviews
  precedence: 2
  match:
    request:
      headers:
        cookie:
          regex: "^(.*?;)?(user=jason)(;.*)?$"
  route:
  - labels:
      version: v2
```

Create the rule from another sample file:

    $ istioctl create -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml

List the rules and make sure it's there:

    $ istioctl get routerule

Use -o yaml to see the full output:

    $ istioctl get routerule reviews-test-v2 -o yaml

Now you have routed requests from the user "jason" to use the reviews:v2 service.

Test how the page behavior changes with this new rule.

Log in as user "jason" on the product page web page by clicking the Sign in button at the top of the screen, typing jason as the user name - you don't need a password - then clicking Sign in.

Refresh the browser. You should now see black ratings stars next to each review.

If you try logging in as any other user (log out as Jason and sign in as Kylie), or don't log in at all, you will continue to see reviews: v1.

Run these commands to remove the routing rules:

    $ istioctl delete -f samples/bookinfo/kube/route-rule-all-v1.yaml
    $ istioctl delete -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml

You can go back to the tab with the web app to see that when you refresh the page, you're back to cycling through the 3 types of Reviews available.
