---
layout: page
title: Harbor
description: Harbor
keywords: gitops, containers, kubernetes,registries, harbor
permalink: /tools/containers/kubernetes/tools/registries/harbor/
---

# Harbor

<br/>

// Не заработало!  
**Делаю:**  
08.05.2023

<br/>

Инсталляция [MiniKube](/tools/containers/kubernetes/minikube/setup/)

**Испольновалась версия KUBERNETES_VERSION=v1.27.1**

<br/>

https://www.youtube.com/watch?v=f931M4-my1k

https://gist.github.com/vfarcic/0a322f969368bec74b75677da217291c

<!-- Signing And Verifying Container Images With Sigstore Cosign And Kyverno
https://www.youtube.com/watch?v=HLb1Q086u6M&t=0s -->

<br/>

## Setup

<br/>

```
$ export PROFILE=${USER}-minikube
$ export INGRESS_HOST=$(minikube --profile ${PROFILE} ip)
```

<br/>

```
$ echo ${INGRESS_HOST}
192.168.49.2
```

<br/>

```
$ helm repo add harbor https://helm.goharbor.io
$ helm repo update
```

<br/>

```
$ mkdir -p ~/tmp/harbor
$ cd ~/tmp/harbor
$ vi values.yaml
```

<br/>

```yaml
expose:
  tls:
    enabled: false
  ingress:
    annotations:
      ingress.kubernetes.io/proxy-body-size: '0'
      ingress.kubernetes.io/ssl-redirect: 'false'
      nginx.ingress.kubernetes.io/proxy-body-size: 0                                │
      nginx.ingress.kubernetes.io/ssl-redirect: 'false'
harborAdminPassword: Harbor12345
```

<br/>

```
$ helm upgrade --install harbor harbor/harbor \
    --namespace harbor \
    --create-namespace \
    --set expose.ingress.hosts.core=harbor.$INGRESS_HOST.nip.io \
    --set expose.ingress.hosts.notary=notary.$INGRESS_HOST.nip.io \
    --set externalURL=http://harbor.$INGRESS_HOST.nip.io \
    --values values.yaml \
    --wait

$ echo "http://harbor.$INGRESS_HOST.nip.io"
```

<br/>

```
// OK!
// User: admin
// Password: Harbor12345
http://harbor.192.168.49.2.nip.io
```

<br/>

```
# `Administration` > `Registries` > `+ NEW ENDPOINT` > Add Docker Hub registry
# `Projects` > `NEW PROJECT`
# - Project Name: dot
# - Press the `OK` button
# `Projects` > `dot` > `Configuration`
# - Check `Cosign` in `Deployment Security`
# - Check `Prevent vulnerable images from running` in `Deployment Security` and set the severity to `High`.
# - Set `Automatically scan images on push` in `Vulnerability scanning`
```

<br/>

## Build And Push Container (Docker) Images

<br/>

```
$ export PROFILE=${USER}-minikube
$ export INGRESS_HOST=$(minikube --profile ${PROFILE} ip)
$ echo harbor.$INGRESS_HOST.nip.io
```

<br/>

```
// admin / Harbor12345
$ docker login --username admin harbor.$INGRESS_HOST.nip.io
Error response from daemon: Get "https://harbor.192.168.49.2.nip.io/v2/": x509: certificate is valid for ingress.local, not harbor.192.168.49.2.nip.io
```

<br/>

```
// admin / Harbor12345
$ docker login --username admin harbor.$INGRESS_HOST.nip.io --insecure-registry
unknown flag: --insecure-registry
See 'docker login --help'.
```

<!-- <br/>
<br/>

```
$ git clone https://github.com/vfarcic/harbor-demo
$ cd harbor-demo/
```

<br/>

```
$ yq --inplace \
    ".image.repository = \"harbor.$INGRESS_HOST.nip.io/dot/silly-demo\"" \
    helm/values.yaml

$ yq --inplace \
    ".ingress.host = \"silly-demo.$INGRESS_HOST.nip.io\"" \
    helm/values.yaml
``` -->
