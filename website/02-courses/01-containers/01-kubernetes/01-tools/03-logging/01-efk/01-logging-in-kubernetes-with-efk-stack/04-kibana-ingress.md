---
layout: page
title: Logging in Kubernetes with EFK Stack | Запускаем kibana ingress
description: Logging in Kubernetes with EFK Stack | Запускаем kibana ingress
keywords: courses, containers, kubernetes, tools, logging, elastic, fluentd, kibana, ingress
permalink: /courses/containers/kubernetes/tools/logging/efk/logging-in-kubernetes-with-efk-stack/kibana-ingress/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack : Запускаем kibana ingress

<br/>

Делаю:  
2024.04.13

<br/>

```
$ export INGRESS_HOST=$(minikube --profile ${PROFILE} ip)
$ echo ${INGRESS_HOST}
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: kibana-cluster-ip
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 5601
  selector:
    app: kibana
EOF
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kibana-ingress
  annotations:
    nginx.ingress.kubernetes.io/default-backend: ingress-nginx-controller
    kubernetes.io/ingress.class: nginx
    ## tells ingress to check for regex in the config file
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
  - host: ${INGRESS_HOST}.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kibana-cluster-ip
            port:
              number: 8080
EOF
```

<br/>

```
$ kubectl get ingress
NAME             CLASS    HOSTS                 ADDRESS   PORTS   AGE
kibana-ingress   <none>   192.168.49.2.nip.io             80      11s
```

<br/>

```
$ echo ${INGRESS_HOST}.nip.io
```

<br/>

```
// [OK!]
http://192.168.49.2.nip.io/
```

<br/>

```
// Удалить, если не нужен
// $ kubectl delete ingress kibana-ingress
```
