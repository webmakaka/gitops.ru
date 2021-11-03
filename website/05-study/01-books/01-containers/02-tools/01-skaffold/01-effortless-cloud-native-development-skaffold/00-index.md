---
layout: page
title: Building CI/CD Systems Using Tekton - Develop flexible and powerful CI/CD pipelines using Tekton Pipelines and Triggers
description: Building CI/CD Systems Using Tekton - Develop flexible and powerful CI/CD pipelines using Tekton Pipelines and Triggers
keywords: books, ci-cd, tekton
permalink: /study/books/containers/kubernetes/tools/skaffold/effortless-cloud-native-development-skaffold/
---

# Effortless Cloud-Native App Development Using Skaffold: Simplify the development and deployment of cloud-native Spring Boot applications on Kubernetes with Skaffold [ENG, 2021]

<br/>

English | 2021 | ISBN: 978-1801077118 | 272 Pages | PDF, EPUB | 20 MB

<br/>

Делаю:  
19.10.2021

<br/>

**GitHub:**  
https://github.com/PacktPublishing/Effortless-Cloud-Native-App-Development-Using-Skaffold

**Цветные картинки**  
http://www.packtpub.com/sites/default/files/downloads/9781801078214_ColorImages.pdf

<br/>

### [Подготовка стенда](/study/books/containers/kubernetes/tools/skaffold/setup/)

<br/>

### Section 1: The Kubernetes Nightmare – Skaffold to the Rescue

<br/>

-   Chapter 1. Code, Build, Test, and Repeat – The Application Development Inner Loop
-   Chapter 2. Developing Cloud-Native Applications with Kubernetes – A Developer's Nightmare
-   [Chapter 3. Skaffold – Easy-Peasy Cloud-Native Kubernetes Application Development](/study/books/containers/kubernetes/tools/skaffold/skaffold-easy-peasy-cloud-native-kubernetes-application-development/)

<br/>

### Section 2: Getting Started with Skaffold

<br/>

-   Chapter 4. Understanding Skaffold's Features and Architecture
-   Chapter 5. Installing Skaffold and Demystifying Its Pipeline Stages
-   [Chapter 6. Working with Skaffold Container Image Builders and Deployers](/study/books/containers/kubernetes/tools/skaffold/working-with-skaffold-container-image-builders-and-deployers/)

<br/>

### Section 3: Building and Deploying Cloud-Native Spring Boot Applications with Skaffold

<br/>

-   Chapter 7. Building and Deploying a Spring Boot Application with the Cloud
    Code Plugin
-   Chapter 8. Deploying a Spring Boot Application to Google Kubernetes Engine
    Using Skaffold
-   Chapter 9. Creating a Production-Ready CI/CD Pipeline with Skaffold
-   Chapter 10. Exploring Skaffold Alternatives, Best Practices, and Pitfalls

<br/>

### Chapter 7

<br/>

```
$ cd ~/tmp/Effortless-Cloud-Native-App-Development-Using-Skaffold/Chapter07/
$ ./mvnw package
$ skaffold dev
```

<br/>

```
$ skaffold dev
Listing files to watch...
 - breathe
Generating tags...
 - breathe -> breathe:4c4f2c8-dirty
Checking cache...
 - breathe: Found Locally
Tags used in deployment:
 - breathe -> breathe:dfadbcd7482c04d0eda01131ebb1b2e32132d8b3256aae76b63bef1c17545eb8
Starting deploy...
 - service/scanner created
 - deployment.apps/scanner created
 - ingress.networking.k8s.io/scanner created
Waiting for deployments to stabilize...
 - deployment/scanner: creating container scanner
    - pod/scanner-57869fc7cb-2sd97: creating container scanner
    - pod/scanner-57869fc7cb-l5jgx: container scanner is waiting to start: a3333333/scanner:0.0.1-SNAPSHOT can't be pulled
 - deployment/scanner failed. Error: creating container scanner.
Cleaning up...
 - service "scanner" deleted
 - deployment.apps "scanner" deleted
 - ingress.networking.k8s.io "scanner" deleted
1/1 deployment(s) failed
```

<br/>

Из коробки не работает. Плагины для IDE не интересуют. УГ.
