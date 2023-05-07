---
layout: page
title: Setup Lets Encrypt cert-manager in Kubernetes Bare Metal
description: Setup Lets Encrypt cert-manager in Kubernetes Bare Metal
keywords: devops, linux, kubernetes, Setup Lets Encrypt cert-manager in Kubernetes Bare Metal
permalink: /devops/containers/kubernetes/kubeadm/lets-encrypt/
---

# Setup Lets Encrypt cert-manager in Kubernetes Bare Metal

<br/>

Делаю:  
13.05.2019

По материалам из видео индуса:

https://www.youtube.com/watch?v=Hwqm1D2EfFU

<br/>

![Kubernetes Lets Encrypt](/img/devops/containers/kubernetes/kubeadm/lets-encrypt.png 'Kubernetes Lets Encrypt'){: .center-image }

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

Подготавливаем Ingress Ingress Controller как <a href="/devops/containers/kubernetes/kubeadm/ingress/nginxinc-kubernets-ingress-install/">здесь</a>.

<br/>

Устанавливаем haproxy как <a href="/devops/containers/kubernetes/kubeadm/ingress/haproxy/">здесь</a>.

Только конфиг нужно подправить:

```
#---------------------------------------------------------------------
# User defined
#---------------------------------------------------------------------

frontend http_front
  bind *:443
  mode tcp
  option tcplog
  default_backend http_back

backend http_back
  mode tcp
  balance roundrobin
  server kworker1 192.168.0.11:443
  server kworker2 192.168.0.12:443
```

<br/>

Устанавливаем helm/tiller.

UPD. Heml2 выпилен, предлагаю попробовать Helm3 как <a href="/devops/containers/kubernetes/packages/heml/setup/">здесь</a>.

<br/>

### Устанавливаем cert-manager

http://hub.helm.sh/charts/jetstack/cert-manager

<br/>

    $ kubectl apply \
        -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/deploy/manifests/00-crds.yaml

    $ helm repo add jetstack https://charts.jetstack.io

    $ helm repo list
    NAME    	URL
    stable  	https://kubernetes-charts.storage.googleapis.com
    local   	http://127.0.0.1:8879/charts
    jetstack	https://charts.jetstack.io


    $ helm install --name cert-manager --namespace cert-manager jetstack/cert-manager

    $ kubectl -n cert-manager get all
    NAME                                           READY   STATUS    RESTARTS   AGE
    pod/cert-manager-77844c9b4d-t9xgf              1/1     Running   0          2m16s
    pod/cert-manager-cainjector-78bbcdc47c-klj9h   1/1     Running   0          2m16s
    pod/cert-manager-webhook-79d48667bd-nx4ng      1/1     Running   0          2m16s

    NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
    service/cert-manager-webhook   ClusterIP   10.103.151.198   <none>        443/TCP   2m16s

    NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/cert-manager              1/1     1            1           2m16s
    deployment.apps/cert-manager-cainjector   1/1     1            1           2m16s
    deployment.apps/cert-manager-webhook      1/1     1            1           2m16s

    NAME                                                 DESIRED   CURRENT   READY   AGE
    replicaset.apps/cert-manager-77844c9b4d              1         1         1       2m16s
    replicaset.apps/cert-manager-cainjector-78bbcdc47c   1         1         1       2m16s
    replicaset.apps/cert-manager-webhook-79d48667bd      1         1         1       2m16s

<br/>

    $ kubectl get crds
    NAME                                CREATED AT
    certificates.certmanager.k8s.io     2019-05-13T11:38:53Z
    challenges.certmanager.k8s.io       2019-05-13T11:38:53Z
    clusterissuers.certmanager.k8s.io   2019-05-13T11:38:53Z
    issuers.certmanager.k8s.io          2019-05-13T11:38:53Z
    orders.certmanager.k8s.io           2019-05-13T11:38:53Z

<br/>

### Cluster Issuer

https://docs.cert-manager.io/en/latest/tasks/issuers/setup-acme.html#creating-a-basic-acme-issuer

Индус уже подготовил за нас конфиг.

<br/>

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/e6840743ac311347e4b5cabaceb0e6083f009799/yamls/cert-manager-demo/ClusterIssuer.yaml

    Указать реальный email (Иначе не будет работать).

    $ kubectl create -f ClusterIssuer.yaml

<br/>

### Deploy & Service

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/e6840743ac311347e4b5cabaceb0e6083f009799/yamls/cert-manager-demo/nginx-deployment.yaml

    $ kubectl get all
    NAME                         READY   STATUS    RESTARTS   AGE
    pod/nginx-65f88748fd-gnm5x   1/1     Running   0          27s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   70m

    NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx   1/1     1            1           27s

    NAME                               DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-65f88748fd   1         1         1       27s

<br/>

    $ kubectl expose deploy nginx --port 80

<br/>

### Ingress Resource

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/e6840743ac311347e4b5cabaceb0e6083f009799/yamls/cert-manager-demo/ingress-resource.yaml

<br/>

    $ kubectl get ing
    NAME                        HOSTS               ADDRESS   PORTS     AGE
    cm-acme-http-solver-wctxd   nginx.example.com             80        16s
    ingress-resource            nginx.example.com             80, 443   20s

<br/>

    $ kubectl describe ing ingress-resource
    Name:             ingress-resource
    Namespace:        default
    Address:
    Default backend:  default-http-backend:80 (<none>)
    TLS:
      letsencrypt-staging terminates nginx.example.com
    Rules:
      Host               Path  Backends
      ----               ----  --------
      nginx.example.com
                            nginx:80 (10.244.1.5:80)
    Annotations:
      certmanager.k8s.io/cluster-issuer:  letsencrypt-staging
    Events:
      Type    Reason             Age   From                      Message
      ----    ------             ----  ----                      -------
      Normal  CreateCertificate  83s   cert-manager              Successfully created Certificate "letsencrypt-staging"
      Normal  AddedOrUpdated     82s   nginx-ingress-controller  Configuration for default/ingress-resource was added or updated
      Normal  AddedOrUpdated     82s   nginx-ingress-controller  Configuration for default/ingress-resource was added or updated
      Normal  Updated            82s   nginx-ingress-controller  Configuration was updated due to updated secret default/letsencrypt-staging
      Normal  Updated            82s   nginx-ingress-controller  Configuration was updated due to updated secret default/letsencrypt-staging

<br/>

    $ kubectl get certificates
    NAME                  READY   SECRET                AGE
    letsencrypt-staging   False   letsencrypt-staging   2m25s

<br/>

    $ kubectl describe certificates letsencrypt-staging
    Name:         letsencrypt-staging
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>
    API Version:  certmanager.k8s.io/v1alpha1
    Kind:         Certificate
    Metadata:
      Creation Timestamp:  2019-05-13T12:00:04Z
      Generation:          3
      Owner References:
        API Version:           extensions/v1beta1
        Block Owner Deletion:  true
        Controller:            true
        Kind:                  Ingress
        Name:                  ingress-resource
        UID:                   a5286269-7576-11e9-a059-525400261060
      Resource Version:        7793
      Self Link:               /apis/certmanager.k8s.io/v1alpha1/namespaces/default/certificates/letsencrypt-staging
      UID:                     a52a7ca9-7576-11e9-a059-525400261060
    Spec:
      Acme:
        Config:
          Domains:
            nginx.example.com
          Http 01:
      Dns Names:
        nginx.example.com
      Issuer Ref:
        Kind:       ClusterIssuer
        Name:       letsencrypt-staging
      Secret Name:  letsencrypt-staging
    Status:
      Conditions:
        Last Transition Time:  2019-05-13T12:00:04Z
        Message:               Certificate issuance in progress. Temporary certificate issued.
        Reason:                TemporaryCertificate
        Status:                False
        Type:                  Ready
    Events:
      Type    Reason              Age   From          Message
      ----    ------              ----  ----          -------
      Normal  Generated           3m2s  cert-manager  Generated new private key
      Normal  GenerateSelfSigned  3m2s  cert-manager  Generated temporary self signed certificate
      Normal  OrderCreated        3m2s  cert-manager  Created Order resource "letsencrypt-staging-230902236"

<br/>

### DNS Update

На хост машине

    $ sudo vi /etc/hosts

    192.168.0.5 nginx.example.com

<br/>

### Testing

https://nginx.example.com

Все ОК.
