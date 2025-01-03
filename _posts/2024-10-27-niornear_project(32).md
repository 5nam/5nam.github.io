---
title:  "[니어니어 프로젝트] 시스템 환경 및 CI/CD 정리"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"

---

## 아키텍처

![image-20241027200813648]({{site.url}}/assets/images/2024-10-27-niornear_project(32)/image-20241027200813648.png)

## 운영 서버 배포

**point!** RDS 를 private VPC 에 생성

- RDS 가 private VPC 에 있기 때문에 bastion-host 를 통해 접근해야 함

**[운영 서버에 배포하는 과정]**

1. develop 브랜치 > main 브랜치로 PR, merge

2. 배포 스크립트 실행

   1️⃣ bastion-host 접속

   2️⃣ git pull 로 최신 코드 반영

   3️⃣ 빌드 및 테스트

   4️⃣ jar 파일 운영 서버로 scp 를 통해 전송

   5️⃣ 운영 서버로 접속 후 jar 파일 배포

![image-20241027202038289]({{site.url}}/assets/images/2024-10-27-niornear_project(32)/image-20241027202038289.png)

```yaml
name: CI/CD 배포 자동화

on:
  push:
    branches:
      - main

jobs:
  Deploy:
    runs-on: ubuntu-latest

    steps:
      - name: 리포지토리 파일 불러오기
        uses: actions/checkout@v4

      - name: 빌드, 테스트를 위한 java17 설치
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: bastion-server 에서 빌드하고, 빌드된 파일 운영 서버에 scp 로 전송
        uses: appleboy/ssh-action@v1.0.3
        env:
          APPLICATION_PROPERTIES_S3: ${{ secrets.APPLICATION_PROPERTIES_S3 }}
          APPLICATION_PROPERTIES_iamport: ${{ secrets.APPLICATION_PROPERTIES_IMPORT }}
          APPLICATION_PROPERTIES_oauth: ${{ secrets.APPLICATION_PROPERTIES_OAUTH }}
          APPLICATION_PROPERTIES_rds: ${{ secrets.APPLICATION_PROPERTIES_RDS }}
          APPLICATION_PROPERTIES_sms: ${{ secrets.APPLICATION_PROPERTIES_SMS }}
          APPLICATION_PROPERTIES_ssl: ${{ secrets.APPLICATION_PROPERTIES_SSL }}
          EC2_PRIVATE_KEY: ${{secrets.EC2_PRIVATE_KEY}}
        with:
          host: ${{ secrets.EC2_BASTION_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          envs: |
            APPLICATION_PROPERTIES_S3
            APPLICATION_PROPERTIES_import
            APPLICATION_PROPERTIES_oauth
            APPLICATION_PROPERTIES_rds
            APPLICATION_PROPERTIES_sms
            APPLICATION_PROPERTIES_ssl
            EC2_PRIVATE_KEY
          script_stop: true
          script: |
            cd /home/ubuntu/BE
            git pull origin main
            ./gradlew clean build
            mv ./build/libs/*SNAPSHOT.jar /home/ubuntu/niornear/deploy/project.jar
            cd /home/ubuntu/niornear/deploy
            sudo scp -i niornear-server.pem project.jar ubuntu@43.202.33.58:/home/ubuntu/niornear/tobe
            echo "yes"

      - name: 서버 배포
        uses: appleboy/ssh-action@v1.0.3 # 사용할 라이브러리 명시

        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}

          script: |
            rm -rf ~/niornear/current
            mkdir ~/niornear/current
            mv ~/niornear/tobe/project.jar ~/niornear/current/project.jar
            cd ~/niornear/current
            sudo fuser -k -n tcp 443 || true
            sudo nohup java -jar project.jar > ../output.log 2>&1 &
```



## 개발 서버 배포

[개발 서버에 배포하는 과정]

1. 각자 작업 브랜치 > develop 브랜치로 PR, merge

2. 배포 스크립트 실행

   1️⃣ github action 서버에서 빌드 및 테스트

   2️⃣ jar 파일 개발 서버로 scp 를 통해 전송

   3️⃣ 개발 서버로 접속 후 jar 파일 배포

![image-20241027202605007]({{site.url}}/assets/images/2024-10-27-niornear_project(32)/image-20241027202605007.png)

```yaml
name: niornear dev CI/CD

on:
  pull_request:
    types: [closed] # merge 가 되었을 때 돌으라는 의미
  workflow_dispatch: # (2).수동 실행도 가능하도록

jobs:
  build:
    runs-on: ubuntu-latest # (3).OS환경
    if: github.event.pull_request.merged == true && github.event.pull_request.base.ref == 'develop' # 모든 브랜치에 merge 됐을 때가 아니라, develop 이라는 브랜치에 merge 가 됐을 때만 돌아가라!

    steps:
      - name: Checkout # 코드 가져오는 과정
        uses: actions/checkout@v2 # (4).코드 check out / check out 하는 범위는 github 의 최상단 루트부터

      - name: Set up JDK 17
        uses: actions/setup-java@v3 # jdk 설치할 때 actions 에서 제공해주는 거 사용하겠다는 것
        with:
          java-version: 17 # (5).자바 설치
          distribution: 'adopt'

      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew
        shell: bash # (6).권한 부여

      - name: application.properties 생성
        env:
          APPLICATION_PROPERTIES_S3: ${{ secrets.APPLICATION_PROPERTIES_S3 }}
          APPLICATION_PROPERTIES_iamport: ${{ secrets.APPLICATION_PROPERTIES_IMPORT }}
          APPLICATION_PROPERTIES_oauth: ${{ secrets.APPLICATION_PROPERTIES_OAUTH }}
          APPLICATION_PROPERTIES_rds: ${{ secrets.APPLICATION_PROPERTIES_RDS }}
          APPLICATION_PROPERTIES_sms: ${{ secrets.APPLICATION_PROPERTIES_SMS }}
          APPLICATION_PROPERTIES_ssl: ${{ secrets.APPLICATION_PROPERTIES_SSL }}

        run: |
          echo "$APPLICATION_PROPERTIES_S3" > ./src/main/resources/application-s3.properties
          echo "$APPLICATION_PROPERTIES_iamport" > ./src/main/resources/application-iamport.properties
          echo "$APPLICATION_PROPERTIES_oauth" > ./src/main/resources/application-oauth.properties
          echo "$APPLICATION_PROPERTIES_rds" > ./src/main/resources/application-rds.properties
          echo "$APPLICATION_PROPERTIES_sms" > ./src/main/resources/application-sms.properties
          echo "$APPLICATION_PROPERTIES_ssl" > ./src/main/resources/application-ssl.properties

      - name: 빌드 및 테스트
        run: |
          ./gradlew clean build

      - name: 빌드된 파일 이름 변경
        run: mv ./build/libs/*SNAPSHOT.jar ./project.jar

      - name: SCP 로 EC2 에 파일 전송
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_DEV_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_DEV_PRIVATE_KEY }}
          source: ./project.jar
      #          source: ./build/libs/*SNAPSHOT.jar
          target: ~/niornear/tobe

      - name: 서버 배포
        uses: appleboy/ssh-action@v1.0.3 # 사용할 라이브러리 명시
        with:
          host: ${{ secrets.EC2_DEV_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_DEV_PRIVATE_KEY }}

          script: |
            rm -rf ~/niornear/current
            mkdir ~/niornear/current
            mv ~/niornear/tobe/project.jar ~/niornear/current/project.jar
            cd ~/niornear/current
            sudo fuser -k -n tcp 443 || true
            sudo nohup java -jar project.jar > ../output.log 2>&1 &
            rm -rf /home/ubuntu/niornear/tobe
```

