+++ 
draft = false
date = 2021-04-04T22:47:09+07:00
title = "Continous deployment dengan gitlab ci"
description = "Artikel ini akan menjelaskan command yang dipakai pada gitlab ci"
slug = ""
authors = [ "Muhammad Aulia Akbar" ]
tags = [ "PPL2021", "gitlab ci" ]
categories = [ "Software Development" ]
externalLink = ""
series = []
+++

# Continous deployment dengan gitlab ci

## Pendahuluan

Pada saat kita selesai mengembangkan sebuah aplikasi, tentu saja kita ingin agar fitur baru yang telah dikerjakan dapat dirasakan oleh pengguna. Hal ini dapat dilakukan dengan melakukan deployment. Deployment dapat dilakukan secara manual, namun pada artikel kali ini, kita akan membahas cara agar deployment dapat dilakukan secara otomatis menggunakan runner yang disedakan oleh gitlab.

## .gitlab-ci.yml

Ini adalah .gitlab-ci.yml yang dipakai pada proyek PPL kelompok kami, saya akan menjelaskan fungsi dari bagian-bagian yang ada:

``` .gitlab-ci.yml
stages:
  - test
  - sonar-scanner-test
  - deploy


unit-test:
  image: golang:alpine3.13
  stage: test
  script:
    - apk add --no-cache git build-base
    - go get ./...
    - go test $(go list ./... | grep -v /vendor/) -v -coverprofile .testCoverage.txt
  artifacts:
    paths:
      - .testCoverage.txt

checkstyle:
  image: golang:1.16
  stage: test
  script:
    - go get ./...
    - go get -u golang.org/x/lint/golint
    - golint -set_exit_status ./...
  allow_failure: true

sonarqube-dev:
  stage: sonar-scanner-test
  image:
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  script:
    - sonar-scanner
      -Dsonar.host.url=$SONARQUBE_HOST_URL
      -Dsonar.login=$SONARQUBE_TOKEN
      -Dsonar.projectKey=$SONARQUBE_PROJECT_KEY
      -Dsonar.branch.target=staging
      -Dsonar.branch.name=$CI_COMMIT_REF_NAME
      -Dsonar.exclusions=mocks/**,infrastructures/**,main.go
  except:
    - staging
    - master

sonarqube:
  stage: sonar-scanner-test
  image:
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  script:
    - sonar-scanner
      -Dsonar.host.url=$SONARQUBE_HOST_URL
      -Dsonar.login=$SONARQUBE_TOKEN
      -Dsonar.projectKey=$SONARQUBE_PROJECT_KEY
      -Dsonar.branch.name=$CI_COMMIT_REF_NAME
      -Dsonar.exclusions=mocks/**,infrastructures/**,main.go
  only:
    - staging
    - master
    

deployment-staging:
  image: ruby:2.4
  stage: deploy
  before_script:
    - gem install dpl
    - wget -qO- https://cli-assets.heroku.com/install-ubuntu.sh | sh
  script:
    - dpl --provider=heroku --app=$HEROKU_STAGING_APP --api-key=$HEROKU_TOKEN
  environment:
    name: Staging
    url: $HEROKU_STAGING_URL
  only:
    - staging

deployment-production:
  image: ruby:2.4
  stage: deploy
  before_script:
    - gem install dpl
    - wget -qO- https://cli-assets.heroku.com/install-ubuntu.sh | sh
  script:
    - dpl --provider=heroku --app=$HEROKU_PRODUCTION_APP --api-key=$HEROKU_TOKEN
  environment:
    name: Production
    url: $HEROKU_PRODUCTION_URL
  only:
    - master
```

### Stages

``` .gitlab-ci.yml
stages:
  - test
  - sonar-scanner-test
  - deploy
```

stages merupakan urutan dari tahapan yang akan dikerjakan oleh runner, pada kasus kali ini, urutan dari pekerjaan yang ada adalah `test -> sonar-scanner-test -> deploy`. Agar pekerjaan pada tahapan selanjutnya dapat dimulai, semua pekerjaan pada stage yang dikerjakan harus selesai dan sukses dikerjakan(kecuali pekerjaan tersebut ditandai sebagai `allow_failure=true`). Jika terdapat beberapa pekerjaan yang valid, maka pekerjaan tersebut akan dikerjakan secara bersamaan.

### jobs

``` .gitlab-ci.yml
deployment-staging:
  image: ruby:2.4
  stage: deploy
  before_script:
    - gem install dpl
    - wget -qO- https://cli-assets.heroku.com/install-ubuntu.sh | sh
  script:
    - dpl --provider=heroku --app=$HEROKU_STAGING_APP --api-key=$HEROKU_TOKEN
  environment:
    name: Staging
    url: $HEROKU_STAGING_URL
  only:
    - staging
```

*Jobs* merupakan fitur utama dari runner ini, proses deployment dikerjakan disini. Terdapat beberapa bagian dari jobs yang akan dijelaskan satu per satu disini.

#### image

``` .gitlab-ci.yml
image: ruby:2.4
```

*Jobs* menggunakan image dari docker sebagai environment saat melakukan tugasnya. Untuk melihat image yang dapat dipakai, dapat dilihat di https://hub.docker.com/search.

#### stage

``` .gitlab-ci.yml
stage: deploy
```

Disini, kita mendefinisikan pekerjaan ini berada di tahapan apa.

#### before script & script

``` .gitlab-ci.yml
  before_script:
    - gem install dpl
    - wget -qO- https://cli-assets.heroku.com/install-ubuntu.sh | sh
  script:
    - dpl --provider=heroku --app=$HEROKU_STAGING_APP --api-key=$HEROKU_TOKEN
```

Pada before script, kita memasukkan command yang sifatnya sebagai persiapan dari deployment. Pada script, kita memasukkan command yang merupakan tahapan dari deployment itu sendiri, pada kasus ini kita menggunakan program bernama dpl untuk melakukan proses deployment ke heroku secara otomatis.

#### environment

``` .gitlab-ci.yml
  environment:
    name: Staging
    url: $HEROKU_STAGING_URL
```

Bagian ini digunakan untuk menandai untuk enviroment mana deployment ini dilakukan. Pada kasus ini, deployment akan dilakukan ke environment staging, dengan URL '$HEROKU_STAGING_URL'.

#### only & except

``` .gitlab-ci.yml
  only:
    - staging
```

Jika anda ingin pekerjaan tersebut hanya dikerjakan pada branch tertentu, maka anda dapat menambahkan perintah `only` pada gitlab ci.

``` .gitlab-ci.yml
  except:
    - staging
    - master
```

Sedangkan jika anda ingin pekerjaan tersebut tidak dikerjakan pada branch tertentu, maka anda dapat menambahkan perintah `except` pada gitlab ci.

#### allow failure

``` .gitlab-ci.yml
  allow_failure: true
```

Pada saat suatu pekerjaan gagal diselesaikan, maka pekerjaan tahapan selanjutnya tidak akan dikerjakan. Pada kasus dimana pekerjaan tersebut tidak krusial pada pengembangan software(contohnya pada *jobs* checkstyle), anda dapat menambahkan `allow_failure: true` untuk menanadai bahwa tahapan pekerjaan selanjutnya dapat dikerjakan walaupun pekerjaan ini gagal.

### environment variable

``` .gitlab-ci.yml
$HEROKU_STAGING_URL
```

Pada proses deployment, sebaiknya kita tidak memberikan api key atau informasi yang sensitif pada .gitlab-ci.yml . Oleh karena itu, kita dapat menggunakan fitur *environment variable* yang disediakan oleh gitlab untuk menyembunyikan variabel tersebut. Untuk menambahkan *environment variable*, anda memerlukan akses *maintainer* atau lebih tinggi pada project. Jika anda memiliki akses tersebut, maka akan terdapat tombol CI/CD pada setting.  
![settings->ci/cd](/images/posts/how-to-deploy-with-gitlab-ci/settings.png)

Pada bagian settings, terdapat kolom variable yang dapat dikembangkan menjadi tabel berisi mapping key dan value.  

![settings->ci/cd](/images/posts/how-to-deploy-with-gitlab-ci/ci.png)

Disini kita dapat menambahkan variabel ke dalam project. Terdapat dua atribut khusus untuk masing-masing variabe yaitu: protected dan masked. Jika variabel tersebut ditandai sebagai protected, maka variabel tersebut hanya dapat dipakai bada branch yang sifatnya protected. Sedangkan jika variabel tersebut ditandai sebagai masked, maka runner akan menyembunyikan variabel tersebut jika variabel tersebut ditampilkan pada output log.

## Kesimpulan

Perintah yang ditampilkan di atas dapat membantu dalam otomasi deployment. Continous deployment dengan gitlab ci dapat menghemat waktu dan biaya pada tahapan pengembangan software.
