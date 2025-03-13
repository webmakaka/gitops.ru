---
layout: page
title: Kafka
description: Kafka
keywords: messaging, kafka
permalink: /tools/messaging/kafka/
---

# Kafka

Делаю:  
2024.12.08

Не отработало! Простые ошибки при запуске контейнера с помощью docker-compose.

<br/>

## [That DevOps Guy] Running Kafka on Docker with Compose

https://www.youtube.com/watch?v=ncTosfaZ5cQ

https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/tools/messaging/kafka

<br/>

```
$ cd ~/tmp/

$ git clone https://github.com/marcel-dempers/docker-development-youtube-series

$ cd docker-development-youtube-series/tools/messaging/kafka/

$ docker-compose up --build

$ docker ps
```

<br/>

```
$ docker ps
CONTAINER ID   IMAGE                       COMMAND                CREATED          STATUS          PORTS     NAMES
ab663814f31a   aimvector/kafka:2.7.0       "start-kafka.sh"       31 seconds ago   Up 29 seconds             kafka-2
5d27f8045446   aimvector/kafka:2.7.0       "/bin/bash"            32 seconds ago   Up 30 seconds             kafka-producer
d61ba1768c4c   aimvector/zookeeper:2.7.0   "start-zookeeper.sh"   32 seconds ago   Up 30 seconds             zookeeper-1
0d88060f27fb   aimvector/kafka:2.7.0       "start-kafka.sh"       32 seconds ago   Up 30 seconds             kafka-1
c52f97bb053c   aimvector/kafka:2.7.0       "/bin/bash"            32 seconds ago   Up 30 seconds             kafka-consumer
ff179d11e800   aimvector/kafka:2.7.0       "start-kafka.sh"       32 seconds ago   Up 30 seconds             kafka-3
```

<br/>

### Создаем Topic

<br/>

```
$ docker exec -it zookeeper-1 bash
```

<br/>

```
// Create the Topic:
# /kafka/bin/kafka-topics.sh \
    --create \
    --zookeeper zookeeper-1:2181 \
    --replication-factor 1 \
    --partitions 3 \
    --topic Orders
```

<br/>

```
// Describe our Topic:
# /kafka/bin/kafka-topics.sh \
    --describe \
    --topic Orders \
    --zookeeper zookeeper-1:2181
```

<br/>

```
Topic: Orders	PartitionCount: 3	ReplicationFactor: 1	Configs:
	Topic: Orders	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: Orders	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
	Topic: Orders	Partition: 2	Leader: 2	Replicas: 2	Isr: 2
```

<br/>

### Simple Producer & Consumer

<br/>

```
$ docker exec -it zookeeper-1 bash
```

<br/>

```
// Получать сообщения и выводить их в консоль:
# /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka-1:9092,kafka-2:9092,kafka-3:9092 \
    --topic Orders \
    --from-beginning
```

<br/>

Еще 1 терминал.

<br/>

```
$ docker exec -it kafka-producer bash
```

<br/>

```
// Create the Message
# echo "New Order: 1" | \
    /kafka/bin/kafka-console-producer.sh \
    --broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 \
    --topic Orders > /dev/null
```

<br/>

```
// Create the Message
# echo "New Order: 2" | \
    /kafka/bin/kafka-console-producer.sh \
    --broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 \
    --topic Orders > /dev/null
```

<br/>

```
// Create the Message
# echo "New Order: 3" | \
    /kafka/bin/kafka-console-producer.sh \
    --broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 \
    --topic Orders > /dev/null
```

<br/>

###

```
$ docker exec -it kafka-1 bash
```

<br/>

```
# apt install -y tree
# tree /tmp/kafka-logs/
```

<br/>

```
# tree /tmp/kafka-logs/
/tmp/kafka-logs/
├── Orders-1
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-11
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-14
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-17
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-2
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-20
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-23
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-26
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-29
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-32
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-35
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-38
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-41
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-44
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-47
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-5
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── __consumer_offsets-8
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   └── leader-epoch-checkpoint
├── cleaner-offset-checkpoint
├── log-start-offset-checkpoint
├── meta.properties
├── recovery-point-offset-checkpoint
└── replication-offset-checkpoint

17 directories, 73 files
```

<br/>

```
# ls -lh /tmp/kafka-logs/Orders-*
total 8.0K
-rw-r--r-- 1 root root 10M Sep 24 05:47 00000000000000000000.index
-rw-r--r-- 1 root root 240 Sep 24 05:58 00000000000000000000.log
-rw-r--r-- 1 root root 10M Sep 24 05:47 00000000000000000000.timeindex
-rw-r--r-- 1 root root   8 Sep 24 05:47 leader-epoch-checkpoint
```

<br/>

```
# cat /tmp/kafka-logs/Orders-1/\*.log
```

<br/>

## [That DevOps Guy] Learning How to Consume Data from Kafka

https://www.youtube.com/watch?v=xNDwt1tEZkw

https://github.com/marcel-dempers/docker-development-youtube-series/blob/master/tools/messaging/kafka/consumer.md

<br/>

```
$ cd ~/tmp/

$ git clone https://github.com/marcel-dempers/docker-development-youtube-series

$ cd docker-development-youtube-series/tools/messaging/kafka/

$ docker-compose up zookeeper-1 kafka-1 kafka-2 kafka-3

$ docker ps
```

<br/>

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
ab663814f31a aimvector/kafka:2.7.0 "start-kafka.sh" 34 minutes ago Up 8 seconds kafka-2
d61ba1768c4c aimvector/zookeeper:2.7.0 "start-zookeeper.sh" 34 minutes ago Up 8 seconds zookeeper-1
0d88060f27fb aimvector/kafka:2.7.0 "start-kafka.sh" 34 minutes ago Up 8 seconds kafka-1
ff179d11e800 aimvector/kafka:2.7.0 "start-kafka.sh" 34 minutes ago Up 8 seconds kafka-3

<br/>

### Create a Topic: Orders

```
$ docker exec -it zookeeper-1 bash
```

<br/>

```
// create
# /kafka/bin/kafka-topics.sh \
    --create \
    --zookeeper zookeeper-1:2181 \
    --replication-factor 1 \
    --partitions 3 \
    --topic Orders


// describe
# /kafka/bin/kafka-topics.sh \
    --describe \
    --zookeeper zookeeper-1:2181 \
    --topic Orders

exit
```

<br/>

## Building a consumer in Go

```
$ cd ~/tmp/docker-development-youtube-series/tools/messaging/kafka/

$ docker-compose up kafka-consumer-go

$ docker-compose up kafka-producer

$ docker exec -it kafka-producer bash
```

<br/>

```
upperlim=10
for ((i=0; i<=upperlim; i++)); do
   echo "{ 'id' : 'order-$i', 'data' : 'random-data'}" | \
    /kafka/bin/kafka-console-producer.sh \
    --broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 \
    --topic Orders > /dev/null
done
```

<br/>

**consumer**

```
Attaching to kafka-consumer-go
kafka-consumer-go    | Sarama consumer up and running!...
kafka-consumer-go    | Partition:	0
kafka-consumer-go    | Offset:	0
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-0', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	0
kafka-consumer-go    | Offset:	1
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-1', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	1
kafka-consumer-go    | Offset:	0
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-2', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	1
kafka-consumer-go    | Offset:	1
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-3', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	1
kafka-consumer-go    | Offset:	2
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-4', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	2
kafka-consumer-go    | Offset:	0
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-5', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	0
kafka-consumer-go    | Offset:	2
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-6', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	2
kafka-consumer-go    | Offset:	1
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-7', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	1
kafka-consumer-go    | Offset:	3
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-8', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	2
kafka-consumer-go    | Offset:	2
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-9', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
kafka-consumer-go    | Partition:	2
kafka-consumer-go    | Offset:	3
kafka-consumer-go    | Key:
kafka-consumer-go    | Value:	{ 'id' : 'order-10', 'data' : 'random-data'}
kafka-consumer-go    | Topic:	Orders
kafka-consumer-go    |
```

<br/>

```
$ docker-compose up kafka-consumer

$ docker exec -it kafka-consumer bash
```

<br/>

```
# /kafka/bin/kafka-console-consumer.sh \
--bootstrap-server kafka-1:9092,kafka-2:9092,kafka-3:9092 \
--topic Orders --from-beginning
```

<br/>

```
{ 'id' : 'order-5', 'data' : 'random-data'}
{ 'id' : 'order-7', 'data' : 'random-data'}
{ 'id' : 'order-9', 'data' : 'random-data'}
{ 'id' : 'order-10', 'data' : 'random-data'}
{ 'id' : 'order-2', 'data' : 'random-data'}
{ 'id' : 'order-3', 'data' : 'random-data'}
{ 'id' : 'order-4', 'data' : 'random-data'}
{ 'id' : 'order-8', 'data' : 'random-data'}
{ 'id' : 'order-0', 'data' : 'random-data'}
{ 'id' : 'order-1', 'data' : 'random-data'}
{ 'id' : 'order-6', 'data' : 'random-data'}
```

<br/>

### Ordering

```
upperlim=10
for ((i=0; i<=upperlim; i++)); do
   echo "order-11: { 'id' : 'order-11', 'data' : '$i'}" | \
    /kafka/bin/kafka-console-producer.sh \
    --broker-list kafka-1:9092,kafka-2:9092,kafka-3:9092 \
    --topic Orders > /dev/null \
    --property "parse.key=true" --property "key.separator=:"
done
```
