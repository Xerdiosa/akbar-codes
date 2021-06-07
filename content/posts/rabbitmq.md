+++ 
draft = false
date = 2021-06-07T10:40:33+07:00
title = "Asynchronous menggunakan message broker rabbitmq"
description = ""
slug = ""
authors = [ "Muhammad Aulia Akbar" ]
tags = [ "PPL2021", "rabbitmq", "message broker" ]
categories = [ "Software Development" ]
externalLink = ""
series = []
+++

## Asynchronous web application

Asynchronous web application adalah pemahaman dimana saat client melakukan request kepada server, proses tersebut tidak akan ditahan oleh respon dari server melainkan akan terus berjalan dan akan diupdate saat mendapat respon dari server.

![ajax](/images/posts/rabbitmq/ajax.png)
Gambar: Asynchronous dengan ajax

Keuntungan dari asynchronous web application adalah aplikasi akan menjadi lebih dinamis dan jika server membutuhkan waktu lama untuk memberikan respon, maka aplikasi tetap dapat berjalan.

## Apa itu message broker

Message broker adalah service perantara yang digunakan untuk mengirim pesan secara asynchronous dan menerapkan sistem antrian pada server. Dengan menggunakan message broker, kita dapat membuat komunikasi dengan bahasa pemrogaman yang berbeda-beda.

### cara kerja message broker

Secara singkat terdapat tiga tahapan saat menjalankan message queue.

1. Producing merupakan tahapan dimana pesan dikirimkan oleh producer kepada message queue.
2. Queue adalah tahapan dimana message queue memproses pesan yang masuk sebelum diserahkan kepada penerima.
3. Consuming adalah tahapan menerima pesan yang masuk oleh consumer dari message queue.

![rabbitmq](/images/posts/rabbitmq/rabbitmq.png)

## RabbitMQ

RabbitMQ adalah open source message broker yang dikembangkan dengan bahasa erlang. Rabbitmq melayani message queue dengan protokol AMQP, MQTT, STOMP, dll.

## Instalasi rabbitmq

Saya menggunakan docker dan docker compose untuk menjalankan rabbitmq pada environment local.

Dockerfile

``` Dockerfile
FROM rabbitmq:3.7-management
RUN rabbitmq-plugins enable rabbitmq_web_stomp
```

.docker-compose.yml

``` .docker-compose.yml
version: "3.2"
services:
  rabbitmq:
    build: .
    container_name: 'rabbitmq'
    ports:
        - 5672:5672
        - 15672:15672
        - 15674:15674
    volumes:
        - ~/.docker-conf/rabbitmq/data/:/var/lib/rabbitmq/
        - ~/.docker-conf/rabbitmq/log/:/var/log/rabbitmq
    networks:
        - rabbitmq_go_net

networks:
  rabbitmq_go_net:
    driver: bridge
```

## Implementasi RabbitMQ pada golang

``` golang
    // declaring rabbitmq
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    if err != nil {
        panic(err)
    }
    defer conn.Close()

    ch, err = conn.Channel()
    if err != nil {
        panic(err)
    }
    defer ch.Close()

    err = ch.ExchangeDeclare(
        "1806133805", // name
        "direct",     // type
        true,         // durable
        false,        // auto-deleted
        false,        // internal
        false,        // no-wait
        nil,          // arguments
    )
    if err != nil {
        panic(err)
    }

    // publishing message
    ch.Publish(
    "1806133805", // exchange
    id,           // routing key
    false,        // mandatory
    false,        // immediate
    amqp.Publishing{
        ContentType: "text/plain",
        Body:        []byte("hello, world!"),
    })
```

Berikut adalah contoh implementasi dari rabbitmq producer pada bahasa pemrograman golang. Pada contoh diatas, topik yang dibuat adalah "1806133805" dengan pesan yang dikirim adalah "hello, world!"

## Keuntungan menggunakan RabbitMQ

Dengan menggunakan rabbitmq, kita dimudahkan untuk menerapkan asynchronous web application tanpa perlu mengurus kasus-kasus concurency yang terjadi.  

## Referensi

[apa itu message broker](https://teknosejahtera.com/apa-itu-message-broker/)