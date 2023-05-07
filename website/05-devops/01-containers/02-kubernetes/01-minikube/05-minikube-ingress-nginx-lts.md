---
layout: page
title: Пример Ingress в minikube (Nginx) + TLS
description: Пример Ingress в minikube (Nginx) + TLS
keywords: devops, linux, kubernetes,  Пример Ingress в minikube (Nginx) + TLS
permalink: /devops/containers/kubernetes/kubeadm/minikube-ingress-nginx/tls/
---

# Пример Ingress в minikube (Nginx) + TLS

Делаю: 23.04.2019

<br/>

По материалам:

https://www.youtube.com/watch?v=7K0gAYmWWho&list=PLShDm2AZYnK3cWZpOjV7nOpL7plH2Ztz0&index=2

<br/>

    $ minikube start
    $ minikube addons enable ingress

    $ kubectl get pods -n kube-system | grep ingress
    nginx-ingress-controller-7c66d668b-vjnqc   1/1     Running   0          4m22s

<br/>

    $ kubectl run nginx --image=nginx
    $ kubectl expose deployment nginx --port 80

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: example.com
    http:
      paths:
        - backend:
            serviceName: nginx
            servicePort: 80
EOF

```

<br/>

```
$ echo "$(minikube ip) example.com" | sudo tee -a /etc/hosts

$ curl example.com
OK
```

<br/>

## Самоподписанный сертификат

<br/>

### Генерим сертификат

```
$ cd ~/tmp

$ openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.crt -subj "/CN=example.com" -days 365
```

<br/>

### Создаем секрет cо сгенерированным сертификатом

```
$ kubectl create secret tls example-com-tls --cert=tls.crt --key=tls.key

// удалить потом можно командой
$ kubectl delete secret example-com-tls

$ kubectl get secret -o yaml
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx
spec:
  tls:
    - secretName: example-com-tls
      hosts:
        - example.com
  rules:
  - host: example.com
    http:
      paths:
        - backend:
            serviceName: nginx
            servicePort: 80
EOF
```

<br/>

```
$ curl -k https://example.com
$ curl --cacert tls.crt https://example.com
```

<br/>

### Automatically Provision TLS Certificates in K8s with cert-manager

https://www.youtube.com/watch?v=JJTJfl-V_UM&list=PLShDm2AZYnK3cWZpOjV7nOpL7plH2Ztz0&index=3

<br/>

https://github.com/jetstack/cert-manager

    // удалить потом можно командой
    $ kubectl delete secret example-com-tls

    $ kubectl delete ing nginx

<br/>

    // У меня уже был установлен локально
    $ helm init

<br/>

    https://docs.cert-manager.io/en/latest/getting-started/install.html

<br/>

    # Install the CustomResourceDefinition resources separately
    kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.7/deploy/manifests/00-crds.yaml

    # Create the namespace for cert-manager
    kubectl create namespace cert-manager

    # Label the cert-manager namespace to disable resource validation
    kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

    # Add the Jetstack Helm repository
    helm repo add jetstack https://charts.jetstack.io

    # Update your local Helm chart repository cache
    helm repo update

    # Install the cert-manager Helm chart
    helm install \
    --name cert-manager \
    --namespace cert-manager \
    --version v0.7.0 \
    jetstack/cert-manager

<br/>

    $ kubectl get crd
    NAME                                CREATED AT
    certificates.certmanager.k8s.io     2019-04-23T01:34:01Z
    challenges.certmanager.k8s.io       2019-04-23T01:34:05Z
    clusterissuers.certmanager.k8s.io   2019-04-23T01:34:05Z
    issuers.certmanager.k8s.io          2019-04-23T01:34:05Z
    orders.certmanager.k8s.io           2019-04-23T01:34:05Z

<br/>

    $ openssl genrsa -out ca.key 2048
    $ cp /etc/ssl/openssl.cnf openssl-with-ca.cnf
    $ vi openssl-with-ca.cnf

<br/>

    https://github.com/jetstack/cert-manager/issues/279

<br/>

Вставляю перед [ usr_cert ]

    [ v3_ca ]
    basicConstraints = critical,CA:TRUE
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid:always,issuer:always

<br/>

    $ openssl req -x509 -new -nodes -key ca.key -sha256 -subj "/CN=sampleissuer.local" -days 1024 -out ca.crt -extensions v3_ca -config openssl-with-ca.cnf

<br/>

    $ kubectl create secret tls ca-key-pair --key=ca.key --cert=ca.crt

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: ca-issuer
  namespace: default
spec:
  ca:
    secretName: ca-key-pair
EOF
```

<br/>

    $ kubectl get issuer
    NAME        AGE
    ca-issuer   11s

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: example-com
  namespace: default
spec:
  secretName: example-com-tls
  issuerRef:
    name: ca-issuer
    kind: Issuer
  commonName: example.com
  dnsNames:
    - www.example.com
EOF

```

<br/>

    $ kubectl get certificate
    NAME
    example-com

    $ kubectl describe certificate example-com

    $ kubectl get secret
    NAME                  TYPE                                  DATA   AGE
    ca-key-pair           kubernetes.io/tls                     2      6m13s
    default-token-kjt2v   kubernetes.io/service-account-token   3      59m
    example-com-tls       kubernetes.io/tls                     3      94s

<br/>

    $ kubectl get secret example-com-tls -o yaml

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx
spec:
  tls:
    - secretName: example-com-tls
      hosts:
        - example.com
  rules:
  - host: example.com
    http:
      paths:
        - backend:
            serviceName: nginx
            servicePort: 80
EOF

```

    $ kubectl get ing nginx -o yaml

<br/>

https://example.com/

<br/>

### Использование сертификатов Let's Encrypt совместно с cert-manager.

https://www.youtube.com/watch?v=etC5d0vpLZE&list=PLShDm2AZYnK3cWZpOjV7nOpL7plH2Ztz0&index=4

Не доразобрался. Не хочется пока разбираться с доступом к сайту из интернета. И мне пока видится, что лучше поставить сертификат на haproxy сервер который будет стоять перед kubernetes кластером (по крайней мере на локальном, а не на облачном).

<!--

<br/>

### Use cert-manager with Let's Encrypt® Certificates Tutorial Automatic Browser-Trusted HTTPS

https://www.youtube.com/watch?v=etC5d0vpLZE&list=PLShDm2AZYnK3cWZpOjV7nOpL7plH2Ztz0&index=4

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: blablalba@mail.ru
    privateKeySecretRef:
        name: lesencrypt-staging
    http01: {}
EOF

```

    $ kubectl describe issuer letsencrypt-staging

Должно быть:

    Message:               The ACME account was registered with the ACME server
        Reason:                ACMEAccountRegistered

<br/> -->

<!-- $ certificate-staging.yaml -->
<!--
```
$ cat <<EOF | kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: youtube-lets-encrypt-staging
spec:
  secretName: youtube-lets-encrypt-staging-tls
  issuerRef:
    name; letsencrypt-staging
  commonName: youtube-lets-encrypt.kubacation.com
  acme:
    config:
        - http01:
            ingress: nginx
          domains:
            - youtube-lets-encypt.kubacation.com
EOF

```

    $ kubectl get certificate
    $ kubectl describe certificate youtube -->
