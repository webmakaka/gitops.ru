---
layout: page
title: Clouds Amazon
description: Deploy Clouds, AWS
keywords: Deploy, Clouds, AWS
permalink: /tools/clouds/aws/
---

# Clouds Amazon (AWS)

<br/>

[Прикрутить свой домен к AWS без переноса его в AWS и использованием CloudFlare](https://medium.com/@bobthomas295/combining-aws-serverless-with-cloudflare-sub-domains-338a1b7b2bd)

<br/>

**В заработавшем варианте:**

Cloudflare -> SSL/TLS -> Full (strict)

В DNS

```
Type: CNAME
Name: _6be2d4c99b2c4199a3abf29716f8fd4e
Target: _f13db5c800d33fd3174874c84f677c33.bwlshdtstt.acm-validations.aws
Proxy status: DNS only
TTL: Auto

Type: CNAME
Name: api
Target: d2siadvb5bmoyb.cloudfront.net
Proxy status: Proxied
TTL: Auto

Type: CNAME
Name: webmak.site
Target: d2siadvb5bmoyb.cloudfront.net
Proxy status: Proxied
TTL: Auto
```

<!--

<a href="https://emea-resources.awscloud.com/rus-ua-cis19-webinar-how-to-split-monolith-application-into-micro-services" rel="nofollow">RUS/UA/CIS19: Webinar - How to split monolith application into micro services</a>

-->

<a href="https://aws.amazon.com/ru/getting-started/serverless-web-app/" rel="nofollow">Создание бессерверного интернет-приложения</a>

<a href="//jsdev.ru/schools/rs-school/nodejs/aws/" rel="nofollow">Лекции по AWS в RS School</a>
