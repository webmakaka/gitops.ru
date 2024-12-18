---
layout: page
title: Запуск демонстрационного приложения Istio - bookinfo
description: Запуск демонстрационного приложения Istio - bookinfo
keywords: devops, containers, kubernetes, service-mesh, bookinfo
permalink: /tools/containers/kubernetes/utils/service-mesh/istio/bookinfo/
---

# Запуск демонстрационного приложения Istio - bookinfo

<br/>

Делаю:  
08.11.2021

<br/>

Istio и minikube установлены и настроены как <a href="/tools/containers/kubernetes/utils/service-mesh/istio/setup/">здесь</a>

<br/>

```
$ export LATEST_VERSION=$(curl --silent "https://api.github.com/repos/istio/istio/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

$ cd ~/tmp/istio-${LATEST_VERSION}/
```

<br/>

```
$ kubectl create -f samples/bookinfo/platform/kube/bookinfo.yaml

$ kubectl get services | grep productpage
productpage   ClusterIP   10.107.4.42      <none>        9080/TCP   58s
```

<br/>

```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-79f774bdb9-6p85v       2/2     Running   0          2m11s
productpage-v1-6b746f74dc-kl56s   2/2     Running   0          2m10s
ratings-v1-b6994bb9-nlrrc         2/2     Running   0          2m11s
reviews-v1-545db77b95-f6d8g       2/2     Running   0          2m10s
reviews-v2-7bf8c9648f-94528       2/2     Running   0          2m11s
reviews-v3-84779c7bbc-drcff       2/2     Running   0          2m11s
```

<br/>

```
$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
details-v1       1/1     1            1           2m42s
productpage-v1   1/1     1            1           2m42s
ratings-v1       1/1     1            1           2m42s
reviews-v1       1/1     1            1           2m42s
reviews-v2       1/1     1            1           2m42s
reviews-v3       1/1     1            1           2m42s
```

<br/>

```
$ kubectl create -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

<br/>

```
$ kubectl get gateway
NAME               AGE
bookinfo-gateway   7s

```

<br/>

```
$ kubectl describe gateway bookinfo-gateway
```

<br/>

```
$ kubectl -n istio-system get svc istio-ingressgateway
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.99.229.201   192.168.59.20   15021:30590/TCP,80:31488/TCP,443:30766/TCP,31400:32461/TCP,15443:32125/TCP   25m
```

<br/>

```
$ echo EXTERNAL-IP=$(kubectl --namespace istio-system get svc istio-ingressgateway --template="{{range .status.loadBalancer.ingress}}{{.ip}}{{end}}")
```

<br/>

```
$ curl http://192.168.49.20/productpage
OK!
```

<br/>

Должно быть что-то подобное (если правильно настроить доступ)

<br/>

![Istio](/img/tools/containers/kubernetes/utils/service-mesh/istio/bookinfo/pic-01.png 'Istio'){: .center-image }

<br/>

### Так тоже можно

```

$ export INGRESS_HOST=$(minikube ip --profile ${PROFILE})

$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

$ export GATEWAY_URL=${INGRESS_HOST}:${INGRESS_PORT}

$ echo "http://$GATEWAY_URL/productpage"

$ curl "http://$GATEWAY_URL/productpage"
```
