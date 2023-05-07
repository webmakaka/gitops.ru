---
layout: page
title: Запуск базы postgresql в minikube
description: Запуск базы postgresql в minikube
keywords: devops, linux, kubernetes,  Запуск базы postgresql в minikube
permalink: /devops/containers/kubernetes/minikube/postgresql/
---

# Запуск базы postgresql в minikube

Делаю:  
21.04.2020

kubernetes v1.16.1

https://github.com/burrsutter/9stepsawesome/blob/master/9_databases.adoc

<br/>

**postgres-pv.yml**

```
$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
  labels:
    type: local
spec:
  storageClassName: mystorage
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 2Gi
  hostPath:
    path: "/data/mypostgresdata/"
EOF
```

<br/>

**postgres-pvc.yml**

```
$ cat <<EOF | kubectl create -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pvc
  labels:
   app: postgres
spec:
  storageClassName: mystorage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF
```

<br/>

```
$ kubectl get pv/postgres-pv
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
postgres-pv   2Gi        RWO            Retain           Bound    default/postgres-pvc   mystorage               4m8s
```

<br/>

```
$ kubectl get pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pvc   Bound    postgres-pv   2Gi        RWO            mystorage      73s
```

<br/>

**postgres-deployment.yml**

```
$ cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:10.5
        imagePullPolicy: "IfNotPresent"
        env:
        - name: POSTGRES_DB
          value: postgresdb
        - name: POSTGRES_USER
          value: admin
        - name: POSTGRES_PASSWORD
          value: adminS3cret
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
          # mountPath within the container
        - name: postgres-pvc
          mountPath: "/var/lib/postgresql/data/:Z"
      volumes:
          # mapped to the PVC
        - name: postgres-pvc
          persistentVolumeClaim:
            claimName: postgres-pvc
EOF
```

<br/>

```
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
postgres-6df5c4bc5d-vmj2z   1/1     Running   0          73s
```

<br/>

```
$ kubectl port-forward <posgtres-pod> 5432:5432

```

<br/>

    $ telnet localhost 5432
    Trying 127.0.0.1...

<br/>

    // если нужен клиент psql
    $ sudo apt-get install -y postgresql-client

<br/>

    // password - adminS3cret
    $ psql -h localhost -p 5432 -U admin -W postgresdb

OK

<!--

<br/>

**Запускаю pgAdmin**

```
$ docker run -e PGADMIN_DEFAULT_EMAIL='username' -e PGADMIN_DEFAULT_PASSWORD='password' -p 5555:80 --name pgadmin dpage/pgadmin4
```


<br/>

Запускаю в виртуалке minikube

    $ minikube --profile my-profile ssh


<br/>

```
$ docker run -e PGADMIN_DEFAULT_EMAIL='username' -e PGADMIN_DEFAULT_PASSWORD='password' -p 5555:80 --name pgadmin dpage/pgadmin4
```

<br/>

    $ minikube --profile my-profile ip
    192.168.99.115



<br/>

http://192.168.99.115:5555/


-->
