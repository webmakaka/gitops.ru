---
layout: page
title: Dockerfile для сборки образа docker с использованием внутрненних репо
description: Dockerfile для сборки образа docker с использованием внутрненних репо
keywords: Repo, docker, nexus, pip
permalink: /tools/repos/
---

# Dockerfile для сборки образа docker с использованием внутрненних репо

<br/>

```
FROM python:3.8-slim

RUN sed -i /etc/apt/sources.list -e 's/^deb/#deb/'
RUN touch /etc/apt/sources.list.d/ourcompanyname.list
RUN echo "deb http://debianrepo.ourcompanyname.ru/debian bullseye main" >> /etc/apt/sources.list.d/ourcompanyname.list
RUN echo "deb http://debianrepo.ourcompanyname.ru/debian-security bullseye-security main" >> /etc/apt/sources.list.d/ourcompanyname.list
RUN echo "deb http://debianrepo.ourcompanyname.ru/debian bullseye-updates main" >> /etc/apt/sources.list.d/ourcompanyname.list

RUN apt-get update && apt-get install -y procps gcc make curl wget vim net-tools iputils-ping && apt-get clean
RUN python -m pip install --upgrade pip

WORKDIR /app

COPY ./ ./

RUN pip3 install -r requirements.txt --index-url https://nexus.ournexusreposite.ru/repository/pypi-proxy/simple/ --trusted-host ournexusreposite.ourcompanyname.ru

RUN adduser --system --no-create-home --disabled-login --group app && chown -R app:app /app
```
