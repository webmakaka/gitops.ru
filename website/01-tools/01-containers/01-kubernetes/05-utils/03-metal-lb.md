---
layout: page
title: Инсталляция Metal LB
description: Инсталляция Metal LB
keywords: gitops, containers, kubernetes, metal lb
permalink: /tools/containers/kubernetes/utils/metal-lb/
---

# Инсталляция Metal LB

<br/>

**Делаю:**  
2025.04.27

<br/>

Metal LB позволит получить внешний IP в миникубе на локалхосте. Аналогично тому, как это происходит в облаках, когда облачный сервис выделяет ip адрес, к котому можно будет подключиться извне.

<br/>

```
$ LATEST_VERSION=$(curl --silent "https://api.github.com/repos/metallb/metallb/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')


// v0.11.0 - обычно работал с этой версией
// v0.12.1 - последняя с которой пробовал. В следующих версиях как-то по-другому нужно устанавливать.

$ export LATEST_VERSION=v0.12.1
$ echo ${LATEST_VERSION}
```

<br/>

```
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/${LATEST_VERSION}/manifests/namespace.yaml

$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/${LATEST_VERSION}/manifests/metallb.yaml

# On first install only
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

<br/>

### Minikube

```
$ minikube --profile ${PROFILE} ip
192.168.49.2
```

<br/>

Задаем диапазон ip адресов, которые можно выдать виртуальному сервису. Нужно, чтобы он был в той же подсети, что и ip minikube.

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: custom-ip-space
      protocol: layer2
      addresses:
      - 192.168.49.20-192.168.49.30
EOF
```

<br/>

### Kind

<br/>

```
$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' kind-worker
172.18.0.3
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: custom-ip-space
      protocol: layer2
      addresses:
      - 172.18.0.20-172.18.0.30
EOF
```

<!-- <br/>

```
$ export INGRESS_HOST=$(kubectl \
 --namespace istio-system \
 get service istio-ingressgateway \
 --output jsonpath="{.status.loadBalancer.ingress[0].ip}")

$ echo ${INGRESS_HOST}
```

<br/>

```
$ kubectl get pods --all-namespaces
``` -->
