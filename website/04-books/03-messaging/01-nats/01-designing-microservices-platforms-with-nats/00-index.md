---
layout: page
title: Designing Microservices Platforms with NATS
description: Designing Microservices Platforms with NATS
keywords: study, books, messaging, nats, Designing Microservices Platforms with NATS
permalink: /books/tools/messaging/nats/designing-microservices-platforms-with-nats/
---

# [Chanaka Fernando] Designing Microservices Platforms with NATS [ENG, 2021]

<br/>

English | 2021 | ISBN: 978-1801072212 | 356 Pages | PDF, EPUB | 20 MB

<br/>

**GitHub:**  
https://github.com/PacktPublishing/Designing-Microservices-Platforms-with-NATS

<br/>

### A Practical Example of Microservices with NATS

Сразу пробуем практику

<br/>

```
$ apt install -y jq
```

<br/>

```
$ docker run -p 4222:4222 -ti nats:latest
```

<br/>

```
$ cd ~/tmp/
$ git clone git@github.com:PacktPublishing/Designing-Microservices-Platforms-with-NATS.git
```

<br/>

```
$ cd Designing-Microservices-Platforms-with-NATS/chapter6/
```

<br/>

    $ vi docker-compose.yml

<br/>

```yaml
version: '3'
services:
  mysql-dev:
    restart: always
    image: mysql
    ports:
      - '3306:3306'
    volumes:
      - ./mysql:/etc/mysql/conf.d
    environment:
      MYSQL_DATABASE: opd_data
      MYSQL_ROOT_PASSWORD: pA55w0rd1
```

<br/>

```
$ docker-compose up
```

<br/>

```
$ telnet localhost 3306
```

<br/>

```
$ sudo apt update -y
$ sudo apt install -y mysql-client
```

<br/>

```
$ mysql --user=root --password=pA55w0rd1 -h 127.0.0.1 opd_data
```

<br/>

```
mysql> SELECT DATABASE();
```

<br/>

```
$ cd ~/tmp/Designing-Microservices-Platforms-with-NATS/chapter6/resources/
```

<br/>

```
// Добавляем данные в базу
$ mysql --user=root --password=pA55w0rd1 -h 127.0.0.1 opd_data < ./mysql.sql
```

<br/>

```
$ mysql --user=root --password=pA55w0rd1 -h 127.0.0.1 opd_data
```

<br/>

```
mysql> show tables;
+-----------------------+
| Tables_in_opd_data    |
+-----------------------+
| discharge_details     |
| inspection_details    |
| inspection_reports    |
| medication_reports    |
| patient_details       |
| patient_registrations |
| release_reports       |
| test_reports          |
+-----------------------+
```

<br/>

### Registration service

```
$ cd ~/tmp/Designing-Microservices-Platforms-with-NATS/chapter6/registration-service/
```

<br/>

```
$ go run cmd/main.go -dbName opd_data -dbUser root -dbPassword pA55w0rd1
```

**output:**

```
***
2021/12/25 19:37:34 Listening for HTTP requests on 0.0.0.0:9090
```

<br/>

```
$ curl \
    --data '{"full_name":"chanaka fernando","address":"44, sw19, london","id":200, "sex":"male", "phone":222222222}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9090/opd/patient/register" \
    | jq
```

**output:**

```
{
  "id": 200,
  "token": 1
}
```

<br/>

### Inspection service

```
$ cd ~/tmp/Designing-Microservices-Platforms-with-NATS/chapter6/inspection-service/
```

<br/>

```
$ go run cmd/main.go -dbName opd_data -dbUser root -dbPassword pA55w0rd1
```

**output:**

```
***
2021/12/25 19:44:20 Listening for HTTP requests on 0.0.0.0:9091
```

<br/>

```
$ curl \
    --data '{"id":200, "time":"2021/07/12 10:21 AM", "observations":"ABC Syncrome", "medication":"XYZ x 3", "tests":"FBT, UC", "notes":"possible Covid-19"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9091/opd/inspection/record" \
    | jq
```

**output:**

```
{
  "id": 200,
  "medication": "XYZ x 3",
  "tests": "FBT, UC",
  "notes": "possible Covid-19"
}
```

<br/>

### Treatment service

```
$ cd ~/tmp/Designing-Microservices-Platforms-with-NATS/chapter6/treatment-service
```

<br/>

```
$ go run cmd/main.go -dbName opd_data -dbUser root -dbPassword pA55w0rd1
```

**output:**

```
***
2021/12/25 19:48:09 Listening for HTTP requests on 0.0.0.0:9092
```

<br/>

```
$ curl \
    --data '{"id":200,"time":"2021 07 12 4:35PM","dose":"xyz x 1, abc x 2","notes":"low fever"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9092/opd/treatment/medication" \
    | jq
```

**output:**

```
"Record updated successfully"
```

<br/>

### Release service

```
$ cd ~/tmp/Designing-Microservices-Platforms-with-NATS/chapter6/release-service
```

<br/>

```
$ go run cmd/main.go -dbName opd_data -dbUser root -dbPassword pA55w0rd1
```

**output:**

```
***
2021/12/25 19:49:49 Listening for HTTP requests on 0.0.0.0:9093
```

<br/>

```
$ curl \
    --data '{"id":200,"time":"2021 07 12 9:35 PM","state":"discharge","post_medication":"NM x 10 days","next_visit":"2021 08 12 09:00AM"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9093/opd/release/discharge" \
    | jq
```

**output:**

```
"Patient discharge recorded successfully"
```

<br/>

### Testing the OPD application

<br/>

```
$ curl \
    --data '{"full_name":"John Doe","address":"44, sw19, london","id":200, "sex":"male", "phone":222222222}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9090/opd/patient/register" \
    | jq
```

<br/>

```
$ curl "http://localhost:9091/opd/inspection/pending"  \
    | jq
```

<br/>

```
[
  {
    "id": 200,
    "token": 2
  }
]
```

<br/>

```
$ curl \
    --data '{"id":200, "time":"2021/07/12 10:21 AM", "observations":"ABC Syncrome", "medication":"XYZ x 3", "tests":"FBT, UC", "notes":"possible Covid-19"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9091/opd/inspection/record" \
    | jq
```

<br/>

```
{
  "id": 200,
  "medication": "XYZ x 3",
  "tests": "FBT, UC",
  "notes": "possible Covid-19"
}
```

<br/>

```
$ curl "http://localhost:9092/opd/treatment/pending" \
 | jq
```

<br/>

```
[
  {
    "id": 200,
    "medication": "XYZ x 3",
    "tests": "FBT, UC",
    "notes": "possible Covid-19"
  }
]
```

<br/>

```
$ curl \
    --data '{"id":200,"time":"2021 07 12 4:35 PM","dose":"xyz x 1, abcx 2","notes":"low fever"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9092/opd/treatment/medication" \
    | jq
```

<br/>

```
"Record updated successfully"
```

<br/>

```
$ curl \
    --data '{"id":200,"time":"2021 07 12 4:35 PM","test_name":"FBC","status":"sample collected", "notes":"possible covid-19"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9092/opd/treatment/tests" \
    | jq
```

<br/>

```
"Test recorded successfully"
```

<br/>

```
"Record updated successfully"
```

<br/>

```
$ curl \
    --data '{"id":200,"time":"2021 07 12 8:35 PM","next_state":"discharge","post_medication":"NM x 10 days"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9092/opd/treatment/release" \
    | jq
```

<br/>

```
"Release event published"
```

<br/>

```
$ curl "http://localhost:9093/opd/release/pending" \
 | jq
```

<br/>

```
[
  {
    "id": 200,
    "time": "2021 07 12 8:35 PM",
    "next_state": "discharge",
    "post_medication": "NM x 10 days"
  }
]
```

<br/>

```
$ curl \
    --data '{"id":200,"time":"2021 07 12 9:35 PM","state":"discharge","post_medication":"NM x 10 days","next_visit":"2021 08 12 09:00AM"}' \
    --header "Content-Type: application/json" \
    --request POST \
    --url "http://localhost:9093/opd/release/discharge" \
    | jq
```

<br/>

```
"Patient discharge recorded successfully"
```
