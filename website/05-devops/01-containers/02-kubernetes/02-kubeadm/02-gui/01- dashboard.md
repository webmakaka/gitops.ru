---
layout: page
title: Install Kubernetes Dashboard Web UI
description: Install Kubernetes Dashboard Web UI
keywords: devops, linux, kubernetes, Install Kubernetes Dashboard Web UI
permalink: /devops/containers/kubernetes/kubeadm/gui/dashboard/
---

# Install Kubernetes Dashboard Web UI

<br/>

Делаю: 11.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=brqAMyayjrI&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=6

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>

<br/>

### Поехали

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/004fd3d9c66f2eb453324d24e94eaabc65491895/dashboard/influxdb.yaml


    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/004fd3d9c66f2eb453324d24e94eaabc65491895/dashboard/heapster.yaml


    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/004fd3d9c66f2eb453324d24e94eaabc65491895/dashboard/dashboard.yaml

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/004fd3d9c66f2eb453324d24e94eaabc65491895/dashboard/sa_cluster_admin.yaml

<br/>

    $ kubectl describe sa dashboard-admin -n kube-system
    Name:                dashboard-admin
    Namespace:           kube-system
    Labels:              <none>
    Annotations:         <none>
    Image pull secrets:  <none>
    Mountable secrets:   dashboard-admin-token-jq99k
    Tokens:              dashboard-admin-token-jq99k
    Events:              <none>

<br/>

    $ kubectl get secret dashboard-admin-token-jq99k -n kube-system
    NAME                          TYPE                                  DATA   AGE
    dashboard-admin-token-jq99k   kubernetes.io/service-account-token   3      87s

<br/>

    $ kubectl describe secret dashboard-admin-token-jq99k -n kube-system
    Name:         dashboard-admin-token-jq99k
    Namespace:    kube-system
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name: dashboard-admin
                kubernetes.io/service-account.uid: 7d4f6c01-5c9c-11e9-b61d-525400261060

    Type:  kubernetes.io/service-account-token

    Data
    ====
    namespace:  11 bytes
    token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tanE5OWsiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiN2Q0ZjZjMDEtNWM5Yy0xMWU5LWI2MWQtNTI1NDAwMjYxMDYwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.aOt56z2ffZWCcV24LTe_m4tAoSOIPZQHPTDpVINYblSvmBYpInZ5OVxZK0yQDK1N_r7kZgraPxUSGHxJ-nv1edqNyrR9UEPpx9iozPJZgPfcWwIh1ExEiP8BICQ6dmJLNfia0F4UZp60U3ve8gFGwGEG3hQhxMgw7gf6aa8n95bRASvfs4zyjmxnXng2dKrVh-LjqgzbmyIxbYOFofylUSUyB7aIO0fJ0BKMYToLoY8FyGi3Y3GiOleplzHdZZdN_VPIQTx0GbcvD_oeiMx8e5qYadwnFIV8qXG7S4znJVlyaxEeXK_1q9Y0I7IBLTSwD0WZRNaRCgZv77owbN8Hkg
    ca.crt:     1025 bytes

<br/>

https://node1:32323

<br/>

![kubernetes Dashboard 01](/img/devops/containers/kubernetes/kubeadm/gui/dashboard/dashboard-01.png 'kubernetes Dashboard 01'){: .center-image }

<br/>

![kubernetes Dashboard 02](/img/devops/containers/kubernetes/kubeadm/gui/dashboard/dashboard-02.png 'kubernetes Dashboard 02'){: .center-image }

<br/>

![kubernetes Dashboard 03](/img/devops/containers/kubernetes/kubeadm/gui/dashboard/dashboard-03.png 'kubernetes Dashboard 03'){: .center-image }

<br/>

![kubernetes Dashboard 04](/img/devops/containers/kubernetes/kubeadm/gui/dashboard/dashboard-04.png 'kubernetes Dashboard 04'){: .center-image }
