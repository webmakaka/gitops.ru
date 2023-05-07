---
layout: page
title: Kubernetes Blue Green deployment
description: Kubernetes Blue Green deployment
keywords: devops, linux, kubernetes, Blue / Green deployment
permalink: /devops/containers/kubernetes/basics/blue-green-deployment/
---

# Blue / Green deployment

<br/>

Делаю:
21.04.2020

<br/>

https://github.com/burrsutter/9stepsawesome/blob/master/8_deployment_techniques.adoc

Подняли 2 версии приложения на разных deployment.

Протестили.

Потом на сервисе

    $ kubectl patch svc/mynode -p '{"spec":{"selector":{"app":"mynodenew"}}}'

Переключили с одного на другой с меткой {"app":"mynodenew"}

Если что-то не то, вернули deployment с нужной меткой.
