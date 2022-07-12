---
layout: page
title: Пример с балансировщиком нагрузки
description: Пример с балансировщиком нагрузки
keywords: devops, clouds, Пример с балансировщиком нагрузки
permalink: /tools/clouds/oracle/free-tier/load-balancer/
---

# Пример с балансировщиком нагрузки

https://www.youtube.com/watch?v=nFlFswEpwnA

<br/>

Networking -> Virtual Cloud Networks -> Create VCN

<br/>

С помощью мастера

<br/>

Name: Marley-VCN

CIDR BLOCK: 10.0.0.0/16

<br/>

Default Security List for Marley-VCN

Ingress Rules

10.0.0.0/16 80

<br/>

Compute -> Instances -> Create Instance

marley-instance

ubuntu 20

<br/>

    $ ssh ubuntu@<public-ip>

<br/>

Запускаю приложение на 80 порту

<br/>

Networking -> Load Balancers -> Create Load Balancer

Marley-LB

Public

Virtual Cloud Network -> Marley-VCN

Subnet -> Public Subnet Marley

Next

Weighted Round Robin

Add Backends

Protocol TCP

Next

Listener Name: marley-listener_lb

HTTP

Create

<br/>

VCN -> Default Security List for Marley-VCN

0.0.0.0/0

80
