---
layout: page
title: Ngrok Ingress Controller for Kubernetes (Доступ к kubernetes кластеру из интернетов)
description: Ngrok Ingress Controller for Kubernetes (Доступ к kubernetes кластеру из интернетов)
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu, ngrock, ingress
permalink: /tools/containers/kubernetes/minikube/ngrok-ingress-controller/
---

# Ngrok Ingress Controller for Kubernetes (Доступ к kubernetes кластеру из интернетов)

<br/>

**Делаю:**  
2024.11.11

Ни за что денег не платил. Все бесплатно, т.е. даром!

<br/>

**Статья:**  
https://ngrok.com/blog-post/ngrok-k8s

<br/>

https://dashboard.ngrok.com/api - создать api key

<br/>

```
$ helm repo add ngrok https://ngrok.github.io/kubernetes-ingress-controller
```

<br/>

```
// https://dashboard.ngrok.com/api-keys
// Я тупанул и вводил Key ID, а оно оказывается != api key (Хоть и копируется)
// Нужно новый api key создать он будет отображаться пока не покинуть страницу
$ export NGROK_API_KEY=[YOUR Secret API KEY]

// https://dashboard.ngrok.com/get-started/your-authtoken
$ export NGROK_AUTHTOKEN=[YOUR Secret Auth Token]
```

<br/>

```
$ helm install ngrok-ingress-controller ngrok/kubernetes-ingress-controller \
   --set credentials.apiKey=${NGROK_API_KEY} \
   --set credentials.authtoken=${NGROK_AUTHTOKEN}
```

<br/>

### Пример на основе приложения cats-app

<br/>

**Делаю:**  
2024.11.10

<br/>

```
$ cd ~/tmp
$ git clone https://github.com/webmakaka/cats-app
$ cd cats-app/k8s/
```

<br/>

```
$ kubectl apply -f ./cats-app-deployment.yaml
$ kubectl apply -f ./cats-app-cluster-ip-service.yaml
```

<br/>

```
// https://dashboard.ngrok.com/cloud-edge/domains - копируем домен
$ export NGROK_DOMAIN="hugely-amusing-owl.ngrok-free.app"
```

<br/>

```yaml
$ envsubst << 'EOF' | cat | kubectl create -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cats-app-ingress-service
spec:
  ingressClassName: ngrok
  rules:
    - host: ${NGROK_DOMAIN}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: minikube-cats-app-cluster-ip-service
                port:
                  number: 8080
EOF
```

<br/>

```
https://dashboard.ngrok.com/cloud-edge/endpoints
```

<br/>

Перехожу по URL - приложение открылось
