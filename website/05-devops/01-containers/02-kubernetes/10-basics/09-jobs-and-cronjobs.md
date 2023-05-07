---
layout: page
title: Jobs & Cronjobs in Kubernetes
description: Jobs & Cronjobs in Kubernetes
keywords: devops, linux, kubernetes, Jobs & Cronjobs in Kubernetes
permalink: /devops/containers/kubernetes/basics/jobs-and-cronjobs/
---

# Jobs & Cronjobs in Kubernetes

<br/>

Делаю: 07.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=uJKE0d6Y_yg&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=12

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

### Job

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/2-job.yaml

    $ kubectl get jobs
    NAME         COMPLETIONS   DURATION   AGE
    helloworld   1/1           6s         12s

    $ kubectl get pods
    NAME               READY   STATUS      RESTARTS   AGE
    helloworld-bc8bn   0/1     Completed   0          42s

    $ kubectl logs helloworld-bc8bn
    Hello Kubernetes!!!

    $ kubectl delete job helloworld

<br/>

### Cronjob

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/2-cronjob.yaml

    $ kubectl get cronjob
    NAME              SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
    helloworld-cron   * * * * *   False     0        67s             5m58s

    $ kubectl delete cronjob helloworld-cron
