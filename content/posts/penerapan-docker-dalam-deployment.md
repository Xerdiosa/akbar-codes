+++ 
draft = false
date = 2021-05-24T09:49:37+07:00
title = "Penerapan Docker Dalam Deployment"
description = "Artikel ini akan menjelaskan imple"
slug = ""
authors = [ "Muhammad Aulia Akbar" ]
tags = [ "PPL2021", "Docker", "deployment" ]
categories = [ "Software Development" ]
externalLink = ""
series = []
+++

## Containerization

Pada saat melakukan software development, kode yang dibuat oleh developer akan di *deploy* ke server yang ada. Apakah kita dapat menjamin bahwa hasil pengerjaan akan persis sama pada kondisi perangkat milik developer dan server. Bahkan pada sesama perangkat developer yang memiliki versi yang berbeda dapat terjadi perbedaan dari perilaku program. Untuk menyelesaikan masalah di atas, kita dapat "membungkus" program yang telah dibuat kedalam kontainer. Kontainer adalah suatu konsep dimana lingkugan dari kode yang berjalan akan distandarisasi agar jalanya program lebih konsisten.

## Perbedaan Containerization dengan Virtualization

![container vs vm](/images/posts/docker/containers-vs-virtual-machines.jpg)
Virtual machine akan memvirtualisasikan semua komponen yang  ada dalam sistem operasi, sedangkan pada kontainer hanya memisahkan antar kerja didalam sistem operasinya sendiri(masih menggunakan kernel yang sama). Dengan menggunakan kontainer maka beban yang pada CPU untuk menjalankan aplikasi akan lebih rendah, sehingga program dapat berjalan dengan lebih efisien.

## Docker

![Docker](/images/posts/docker/docker-logo.png)
Docker adalah salah satu cara untuk melakukan kontainerisasi pada program. Dengan docker, kita dapat membuat sebuah "gambar" dari sistem yang ingin dijalankan. Dari gambar yang ada, kita dapat memastikan bahwa pada tiap perangkat dimana program tersebut dijalankan, perilaku program akan menjadi sama.

## Dockerfile dan docker-compose

Untuk menjalankan docker, maka dibutuhkan sebuah dockerfile. Dockerfile merupakan "resep" tentang bagaimana image docker akan dijalankan. Sebuah dockerfile dapat dibentuk diatas dockerfile lainya yang memungkinkan modularisasi dari docker image.

``` Dockerfile
FROM python:3

COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
RUN useradd guest
USER guest
CMD [ "python" ,"main.py" ]
```

Berikut adalah contoh dari dockerfile, Command di atas akan mengambil image python 3 dan menjalankan perintah yang diperlukan untuk membuat image docker yang akan menjalankan server `main.py`.

Untuk mempermudah jalanya docker, terdapat sebuah fitur yang dinamakan docker-compose. Dengan docker-compose konfigurasi dari dockerfile dapat dibuat menjadi lebih moduler karena tidak perlu membuat image baru ketika perubahan hanya dilakukan pada docker-compose. Selain itu kita bisa menjalankan banyak image docker hanya dengan docker compose.

``` docker-compose
version: '2'

services:
  web:
    build: ./web
    restart: always
    ports:
      - "8000:80"
    environment: 
      - REST_IP=10.5.0.6
    networks: 
      backend:
        ipv4_address: 10.5.0.5
  rest:
    build: ./rest
    restart: always
    environment: 
      - PORT=80
    networks: 
      backend:
        ipv4_address: 10.5.0.6
networks:
  backend:
    driver: "bridge"
    ipam: 
      config: 
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1
```

Berikut adalah contoh dari file docker-compose. Docker-compose dapat menjalankan image rest dan web sekaligus. Selain itu docker compose juga dapat membuat subnet yang "menjembatani" komunikasi antar image docker.

## Penerapan docker pada PPL kami

Projek PPL kali ini tidak menggunakan docker sebagai image pada deployment nya, atau lebih tepatnya kami tidak membuat docker, namun  kontainerisasi tetap diimplementasikan dalam bentuk image heroku yang dikelola oleh [heroku](https://herokuapp.com).
![Docker](/images/posts/docker/stack.png)

Namun bukan berarti bahwa kami tidak menerapkan docker sama sekali dalam pengembangan projek ini. Pada saat [deployment](/posts/how-to-deploy-with-gitlab/), script pada .gitlab-ci.yml akan dijalankan pada kontainer docker(menggunakan image yang ada pada). Hal ini akan memudahkan proses ci karena kelompok kami menggunakan gitlab runner pribadi dan gitlab runner yang disediakan oleh PPL, jika tidak menggunakan container maka eksekusi program mungkin akan berbeda antar runner yang ada.

## Kesimpulan

Implementasi docker sangatlah beragam, kita dapat menulis dockerfile secara manual, kode yang kita jalankan akan di eksekusi dengn docker pada layanan PAAS(platform as a service), docker juga dapat dijalankan pada runner gitlab.

## Referensi

* [A Practical Guide to Choosing between Docker Containers and VMs
](https://www.weave.works/blog/a-practical-guide-to-choosing-between-docker-containers-and-vms)
