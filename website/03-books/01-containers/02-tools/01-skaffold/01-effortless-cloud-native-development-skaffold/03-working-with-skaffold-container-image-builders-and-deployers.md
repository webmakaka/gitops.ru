---
layout: page
title: Building CI/CD Systems Using Tekton - Working with Skaffold Container Image Builders and Deployers
description: Building CI/CD Systems Using Tekton - Working with Skaffold Container Image Builders and Deployers
keywords: books, ci-cd, tekton, Working with Skaffold Container Image Builders and Deployers
permalink: /books/containers/kubernetes/tools/skaffold/working-with-skaffold-container-image-builders-and-deployers/
---

# Chapter 6. Working with Skaffold Container Image Builders and Deployers

<br/>

```
$ cd ~/tmp/Effortless-Cloud-Native-App-Development-Using-Skaffold/Chapter06/
$ skaffold run --profile docker
```

<br/>

```
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/reactive-web-app-5b79d5bbd7-qltjt   1/1     Running   0          25s

NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes         ClusterIP      10.96.0.1       <none>        443/TCP          4m8s
service/reactive-web-app   LoadBalancer   10.110.214.23   <pending>     8080:32585/TCP   25s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/reactive-web-app   1/1     1            1           25s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/reactive-web-app-5b79d5bbd7   1         1         1       25s
```

<br/>

```
$ kubectl get svc
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes         ClusterIP      10.96.0.1       <none>        443/TCP          4m33s
reactive-web-app   LoadBalancer   10.110.214.23   <pending>     8080:32585/TCP   50s
```

<br/>

```
// Убеждаемся, что значение профиля установлено
$ echo ${PROFILE}
```

<br/>

Если нет

<br/>

```
$ export \
    PROFILE=${USER}-minikube
```

<br/>

```
// Перестало отображаться в новых версиях
$ minikube --profile ${PROFILE} service reactive-web-app
|-----------|------------------|-------------|---------------------------|
| NAMESPACE |       NAME       | TARGET PORT |            URL            |
|-----------|------------------|-------------|---------------------------|
| default   | reactive-web-app |        8080 | http://192.168.49.2:32585 |
|-----------|------------------|-------------|---------------------------|
🎉  Opening service default/reactive-web-app in default browser...
👉  http://192.168.49.2:32585



// Но можно
$ minikube --profile ${PROFILE} service --all
```

<br/>

```
$ curl -X GET "http://192.168.49.2:32538/employee" \
  | jq
```

<br/>

```
[
  {
    "id": 1,
    "firstName": "Peter",
    "lastName": "Parker",
    "age": 25,
    "salary": 20000
  },
  {
    "id": 2,
    "firstName": "Tony",
    "lastName": "Stark",
    "age": 30,
    "salary": 40000
  },
  {
    "id": 3,
    "firstName": "Clark",
    "lastName": "Kent",
    "age": 31,
    "salary": 60000
  },
  {
    "id": 4,
    "firstName": "Bruce",
    "lastName": "Wayne",
    "age": 33,
    "salary": 100000
  }
]
```

<br/>

```
$ skaffold delete
```

<br/>

### Jib and Helm

<br/>

```
// Проверка, что проект билдится
// Нет необходимости выполнять
$ ./mvnw package
```

<br/>

OK!

<br/>

```
$ vi pom.xml
```

<br/>

Прописываю подходящую версию библиотеки jib-maven-plugin

<br/>

```xml
<groupId>com.google.cloud.tools</groupId>
<artifactId>jib-maven-plugin</artifactId>
<version>3.1.4</version>
```

<br/>

Если протухнет, смотреть здесь:

https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin#quickstart

<br/>

Helm установлен

<br/>

```
$ skaffold run --profile jibWithHelm
```

<br/>

```
Generating tags...
 - gcr.io/basic-curve-316617/reactive-web-app-helm -> gcr.io/basic-curve-316617/reactive-web-app-helm:4c4f2c8-dirty
Checking cache...
 - gcr.io/basic-curve-316617/reactive-web-app-helm: Found Locally
Starting test...
Tags used in deployment:
 - gcr.io/basic-curve-316617/reactive-web-app-helm -> gcr.io/basic-curve-316617/reactive-web-app-helm:7b7f64704771899fb746b73432c1b2cca9d5cc2ed818246763847847b4a87122
Starting deploy...
Error: UPGRADE FAILED: YAML parse error on reactive-web-app-helm/templates/deployment.yaml: error converting YAML to JSON: yaml: line 5: did not find expected node content
deploying "reactive-web-app-helm": install: exit status 1
```

<br/>

```
$ vi reactive-web-app-helm/templates/deployment.yaml
```

<br/>

Похоже индус забыл выложить файл с нужными параметрами. Не сложно поправить, но пока лень.

Думаю, можно использовать вот это
https://github.com/yrashish/Effortless-Cloud-Native-Apps-Development-using-Skaffold/blob/main/Chapter06/k8s/manifest.yaml

Но пока тоже не работает. Имиджи, что в конфигах не удается скачать. "Project not found or deleted".

<br/>

Fail!

<br/>

### Kustomize

<br/>

```
$ skaffold dev
```

<br/>

```
$ skaffold run --profile=kustomizeBase --default-repo=gcr.io/basic-curve-316617
$ skaffold run --profile=kustomizeProd --default-repo=gcr.io/basic-curve-316617
```

<br/>

```
$ skaffold delete
```

<br/>

```
$ skaffold run --profile=kustomizeDev
```

<br/>

Fail!
