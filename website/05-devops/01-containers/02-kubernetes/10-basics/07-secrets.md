---
layout: page
title: Secrets in Kubernetes
description: Secrets in Kubernetess
keywords: devops, linux, kubernetes, Secrets in Kubernetess
permalink: /devops/containers/kubernetes/basics/secrets/
---

# Secrets in Kubernetes

<br/>

Делаю: 08.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=ch9YlQZ4xTc&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=15

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

### Secrets

<br/>

### Создать в командной строке

    $ kubectl create secret generic secret-demo --from-literal=username='kubeadmin' --from-literal=password='mypassword'

    $ kubectl get secrets
    NAME                  TYPE                                  DATA   AGE
    default-token-sljtn   kubernetes.io/service-account-token   3      26m
    secret-demo           Opaque                                2      12s

    $ kubectl describe secret secret-demo

    $ kubectl get secrets -o yaml

    $ echo 'bXlwYXNzd29yZA==' | base64 --decode
    mypassword

<br/>

### Создать из файлов

    $ vi username
    username

    $ vi password
    password

    $ kubectl create secret generic secret-demo --from-file=username=./username --from-file=password

<br/>

### Закодировать переменные

    $ echo -n 'kubeadmin' | base64
    a3ViZWFkbWlu

    $ echo -n 'mypassword' | base64
    bXlwYXNzd29yZA==

<br/>

    $ rm -rf ~/tmp/k8s/secrets && mkdir -p ~/tmp/k8s/secrets && cd ~/tmp/k8s/secrets

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/5-secrets.yaml

<br/>

    $ vi 5-secrets.yaml

```

apiVersion: v1
kind: Secret
metadata:
  name: secret-demo
type: Opaque
data:
  username: a3ViZWFkbWlu
  password: bXlwYXNzd29yZA==

```

<br/>

    $ kubectl get secrets
    NAME                  TYPE                                  DATA   AGE
    default-token-xhlgl   kubernetes.io/service-account-token   3      63m

<br/>

    $ kubectl create -f 5-secrets.yaml

<br/>

    $ kubectl get secrets
    NAME                  TYPE                                  DATA   AGE
    default-token-xhlgl   kubernetes.io/service-account-token   3      64m
    secret-demo           Opaque                                2      12s

<br/>

    $ kubectl describe secret secret-demo
    Name:         secret-demo
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Type:  Opaque

    Data
    ====
    password:  10 bytes
    username:  9 bytes

<br/>

    $ kubectl delete secret secret-demo

<br/>

### Как пользоваться

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/5-pod-secret-env.yaml

    $ kubectl get pods
    NAME      READY   STATUS    RESTARTS   AGE
    busybox   1/1     Running   0          11s

    $ kubectl exec -it busybox -- sh

    / # env | grep myusername
    myusername=kubeadmin

    / # echo $myusername
    kubeadmin

    ctrl^D

    $ kubectl delete pod busybox

<br/>

### Еще пример

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/5-pod-secret-volume.yaml

    $ kubectl exec -it busybox -- sh

    / # env | grep myusername
    (ничего)

    / # ls /mydata
    password  username

    # cat /mydata/username; echo
    kubeadmin

    # cat /mydata/password; echo
    mypassword

    ctrl^D

<br/>

### Пример с обновлением пароля

    Обновляем 5-secrets.yaml

    $ kubectl apply -f 5-secrets.yaml

<br/>

### Удаляем

    $ kubectl delete pod busybox
