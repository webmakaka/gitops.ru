---
layout: page
title: Logging in Kubernetes with EFK Stack | The Complete Guide
description: Logging in Kubernetes with EFK Stack | The Complete Guide
keywords: courses, containers, kubernetes, tools, logging, elastic, fluentd, kibana
permalink: /courses/containers/kubernetes/tools/logging/efk/logging-in-kubernetes-with-efk-stack/other/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack | The Complete Guide [ENG, 2021]

<br/>

Делаю:  
2024.04.06

<br/>

### Публикация сервиса в интернет без использования ingress

<br/>

```
// --namespace logging
$ kubectl port-forward deployment/kibana 5601:5601
```

<br/>

### Публикация сервиса в интернет с помощью Ingress (хз зачем, ведь и так работает)

<br/>

```
$ kubectl --namespace logging get svc
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
elasticsearch-master            ClusterIP   10.106.92.197   <none>        9200/TCP,9300/TCP    4m55s
elasticsearch-master-headless   ClusterIP   None            <none>        9200/TCP,9300/TCP    4m55s
fluentd-aggregator              ClusterIP   10.108.30.236   <none>        9880/TCP,24224/TCP   3m42s
fluentd-forwarder               ClusterIP   10.97.175.200   <none>        9880/TCP             3m42s
fluentd-headless                ClusterIP   None            <none>        9880/TCP,24224/TCP   3m42s
kibana-kibana                   ClusterIP   10.105.41.51    <none>        5601/TCP             4m14s
```

<br/>

**Публикация сервиса kibana-kibana в неймспейсе logging, чтобы к нему можно было обращаться из ingress**

```
externalName: kibana-kibana.logging.svc.cluster.local
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: kibana-service
spec:
  type: ExternalName
  externalName: kibana-kibana.default.svc.cluster.local
EOF
```

<br/>

```
$ export INGRESS_HOST=$(minikube --profile ${PROFILE} ip)
$ echo ${INGRESS_HOST}
```

<br/>

**kibana-ingress**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/default-backend: ingress-nginx-controller
    kubernetes.io/ingress.class: nginx
    ## tells ingress to check for regex in the config file
    nginx.ingress.kubernetes.io/use-regex: "true"
  name: kibana-ingress
spec:
  rules:
    - host: ${INGRESS_HOST}.nip.io
      http:
        paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: kibana-service
                port:
                  number: 5601
EOF
```

<br/>

```
Warning: annotation "kubernetes.io/ingress.class" is deprecated, please use 'spec.ingressClassName' instead
```

<br/>

```
$ kubectl get ingress
NAME             CLASS    HOSTS                 ADDRESS   PORTS   AGE
kibana-ingress   <none>   192.168.49.2.nip.io             80      5s
```

<br/>

```
$ curl -I ${INGRESS_HOST}.nip.io
OK!
```

<br/>

```
// Удалить, если не нужен
// $ kubectl delete inggress kibana-ingress
```
