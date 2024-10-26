---
layout: page
title: Запуск Prometheus (мониторинг) и Grafana (визуализация) в kuberntes cluster с помощью heml
description: Запуск Prometheus (мониторинг) и Grafana (визуализация) в kuberntes cluster с помощью heml
keywords: devops, linux, kubernetes, Запуск Prometheus (мониторинг) и Grafana (визуализация) в kuberntes cluster с помощью heml
permalink: /devops/containers/kubernetes/monitoring/prometheus-and-grafana/
---

# Запуск Prometheus (мониторинг) и Grafana (визуализация) в kuberntes cluster с помощью heml

<br/>

Делаю:
11.04.2019

<br/>

По материалам из видео индуса:

https://www.youtube.com/watch?v=CmPdyvgmw-A&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=27

<br/>

![prometheus and grafana 01](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-01.png 'prometheus and grafana 01'){: .center-image }

<br/>

- Подготовили кластер и окружение
- Подняли Dynamic NFS
- Инсталлировали helm

<br/>

UPD. Heml2 выпилен (как ненужное), предлагаю попробовать Helm3 как <a href="/devops/containers/kubernetes/packages/heml/setup/">здесь</a>.

<br/>

### Устанавливаем prometheus

    // если нужно посмотреть параметры по умолчанию
    $ helm inspect values stable/prometheus > /tmp/prometheus.values

<br/>

    // я просто создаю файл, чтобы не копаться в куче кода
    $ vi /tmp/prometheus.values.final

```
 server:
    service:
      nodePort: 32323
      type: NodePort
```

<br/>

    $ helm install stable/prometheus --name myprometheus --values /tmp/prometheus.values.final --namespace prometheus

<br/>

    $ kubectl get all -n prometheus
    NAME                                                   READY   STATUS    RESTARTS   AGE
    pod/myprometheus-alertmanager-55f5594766-kjzcp         2/2     Running   0          4m19s
    pod/myprometheus-kube-state-metrics-668df79bd8-dgqpm   1/1     Running   0          4m19s
    pod/myprometheus-node-exporter-5vx6c                   1/1     Running   0          4m19s
    pod/myprometheus-node-exporter-v57qb                   1/1     Running   0          4m19s
    pod/myprometheus-pushgateway-56fccfb787-mkssw          1/1     Running   0          4m19s
    pod/myprometheus-server-6b48bd7c95-74dfj               2/2     Running   0          4m19s

    NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    service/myprometheus-alertmanager         ClusterIP   10.110.83.127    <none>        80/TCP         4m19s
    service/myprometheus-kube-state-metrics   ClusterIP   None             <none>        80/TCP         4m19s
    service/myprometheus-node-exporter        ClusterIP   None             <none>        9100/TCP       4m19s
    service/myprometheus-pushgateway          ClusterIP   10.108.225.88    <none>        9091/TCP       4m19s
    service/myprometheus-server               NodePort    10.102.139.197   <none>        80:32323/TCP   4m19s

    NAME                                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    daemonset.apps/myprometheus-node-exporter   2         2         2       2            2           <none>          4m19s

    NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/myprometheus-alertmanager         1/1     1            1           4m19s
    deployment.apps/myprometheus-kube-state-metrics   1/1     1            1           4m19s
    deployment.apps/myprometheus-pushgateway          1/1     1            1           4m19s
    deployment.apps/myprometheus-server               1/1     1            1           4m19s

    NAME                                                         DESIRED   CURRENT   READY   AGE
    replicaset.apps/myprometheus-alertmanager-55f5594766         1         1         1       4m19s
    replicaset.apps/myprometheus-kube-state-metrics-668df79bd8   1         1         1       4m19s
    replicaset.apps/myprometheus-pushgateway-56fccfb787          1         1         1       4m19s
    replicaset.apps/myprometheus-server-6b48bd7c95               1         1         1       4m19s

<br/>

    $ kubectl get pvc -n prometheus
    NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
    myprometheus-alertmanager   Bound    pvc-c34dbaed-5c72-11e9-bbe3-525400261060   2Gi        RWO            managed-nfs-storage   4m53s
    myprometheus-server         Bound    pvc-c34f1d62-5c72-11e9-bbe3-525400261060   8Gi        RWO            managed-nfs-storage   4m53s

<br/>

    $ kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS          REASON   AGE
    pvc-c34dbaed-5c72-11e9-bbe3-525400261060   2Gi        RWO            Delete           Bound    prometheus/myprometheus-alertmanager   managed-nfs-storage            4m58s
    pvc-c34f1d62-5c72-11e9-bbe3-525400261060   8Gi        RWO            Delete           Bound    prometheus/myprometheus-server         managed-nfs-storage            4m58s

<br/>

http://node1:32323

<br/>

![prometheus and grafana 02](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-02.png 'prometheus and grafana 02'){: .center-image }

<br/>

### Устанавливаем Graphana

    // если нужно посмотреть параметры по умолчанию
    $ helm inspect values stable/grafana > /tmp/grafana.values

<br/>

    // я просто создаю файл, чтобы не копаться в куче кода
    $ vi /tmp/grafana.values.final

<br/>

```
service:
  nodePort: 32324
  type: NodePort

adminUser: admin
adminPassword: admin

persistence:
  enabled: true
```

<br/>

    $ helm install stable/grafana --name mygrafana --values /tmp/grafana.values.final --namespace grafana

<br/>

    $ kubectl get all -n grafana
    NAME                            READY   STATUS            RESTARTS   AGE
    pod/mygrafana-d47947d89-cdmfx   0/1     PodInitializing   0          15s

    NAME                TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
    service/mygrafana   NodePort   10.99.20.18   <none>        80:32324/TCP   15s

    NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/mygrafana   0/1     1            0           15s

    NAME                                  DESIRED   CURRENT   READY   AGE
    replicaset.apps/mygrafana-d47947d89   1         1         0       15s

<br/>

    $  kubectl get pvc -n grafana
    NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
    mygrafana   Bound    pvc-6cdf745f-5c74-11e9-bbe3-525400261060   10Gi       RWO            managed-nfs-storage   79s

<br/>

http://node1:32324

<br/>

![prometheus and grafana 03](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-03.png 'prometheus and grafana 03'){: .center-image }

<br/>

![prometheus and grafana 04](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-04.png 'prometheus and grafana 04'){: .center-image }

<br/>

![prometheus and grafana 05](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-05.png 'prometheus and grafana 05'){: .center-image }

<br/>

![prometheus and grafana 06](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-06.png 'prometheus and grafana 06'){: .center-image }

<br/>

![prometheus and grafana 07](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-07.png 'prometheus and grafana 07'){: .center-image }

<br/>

Подбираем под себя:  
https://grafana.com/dashboards/8588

<br/>

![prometheus and grafana 08](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-08.png 'prometheus and grafana 08'){: .center-image }

<br/>

![prometheus and grafana 09](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-09.png 'prometheus and grafana 09'){: .center-image }

<br/>

![prometheus and grafana 10](/img/devops/containers/kubernetes/kubeadm/helm/prometheus-and-grafana/prometheus-and-grafana-10.png 'prometheus and grafana 10'){: .center-image }

<br/>

### Удаление из kubernetes

    $ helm list
    $ helm delete myprometheus --purge
    $ helm delete mygrafana --purge
