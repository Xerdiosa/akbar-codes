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

## Docker

![Docker](/images/posts/docker/docker-logo.png)
Docker adalah salah satu cara untuk melakukan kontainerisasi pada program. Dengan docker, kita dapat membuat sebuah "gambar" dari sistem yang ingin dijalankan. Dari gambar yang ada, kita dapat memastikan bahwa pada tiap perangkat dimana program tersebut dijalankan, perilaku program akan menjadi sama.

## Penerapan docker pada PPL kami

Projek PPL kali ini tidak menggunakan docker sebagai image pada deployment nya, atau lebih tepatnya kami tidak membuat docker, namun  kontainerisasi tetap diimplementasikan dalam bentuk image heroku yang dikelola oleh [heroku](https://herokuapp.com).
![Docker](/images/posts/docker/stack.png)

Namun bukan berarti bahwa kami tidak menerapkan docker sama sekali dalam pengembangan projek ini. Pada saat [deployment](/posts/how-to-deploy-with-gitlab/), script pada .gitlab-ci.yml akan dijalankan pada kontainer docker(menggunakan image yang ada pada). Hal ini akan memudahkan proses ci karena kelompok kami menggunakan gitlab runner pribadi dan gitlab runner yang disediakan oleh PPL, jika tidak menggunakan container maka eksekusi program mungkin akan berbeda antar runner yang ada.

## Kesimpulan

Implementasi docker sangatlah beragam, kita dapat menulis dockerfile secara manual, kode yang kita jalankan akan di eksekusi dengn docker pada layanan PAAS(platform as a service), docker juga dapat dijalankan pada runner gitlab.
