---
layout: page
title: FluxCD 101 with Hands-On Labs
description: FluxCD 101 with Hands-On Labs
keywords: linux, kubernetes, FluxCD
permalink: /courses/containers/kubernetes/ci-cd/fluxcd/fluxcd-101-with-hands-on-labs/
---

# [Video Course][Siddharth Barahalikar] FluxCD 101 with Hands-On Labs [ENG, 2023][~5h 45m]

<br/>

**Rep'ы автора:**

```
https://github.com/sid-demo?tab=repositories
https://github.com/sidd-harth/block-buster
https://github.com/sidd-harth-2
```

<br/>

## 02. Flux Overview

<br/>

### LAB 1 - Setup FluxCD Server and CLI

[Устанавливаю FluxCD](/tools/containers/kubernetes/tools/ci-cd/fluxcd/setup/)

```
$ flux --version
flux version 2.0.0-rc.1
```

<br/>

```
$ export GITHUB_USER=wildmakaka
$ export REPOSITORY_NAME=block-buster
```

<br/>

```
$ flux bootstrap github \
  --owner=${GITHUB_USER} \
  --repository=${REPOSITORY_NAME} \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```

<br/>

```
$ git clone git@github.com:wildmakaka/block-buster.git
```

<br/>

```
$ LATEST_KUBERNETES_VERSION=v1.27.1
```

<br/>

```
$ export \
    PROFILE=${USER}-minikube \
    CPUS=4 \
    MEMORY=8G \
    HDD=20G \
    DRIVER=docker \
    KUBERNETES_VERSION=${LATEST_KUBERNETES_VERSION}
```

[Поднимаю Minikube](/tools/containers/kubernetes/minikube/setup/)

<br/>

## 03. Source and Kustomize Controller

<br/>

### LAB 2 - Deploy Application Manifest - Flux Repo

<br/>

Fork -> https://github.com/sidd-harth/bb-app-source

<br/>

```
$ cd ~/projects/dev/fluxcd
$ git clone git@github.com:wildmakaka/bb-app-source.git
$ cd bb-app-source/
$ git switch 1-demo
$ git pull
```

<br/>

```
$ flux get sources git
NAME       	REVISION          	SUSPENDED	READY	MESSAGE
flux-system	main@sha1:9f2d417a	False    	True 	stored artifact for revision 'main@sha1:9f2d417a'
```

<br/>

```
$ flux get kustomization
NAME       	REVISION          	SUSPENDED	READY	MESSAGE
flux-system	main@sha1:9f2d417a	False    	True 	Applied revision: main@sha1:9f2d417a
```

<br/>

Скопировали манифесты из bb-app-source/manifests в каталог clusters/my-cluster/1-demo

commit / push

<br/>

```
// Должно появиться 1-demo
$ kubectl get ns
NAME                        STATUS   AGE
1-demo                      Active   28s
***
```

<br/>

```
$ kubectl -n 1-demo get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/block-buster-7c7c5bd4d8-2ggrb   1/1     Running   0          75s

NAME                           TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/block-buster-service   NodePort   10.108.171.222   <none>        80:30001/TCP   75s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/block-buster   1/1     1            1           75s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/block-buster-7c7c5bd4d8   1         1         1       75s
```

<br/>

```
$ export PROFILE=${USER}-minikube
$ minikube --profile ${PROFILE} ip
```

<br/>

```
// OK!
http://192.168.49.2:30001
```

<br/>

```
// Посмотреть где в итоге хранятся манифесты
$ kubectl -n flux-system get pods
$ kubectl -n flux-system exec -it source-controller-664f9c8869-c6nj8 -- sh
$ cd data/gitrepository/flux-system/flux-system/
$ ls
035ab64a849b04b2595d837e6a80d0eb4eb22001.tar.gz
035ab64a849b04b2595d837e6a80d0eb4eb22001.tar.gz.lock

$ tar -tf 035ab64a849b04b2595d837e6a80d0eb4eb22001.tar.gz
.
clusters
clusters/my-cluster
clusters/my-cluster/1-demo
clusters/my-cluster/1-demo/deployment.yml
clusters/my-cluster/1-demo/namespace.yml
clusters/my-cluster/1-demo/service.yml
clusters/my-cluster/flux-system
clusters/my-cluster/flux-system/gotk-components.yaml
clusters/my-cluster/flux-system/gotk-sync.yaml
clusters/my-cluster/flux-system/kustomization.yaml
```

<br/>

### LAB 3 - Deploy Application Manifest - External Git Repo

<br/>

```
$ git switch 2-demo
```

<br/>

```
// Посмотреть, но не создавать
$ flux create source git 2-demo-source-git-bb-app \
  --url https://github.com/wildmakaka/bb-app-source \
  --branch=2-demo \
  --timeout 10s \
  --export
```

<br/>

```
// Создать
$ flux create source git 2-demo-source-git-bb-app \
  --url https://github.com/wildmakaka/bb-app-source \
  --branch=2-demo \
  --timeout 10s
```

<br/>

```
$ flux get sources git
NAME                    	REVISION            	SUSPENDED	READY	MESSAGE
2-demo-source-git-bb-app	2-demo@sha1:310e0bea	False    	True 	stored artifact for revision '2-demo@sha1:310e0bea'
flux-system             	main@sha1:035ab64a  	False    	True 	stored artifact for revision 'main@sha1:035ab64a'
```

<br/>

```
$ flux delete source git 2-demo-source-git-bb-app
```

<br/>

```
$ cd ~/projects/fluxcd/block-buster/clusters/my-cluster/
```

<br/>

```
// Создать yaml
$ flux create source git 2-demo-source-git-bb-app \
  --url https://github.com/wildmakaka/bb-app-source \
  --branch=2-demo \
  --timeout 10s \
  --export > 2-demo-source-git-bb-app.yaml
```

<br/>

```
// Создать yaml
$ flux create kustomization 2-demo-source-git-bb-app \
  --source GitRepository/2-demo-source-git-bb-app \
  --prune true \
  --interval 10s \
  --target-namespace 2-demo \
  --path manifests \
  --export > 2-demo-kustomize-git-bb-app.yaml
```

<br/>

```
// Добавились новые REVISION
$ flux get sources git
$ flux get kustomizations
```

<br/>

```
$ kubectl get ns
NAME                        STATUS   AGE
1-demo                      Active   49m
2-demo                      Active   44s
***
```

<br/>

```
// OK!
http://192.168.49.2:30002
```

<br/>

```
$ kubectl -n 2-demo get all
```

<br/>

```
// Пришлось останавливать ранее запущенное приложение, т.к. мало cpu.
// OK!
http://192.168.49.2:30002
```

<br/>

### Продолжение

<br/>

```
$ git switch 3-demo
```

<br/>

```
$ cd ~/projects/fluxcd/block-buster/clusters/my-cluster/
```

<br/>

```
$ flux create source git 3-demo-source-git-bb-app \
  --url https://github.com/wildmakaka/bb-app-source \
  --branch=3-demo \
  --timeout 10s \
  --export > 3-demo-source-git-bb-app.yaml
```

<br/>

```
$ flux create kustomization 3-demo-source-git-bb-app \
  --source GitRepository/3-demo-source-git-bb-app \
  --prune true \
  --interval 10s \
  --target-namespace 3-demo \
  --path kustomize \
  --export > 3-demo-kustomize-git-bb-app.yaml
```

<br/>

```
$ flux get sources git
$ flux get sources git
$ kubectl get ns
```

<br/>

```
$ kubectl -n 3-demo get all
```

<br/>

```
// Пришлось останавливать ранее запущенное приложение, т.к. мало cpu.
// OK!
http://192.168.49.2:30003
```

<br/>

### LAB 4 - Deploy Application Manifest from a S3 Repo

<br/>

```
$ git switch 4-demo
```

<br/>

```
$ kubectl apply -f minio/minio-s3.yaml
```

<br/>

```
$ kubectl -n minio-dev get all
```

<br/>

```
// OK!
// minioadmin / minioadmin
http://192.168.49.2:30040/
```

<br/>

```
Buckets -> create bucket
name: bucket-bb-app
```

<br/>

```
Object Browser -> Create new path
name: app-740
```

<br/>

```
Upload: manifests
```

<br/>

```
$ flux create source bucket 4-demo-source-minio-s3-bucket-bb-app \
  --bucket-name bucket-bb-app \
  --secret-ref minio-crds \
  --endpoint minio.minio-dev.svc.cluster.local:9000 \
  --provider generic \
  --insecure \
  --export > 4-demo-source-minio-s3-bucket-bb-app.yaml
```

<br/>

```
$ flux create kustomization 4-demo-kustomize-minio-s3-bucket-bb-app \
  --source Bucket/4-demo-source-minio-s3-bucket-bb-app \
  --target-namespace 4-demo \
  --path ./app-740 \
  --prune true \
  --export > 4-demo-kustomize-minio-s3-bucket-bb-app.yaml
```

<br/>

```
$ kubectl -n flux-system create secret generic minio-crds \
    --from-literal=accesskey=minioadmin \
    --from-literal=secretkey=minioadmin
```

<br/>

commit / push

<br/>

```
$ flux get source bucket
NAME                                	REVISION       	SUSPENDED	READY	MESSAGE
4-demo-source-minio-s3-bucket-bb-app	sha256:cce16a8a	False    	True 	stored artifact: revision 'sha256:cce16a8a'
```

<br/>

```
// При необходимости
$ flux reconcile source bucket 4-demo-source-minio-s3-bucket-bb-app
```

<br/>

```
$ kubectl -n 4-demo get all
```

<br/>

```
// Пришлось останавливать ранее запущенное приложение, т.к. мало cpu.
// OK!
http://192.168.49.2:30004
```

<br/>

## 04. Helm Controller and OCI Registry

<br/>

### LAB 5 - Deploy Helm Charts from a Helm Repository

<br/>

**Чистка от предыдущих лаб**

```
$ flux get sources all
```

```
$ flux get sources bucket
$ flux delete source bucket 4-demo-source-minio-s3-bucket-bb-app
```

```
$ flux get sources git
$ flux delete source git 2-demo-source-git-bb-app
$ flux delete source git 3-demo-source-git-bb-app
```

<br/>

```
$ flux get kustomization
$ flux delete kustomization 2-demo-source-git-bb-app
$ flux delete kustomization 3-demo-source-git-bb-app
$ flux delete kustomization 4-demo-kustomize-minio-s3-bucket-bb-app
```

<br/>

```
$ git switch 5-demo
```

<br/>

```
$ flux create source git 5-demo-source-git-helm-bb-app \
  --url https://github.com/wildmakaka/bb-app-source \
  --branch=5-demo \
  --timeout 10s \
  --export > 5-demo-source-git-helm-bb-app.yaml
```

<br/>

```
$ vi 5-demo-values.yaml
```

```yaml
replicaCount: 2

service:
  type: NodePort
  nodePort: 30005

namespace:
  name: 5-demo

labels:
  app:
    name: block-buster
    version: 7.5.0
    env: dev
```

<br/>

```
$ flux create helmrelease 5-demo-helm-release-git-helm-bb-app \
  --chart helm-chart \
  --interval 10s \
  --target-namespace 5-demo \
  --source GitRepository/5-demo-source-git-helm-bb-app \
  --values 5-demo-values.yaml \
  --export > 5-demo-helm-release-git-helm-bb-app.yaml
```

<br/>

```
// Пока не удалил файл 5-demo-values.yaml, ничего не запускалось:
$ rm 5-demo-values.yaml
```

<br/>

commit / push

<br/>

```
$ flux get sources git 5-demo-source-git-helm-bb-app
NAME                         	REVISION            	SUSPENDED	READY	MESSAGE
5-demo-source-git-helm-bb-app	5-demo@sha1:d327af27	False    	True 	stored artifact for revision '5-demo@sha1:d327af27'
```

<br/>

```
$ flux get helmreleases
NAME                               	REVISION	SUSPENDED	READY	MESSAGE
5-demo-helm-release-git-helm-bb-app	7.5.0   	False    	True 	Release reconciliation succeeded
```

<br/>

```
// OK!
http://192.168.49.2:30005/
```

<br/>

```
// labels подтянулись из файла 5-demo-values.yaml
$ kubectl -n 5-demo get pods --show-labels
NAME                                     READY   STATUS    RESTARTS   AGE    LABELS
block-buster-helm-app-6d69ff466d-bjq5x   1/1     Running   0          7m4s   app=block-buster,env=dev,pod-template-hash=6d69ff466d,version=7.5.0
block-buster-helm-app-6d69ff466d-vfhn2   1/1     Running   0          7m4s   app=block-buster,env=dev,pod-template-hash=6d69ff466d,version=7.5.0
```

<br/>

```
$ flux get sources chart
NAME                                           	REVISION	SUSPENDED	READY	MESSAGE
flux-system-5-demo-helm-release-git-helm-bb-app	7.5.0   	False    	True 	packaged 'block-buster-helm-app' chart with version '7.5.0'
```

<br/>

```
$ kubectl -n flux-system get helmcharts.source.toolkit.fluxcd.io
NAME                                              CHART        VERSION   SOURCE KIND     SOURCE NAME                     AGE   READY   STATUS
flux-system-5-demo-helm-release-git-helm-bb-app   helm-chart   *         GitRepository   5-demo-source-git-helm-bb-app   12m   True    packaged 'block-buster-helm-app' chart with version '7.5.0'
```

<br/>

```
$ kubectl -n flux-system get helmcharts.source.toolkit.fluxcd.io -o yaml
```

<br/>

### Продолжение

<br/>

https://sidd-harth.github.io/block-buster-helm-app/

https://artifacthub.io/packages/helm/block-buster-app/block-buster-helm-app

<br/>

```
$ flux create source helm 6-demo-source-helm-bb-app \
  --url https://sidd-harth.github.io/block-buster-helm-app \
  --timeout 10s \
  --export > 6-demo-source-helm-bb-app.yaml
```

<br/>

```
$ vi 6-demo-values.yaml
```

```yaml
replicaCount: 1

service:
  type: NodePort
  nodePort: 30006

namespace:
  name: 6-demo

labels:
  app:
    name: block-buster
    version: 7.6.0
    env: dev
```

<br/>

```
$ flux create helmrelease 6-demo-helm-release-bb-app \
  --chart block-buster-helm-app \
  --interval 10s \
  --target-namespace 6-demo \
  --source HelmRepository/6-demo-source-helm-bb-app \
  --values 6-demo-values.yaml \
  --export > 6-demo-helm-release-bb-app.yaml
```

<br/>

```
$ rm 6-demo-values.yaml
```

<br/>

```
// OK!
http://192.168.49.2:30006/
```

<br/>

```
$ flux get helmreleases
NAME                      	REVISION	SUSPENDED	READY	MESSAGE
6-demo-helm-release-bb-app	7.6.0   	False    	True 	Release reconciliation succeeded
```

