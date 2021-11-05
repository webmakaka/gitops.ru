---
layout: page
title: FluxCD v2 With GitOps Toolkit - Kubernetes Deployment And Sync Mechanism
description: FluxCD v2 With GitOps Toolkit - Kubernetes Deployment And Sync Mechanism
keywords: linux, kubernetes, FluxCD
permalink: /study/videos/containers/kubernetes/tools/ci-cd/fluxcd/fluxcd-v2-with-gitops-toolkit/
---

# FluxCD v2 With GitOps Toolkit - Kubernetes Deployment And Sync Mechanism

<br/>

https://www.youtube.com/watch?v=R6OeIgb7lUI

<br/>

**Original gist**
https://gist.github.com/vfarcic/0431989df4836eb82bdac0cc53c7f3d6

<br/>

```
#########
# Setup
#########
```

<br/>

```
$ curl -s https://toolkit.fluxcd.io/install.sh | sudo bash

$ export INGRESS_HOST=$(minikube --profile my-profile ip)

$ echo ${INGRESS_HOST}

$ export GITHUB_USER=<YOUR_GITHUB_USERNAME>

$ export GITHUB_TOKEN=<YOUR_TOKEN>
```

<!--

$ export GITHUB_PERSONAL=true

-->

```
##############################
# Creating environment repos
##############################
```

<br/>

```
$ cd ~
$ mkdir -p flux-production
$ mkdir -p flux-staging
```

<!--
$ mkdir -p devops-toolkit

-->

<br/>

**GitHub**

```
create repo flux-staging
create repo flux-production
```

<br/>

```
$ cd flux-production/
$ git clone https://github.com/${GITHUB_USER}/flux-production .
$ mkdir apps

$ cd flux-staging/
$ git clone https://github.com/${GITHUB_USER}/flux-staging .
$ mkdir apps
```

<br/>

```
$ kubectl create namespace staging
$ kubectl create namespace production
```

<br/>

```
#################
# Bootstrapping`
#################
```

<br/>

```
$ cd ~
```

<br/>

```
$ flux bootstrap github \
    --owner $GITHUB_USER \
    --repository flux-fleet \
    --branch master \
    --path apps \
    --personal
```

<br/>

```
$ kubectl --namespace flux-system \
    get pods
```

<br/>

```
NAME                                      READY   STATUS    RESTARTS   AGE
helm-controller-86d6475c46-ds6kn          1/1     Running   0          6m37s
kustomize-controller-689f679f79-7fjgp     1/1     Running   0          6m37s
notification-controller-b8fbd5997-g9c6h   1/1     Running   0          6m37s
source-controller-5bb54b4c66-rdcfp        1/1     Running   0          6m37s
```

<br/>

```
$ cd ~
```

<br/>

```
$ git clone \
    https://github.com/${GITHUB_USER}/flux-fleet.git

$ cd flux-fleet

$ ls -1 apps

$ ls -1 apps/flux-system
```

<br/>

```
####################
# Creating sources`
####################
```

<br/>

```
$ flux create source git staging \
    --url https://github.com/$GITHUB_USER/flux-staging \
    --branch master \
    --interval 30s \
    --export \
    | tee apps/staging.yaml
```

<br/>

```
$ flux create source git production \
    --url https://github.com/$GITHUB_USER/flux-production \
    --branch master \
    --interval 30s \
    --export \
    | tee apps/production.yaml
```

<br/>

```
$ flux create source git devops-toolkit \
    --url https://github.com/vfarcic/devops-toolkit \
    --branch master \
    --interval 30s \
    --export \
    | tee apps/devops-toolkit.yaml
```

<br/>

```
$ flux create kustomization staging \
    --source staging \
    --path "./" \
    --prune true \
    --validation client \
    --interval 10m \
    --export \
    | tee -a apps/staging.yaml
```

<br/>

```
$ flux create kustomization production \
    --source production \
    --path "./" \
    --prune true \
    --validation client \
    --interval 10m \
    --export \
    | tee -a apps/production.yaml
```

<br/>

```
$ git add --all

$ git commit -m "Added environments"

$ git push
```

<br/>

```
$ watch flux get sources git

$ watch flux get kustomizations

$ cd ..

```

<br/>

```
###############################
# Deploying the first release
###############################

```

```
$ cd flux-staging
```

<br/>

```
$ echo "image:
tag: 2.9.9
ingress:
host: staging.devops-toolkit.$INGRESS_HOST.xip.io" \
 | tee values.yaml
```

<br/>

```
$ flux create helmrelease \
 devops-toolkit-staging \
 --source GitRepository/devops-toolkit \
 --values values.yaml \
 --chart "helm" \
 --target-namespace staging \
 --interval 30s \
 --export \
 | tee apps/devops-toolkit.yaml
```

<br/>

```
$ rm values.yaml

$ git add --all

$ git commit -m "Initial commit"

$ git push
```

<br/>

```
$ watch flux get helmreleases
```

need to wait

```
NAME READY MESSAGE
REVISION SUSPENDED
devops-toolkit-staging False HelmChart 'flux-system/flux-system-devops-toolki
t-staging' is not ready False
```

```
$ kubectl --namespace staging \
 get pods
```

<br/>

```
NAME READY STATUS RESTARTS AGE
staging-devops-toolkit-staging-devops-toolkit-76b88d8899-j9b8x 1/1 Running 0 3m5s
```

<br/>

```
##########################
# Deploying new releases
##########################
```

<br/>

```
$ cd flux-staging/

$ cat apps/devops-toolkit.yaml \
 | sed -e "s@tag: 2.9.9@tag: 2.9.17@g" \
 | tee apps/devops-toolkit.yaml

$ git add --all

$ git commit -m "Upgrade to 2.9.17"

$ git push

$ watch kubectl --namespace staging \
 get pods

$ watch kubectl --namespace staging \
 get deployment \
 staging-devops-toolkit-devops-toolkit \
 --output jsonpath="{.spec.template.spec.containers[0].image}"

$ cd ..
```

<br/>

```
###########################
# Promoting to production
###########################
```

<br/>

```
$ cd flux-production

$ mkdir apps
```

```
$ echo "image:
tag: 2.9.17
ingress:
host: devops-toolkit.$INGRESS_HOST.xip.io" \
 | tee values.yaml
```

<br/>

```
$ flux create helmrelease \
 devops-toolkit-production \
 --source GitRepository/devops-toolkit \
 --values values.yaml \
 --chart "helm" \
 --target-namespace production \
 --interval 30s \
 --export \
 | tee apps/devops-toolkit.yaml
```

<br/>

```
$ rm values.yaml

$ git add --all

$ git commit -m "Initial commit"

$ git push

$ watch flux get helmreleases

$ kubectl --namespace production \
 get pods
```

<br/>

```
#########################
# Destroying Everything
#########################
```

<br/>

```
$ minikube delete

$ cd ..

$ cd flux-fleet

$ gh repo view --web

$ # Delete the repo

$ cd ..

$ rm -rf flux-fleet

$ cd flux-staging

$ gh repo view --web

$ # Delete the repo

$ cd ..

$ rm -rf flux-staging

$ cd flux-production

$ gh repo view --web

# Delete the repo

$ cd ..

$ rm -rf flux-production
```
