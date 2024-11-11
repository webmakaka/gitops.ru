---
layout: page
title: Запуск kafka с помощью docker-compose для локальной разработки
description: Запуск kafka с помощью docker-compose для локальной разработки
keywords: devops, linux, kafka
permalink: /tools/messaging/kafka/
---

# Kafka

### Запуск kafka с помощью docker-compose для локальной разработки

<br/>

**Делаю:**  
2024.11.11

<br/>

**docker-compose.yaml**

```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  kowl:
    image: quay.io/cloudhut/kowl:v1.2.1
    restart: on-failure
    hostname: kowl
    volumes:
      - ./config.yaml:/etc/kowl/config.yaml
    ports:
      - '8080:8080'
    entrypoint: ./kowl --config.filepath=/etc/kowl/config.yaml
    depends_on:
      - kafka
```

<br/>

**config.yaml**

```yaml
kafka:
  brokers:
    - kafka:9092
```

<br/>

```
$ docker-compose up
```
