---
layout: page
title: Linkerd
description: Linkerd
keywords: devops, containers, kubernetes, service mesh, Linkerd
permalink: /tools/containers/kubernetes/utils/service-mesh/linkerd/
---

# Linkerd

<br/>

### What Is Linkerd Service Mesh? Linkerd Tutorial Part 1

https://www.youtube.com/watch?v=mDC3KA_6vfg

https://gist.github.com/vfarcic/249f8bb8baa8ca03ff5a1a61f3bda200

<br/>

**Пример:**

1. [Подключение к бесплатному облаку от Google](/tools/clouds/google/google-cloud-shell/setup/)

2. Инсталляция [MiniKube](/tools/containers/kubernetes/minikube/setup/)

**Испольновалась версия KUBERNETES_VERSION=v1.22.3**

3. Инсталляция [Kubectl](/tools/containers/kubernetes/utils/kubectl/)

4. [Инсталляция Linkerd](/tools/containers/kubernetes/utils/service-mesh/linkerd/setup/)

<br/>

```
$ cd ~/tmp/
$ git clone https://github.com/vfarcic/linkerd-demo
$ cd linkerd-demo/
```

<br/>

```
$ export INGRESS_IP=$(minikube ip --profile ${PROFILE})

$ echo ${INGRESS_IP}

$ cat kustomize/overlays/production/ingress.yaml \
    | sed -e "s@host: .*@host: dot.${INGRESS_IP}.nip.io@g" \
    | tee kustomize/overlays/production/ingress.yaml

$ kubectl create namespace production
```

<br/>

```
###########################
# Linkerd Proxy Injection #
###########################
```

<br/>

```
$ cat kustomize/base/deployment.yaml

$ cat kustomize/overlays/production/ingress.yaml

$ kubectl apply \
    --kustomize kustomize/overlays/production

# It could be `linkerd inject -` instead; a bad idea
```

<br/>

```
$ kubectl --namespace production get pods
NAME                            READY   STATUS    RESTARTS   AGE
devops-toolkit-98f5fbf9-hbjbt   2/2     Running   0          2m29s
devops-toolkit-98f5fbf9-pwmhm   2/2     Running   0          2m20s
devops-toolkit-98f5fbf9-wwglh   2/2     Running   0          2m25s
```

<br/>

```
$ echo http://dot.${INGRESS_IP}.nip.io
```

<br/>

```
$ curl \
    -o /dev/null -s -w "%{http_code}\n" \
    http://dot.${INGRESS_IP}.nip.io
```

<!--

<br/>

```
$ curl \
 -o /dev/null -s -w "%{http_code}\n" \
 -H "Host: dot.${INGRESS_IP}.nip.io" \
 "http://192.168.49.2"
```
-->

<br/>

```
# Open it in a browser

##############################
# Observability With Linkerd #
##############################

# Install the latest release of the Ddosify CLI from https://github.com/ddosify/ddosify/releases


# For Debian based (Ubuntu, Linux Mint, etc.)
$ cd ~/tmp
$ wget https://github.com/ddosify/ddosify/releases/download/v0.1.1/ddosify_amd64.deb
$ sudo dpkg -i ddosify_amd64.deb

$ ddosify -t "http://dot.$INGRESS_IP.nip.io"

$ linkerd --namespace production \
    viz stat deploy/devops-toolkit

$ linkerd --namespace production \
    viz top deploy/devops-toolkit

$ linkerd viz dashboard & ddosify -config ddosify.json
```

<br/>

### Introduction to Linkerd for beginners | a Service Mesh

https://www.youtube.com/watch?v=Hc-XFPHDDk4
