---
layout: page
title: Kustomize - How to Simplify Kubernetes Configuration Management
description: Kustomize - How to Simplify Kubernetes Configuration Management
keywords: linux, kubernetes, kustomize
permalink: /devops/containers/kubernetes/packages/kustomize/how-to-simplify-kubernetes-configuration-management/
---

# Kustomize - How to Simplify Kubernetes Configuration Management

<br/>

https://www.youtube.com/watch?v=Twtbg6LFnAg&t=6s

https://gist.github.com/vfarcic/07b0b4642b5694d0239ee7c1629173ce

<br/>

```
#########
# Setup
#########
```

<br/>

https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/

<br/>

```
$ curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash && chmod +x kustomize && sudo mv kustomize /usr/local/bin/
```

<br/>

```
################################
# Exploring kustomization.yaml
################################
```

<br/>

```
git clone https://github.com/vfarcic/argocd-production.git

cd argocd-production

ls -1

ls -1 argo-workflows

cd argo-workflows

ls -1 base/

cat base/kustomization.yaml

https://github.com/argoproj/argo/blob/master/manifests/base/kustomization.yaml

https://github.com/argoproj/argo/blob/master/manifests/base/workflow-controller/workflow-controller-configmap.yaml

cat base/config.yaml
```

<br/>

```
###########################
# Applying base manifests
###########################
```

<br/>

```
$ kustomize build base

$ kustomize build base \
 | kubectl apply --filename -
```

<br/>

```
#####################
# Applying overlays
#####################
```

<br/>

```
$ kubectl --namespace argo get ingresses

$ ls -1 overlays/production/

$ cat overlays/production/kustomization.yaml

$ cat overlays/production/ingress_patch.json

$ kustomize build overlays/production \
 | kubectl apply --filename -

$ kubectl --namespace argo get ingresses
```

<br/>

```
##########################
# Upgrading applications
##########################
```

<br/>

```
$ kubectl --namespace argo \
    get deployment argo-server \
    --output yaml
```

<br/>

```
$ cd overlays/production
```

<br/>

```
$ kustomize edit set image \
    argoproj/argocli=argoproj/argocli:v2.12.4
```

<br/>

```
$ cat kustomization.yaml

$ cd ../../
```

<br/>

```
$ kustomize build \
    overlays/production \
    | kubectl apply --filename -
```

<br/>

```
$ kubectl --namespace argo \
    get deployment argo-server \
    --output yaml
```
