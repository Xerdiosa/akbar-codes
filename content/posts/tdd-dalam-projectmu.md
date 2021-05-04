+++ 
draft = false
date = 2021-05-02T12:42:30+07:00
title = "Aplikasikan TDD dalam projectmu"
description = ""
slug = ""
authors = []
tags = [ "PPL2021", "TDD" ]
categories = [ "Software Development" ]
externalLink = ""
series = []
+++

Apa itu TDD? Apakah TDD akan meningkatkan produktifitas?

## Test Driven Development

Test Driven Development atau yang disebut juga dengan TDD adalah konsep dimana kita menuliskan test case terlebih dahulu, baik itu unit testing maupun functional testing. Pada awalnya TDD akan terdengar tidak intuitif namun setelah menguasai TDD akan banyak keunggulan yang dapat dirasakan.

## Komponen utama dalam TDD

Pada TDD, saat melakukan commit kita akan melakukanya dalam 3 tahapan yaitu RED, GREEN, dan REFACTOR.

### RED

Commit yang berlabel RED berisi testcase yang merupakan ekspektasi dari perilaku program ketika sudah selesai diimplementasikan, pada CI/CD commit red bisaanya akan menghasilkan pipeline failure.  
![RED](/images/posts/TDD/RED.png)

### GREEN

Commit yang berlabel GREEN merupakan implementasi dari testcase yang telah dikerjakan, implementasi pada green mungkin merupakan implementasi yang belum terlalu rapi dan mungkin diperlukan refactor pada kode.  
![RED](/images/posts/TDD/GREEN.png)

### REFACTOR

Commit yang berlabel REFACTOR merupakan perbaikan dari kode yang diimplementasikan pada GREEN. Commit ini bisa berisi perbaikan linter ataupun menyusun ulang logika bisnis agar kode lebih rapi.  
![RED](/images/posts/TDD/REFACTOR.png)

## Masalah yang mungkin muncul saat melakukan TDD

### Code coverage tidak mungkin 100%

Pada saat melakukan TDD, kita akan menuliskan alur program dalam bentuk testcase, pada implementasinya mungkin akan berbeda. Sehingga akan ada kode yang belum terjangkau oleh test. Pada kasus ini saya sarankan untuk menambahkan test untuk mengakomodasi percabangan tersebut.

## TDD dan produktifitas

Menerapkan TDD dapat meningkatkan produktifitas karena saat implementasi kode, kita sudah tahu akan menuliskan kode seperti apa karena percabangan sudah didefinisikan pada test.