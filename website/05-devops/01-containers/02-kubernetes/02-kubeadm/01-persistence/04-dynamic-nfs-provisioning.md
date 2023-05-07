---
layout: page
title: Dynamically NFS provisioning
description: Dynamically NFS provisioning
keywords: devops, linux, kubernetes, Dynamically NFS provisioning
permalink: /devops/containers/kubernetes/kubeadm/persistence/dynamic-nfs-provisioning/
---

# Dynamically NFS provisioning

Делаю:  
22.11.2019

<br/>

    $ kubectl version --short
    Client Version: v1.16.3
    Server Version: v1.16.3

<br/>

По материалам из видео индуса.

https://www.youtube.com/watch?v=AavnQzWDTEk&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=24

<br/>

![kubernetes NFS provisioning](/img/devops/containers/kubernetes/kubeadm/persistence/NFS-provisioning.png 'kubernetes NFS provisioning'){: .center-image }

<br/>

Подготовили кластер и окружение как <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-centos7">здесь</a>.

<br/>

Подготовили экспорт NFS как<a href="/devops/containers/kubernetes/kubeadm/persistence/nfs/">здесь</a>

<br/>

### Подготавливаем кластер для работы с NFS. (выполняем команды на хост машине)

    $ kubectl create -f https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/nfs-provisioner/rbac.yaml

<br/>

    $ kubectl get clusterrole,clusterrolebinding,role,rolebinding | grep nfs

<br/>

    $ rm -rf ~/tmp/k8s/dynamic-nfs-provisioning/ && mkdir -p ~/tmp/k8s/dynamic-nfs-provisioning/ && cd ~/tmp/k8s/dynamic-nfs-provisioning/

<br/>

<!--

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/faf2f86a2c1bb82053c5aba9ea7c96463e4e61b0/yamls/nfs-provisioner/class.yaml

<br/>

-->

```
$ cat <<EOF >> class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: example.com/nfs
parameters:
  archiveOnDelete: "false"
EOF
```

<br/>

    $ kubectl apply -f class.yaml

<br/>

    $ kubectl get storageclass
    NAME                            PROVISIONER       AGE
    managed-nfs-storage (default)   example.com/nfs   10s

<br/>

    $ curl -LJO https://bitbucket.org/sysadm-ru/kubernetes/raw/71509f958c946bf0173392801a7fba45941f5397/yamls/nfs-provisioner/deployment.yaml

<br/>

    $ vi deployment.yaml

    <<NFS Server IP>> меняю на 192.168.0.6 (в 2 местах)

<br/>

    $ kubectl apply -f deployment.yaml

<br/>

    $ kubectl get all
    NAME                                         READY   STATUS    RESTARTS   AGE
    pod/nfs-client-provisioner-b48654857-tmdrm   1/1     Running   0          2m34s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   46m

    NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nfs-client-provisioner   1/1     1            1           2m34s

    NAME                                               DESIRED   CURRENT   READY   AGE
    replicaset.apps/nfs-client-provisioner-b48654857   1         1         1       2m34s

<br/>

Подготовка закончена!

<br/>

## Изучаем всевозможные варианты использования

### Создаем PVC

    $ vi 4-pvc-nfs.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  storageClassName: managed-nfs-storage
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi

```

<br/>

    $ kubectl create -f 4-pvc-nfs.yaml

<br/>

    $ kubectl get pv,pvc
    NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS          REASON   AGE
    persistentvolume/pvc-38a1eff2-5684-11e9-9ec5-525400261060   500Mi      RWX            Delete           Bound    default/pvc1   managed-nfs-storage            21s

    NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
    persistentvolumeclaim/pvc1   Bound    pvc-38a1eff2-5684-11e9-9ec5-525400261060   500Mi      RWX            managed-nfs-storage   31s

<br/>

    [vagrant@nfs-serv ~]$ ls /srv/nfs/kubedata/
    default-pvc1-pvc-38a1eff2-5684-11e9-9ec5-525400261060

<br/>

### Пробуем запустить контейнер

    $ vi 4-busybox-pv-hostpath.yaml

```

apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  volumes:
  - name: host-volume
    persistentVolumeClaim:
      claimName: pvc1
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh"]
    args: ["-c", "sleep 600"]
    volumeMounts:
    - name: host-volume
      mountPath: /mydata

```

<br/>

    $ kubectl create -f 4-busybox-pv-hostpath.yaml

<br/>

    $ kubectl get pods
    NAME                                      READY   STATUS    RESTARTS   AGE
    busybox                                   1/1     Running   0          12s
    nfs-client-provisioner-67cd85d66d-sw4cm   1/1     Running   0          15m

<br/>

Насрем в контейнере, как всегда и уйдем!

    $ kubectl exec -it busybox -- sh

    $ touch /mydata/hello

<br/>

    [vagrant@nfs-serv ~]$ ls /srv/nfs/kubedata/default-pvc1-pvc-38a1eff2-5684-11e9-9ec5-525400261060/
    hello

<br/>

    $ vi 4-pvc-nfs.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc2
spec:
  storageClassName: managed-nfs-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi

```

    $ kubectl create -f 4-pvc-nfs.yaml

<br/>

    $ kubectl get pv,pvc
    NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS          REASON   AGE
    persistentvolume/pvc-38a1eff2-5684-11e9-9ec5-525400261060   500Mi      RWX            Delete           Bound    default/pvc1   managed-nfs-storage            10m
    persistentvolume/pvc-a399632e-5685-11e9-9ec5-525400261060   100Mi      RWO            Delete           Bound    default/pvc2   managed-nfs-storage            21s

    NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
    persistentvolumeclaim/pvc1   Bound    pvc-38a1eff2-5684-11e9-9ec5-525400261060   500Mi      RWX            managed-nfs-storage   10m
    persistentvolumeclaim/pvc2   Bound    pvc-a399632e-5685-11e9-9ec5-525400261060   100Mi      RWO            managed-nfs-storage   21s

<br/>

    [vagrant@nfs-serv ~]$ ls /srv/nfs/kubedata/
    default-pvc1-pvc-38a1eff2-5684-11e9-9ec5-525400261060
    default-pvc2-pvc-a399632e-5685-11e9-9ec5-525400261060

<br/>

### Удаляем, когда надоело играться

    $ kubectl delete pod busybox
    $ kubectl delete pvc --all
    $ kubectl delete deploy nfs-client-provisioner
