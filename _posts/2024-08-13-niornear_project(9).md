---
title:  "[니어니어 프로젝트] AWS EC2 정리 및 배포"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# EC2(Elastic Compute Cloud)

## EC2 개념

- EC2 는 AWS 에서 제공하는 클라우드 컴퓨팅
- 독립된 컴퓨터를 임대해주는 서비스
- EC2 는 컴퓨터를 주문하면 바로 1분 안에 생성, 삭제 될 수 있다.
    - 초기 구입비, 세팅비가 전혀 없고, 사용한 만큼 비용을 지불하면 된다.
- EC2 는 복잡한 공유기 세팅없이 인터넷을 통해서 자유롭게 접속할 수 있고, 이미지(AMI) 기능도 사용할 수 있다.
    - 컴퓨터를 사용하려면 프로그램, 파일, 설정 등 세팅해야 할 게 많은데 이 OS 상태 그대로 저장하는 기능을 이미지(AMI) 라고 한다.
    - 이미지를 이용해서 새로운 컴퓨터를 만들면 이미지에 저장된 상태와 똑같은 컴퓨터를 빠르게 생성할 수 있다.

## EC2 특징 요약

- 컴퓨팅 요구사항의 변화에 따라 컴퓨팅 파워 조정 가능
- 실제 사용한 용량 만큼만 지불
- Linux, Windows 중 OS 선택이 자유로움
- 몇 분이면 전 세계에 컴퓨터 수백여대 생성 가능
- 머신러닝, 웹 서버, 게임 서버, 이미지 처리 등 다양한 용도에 최적화된 서버 쉽게 구성 가능
- 여러 다른 AWS 서비스와의 유기적인 연동 가능

## EC2 구성 (인스턴스 / EBS / AMI)

- 컴퓨팅에 해당하는 인스턴스
- 하드디스크에 해당하는 EBS
- 랜카드에 해당하는 ENI

### EC2 인스턴스 이해하기

- AWS 클라우드에서 사용하는 가상 컴퓨터
- CPU, 메모리, 그래픽카드 등 연산을 위한 하드웨어 부분을 담당
- 다양한 인스턴스 유형을 두어, 컴퓨터 사용 용도에 맞게 컴퓨팅을 구성할 수 있다.

즉, 한정된 요금으로 EC2 인스턴스의 유형을 고르고 사이즈를 골라 각 인스턴스 별로 사용 목적에 따라 최적화 시키는 것

**인스턴스 타입**

- AWS 는 각 인스턴스의 사용 목적(서버용, 머신러닝용, 게임용)에 따라 타입별로 인스턴스에 이름을 부여해 구분
    - t, m 타입은 범용타입이기 때문에 AWS 초보자들이 가장 많이 사용하는 프리티어에서 쓰는 타입이므로 자주 접할 수 있는 타입

**인스턴스 사이즈**

![image-20241107225139836]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225139836.png)

- 인스턴스의 크기는 cpu 개수, 메모리 크기, 성능 등으로 사이즈가 결정됨을 말한다
- 즉 인스턴스 사이즈가 클수록 더 많은 메모리, cpu, 네트워크 대역폭을 가질 수 있음을 의미

**인스턴스 타입 읽는 법**

![image-20241107225146923]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225146923.png)

1. m 은 인스턴스 타입(패밀리) → 범용 애플리케이션 서버용을 의미
2. 5는 5세대를 의미 → 계속해서 세대가 업그레이드 될 것
3. a는 amd 기반의 CPU 프로세서 사용한다는 의미 → t4g = t4 인스턴스 중 AWS Graviton 프로세서 사용
4. xlarge 는 큰 사이즈를 의미한다고 보면 됨

### EBS(Elastic Bloc Storage) 이해하기

- 데이터를 저장하는 역할, 클라우드에서 사용하는 가상 하드디스크(HDD) 인 셈
- EBS 는 AWS 클라우드의 Amazone EC2 인스턴스에 사용할 영구 블록 스토리지 볼륨을 제공
    - 단 몇 분 내에 사용량을 많게 또는 적게 확장할 수 있으며, 프로비저닝(빌리는 행위)한 부분에 대해서만 저렴한 비용을 지불 가능

**EBS 볼륨 유형 타입**

![image-20241107225156029]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225156029.png)

- 총 5가지 타입을 제공
    1. 범용 : SDD
    2. 프로비저닝된 IOPS : SSD
    3. 쓰루풋 최적화(Troughput Optimized HDD or st 1)
    4. 콜드 HDD(SC1)
    5. 마그네틱(Standard)

### AMI(Amazon Machine Image) 이해하기

- AMI 는 EC2 인스턴스를 실행하기 위한 정보를 모은 단위
- EC2를 실행하기 위해서는 CPU 프로세서 타입, 저장공간 용량, 32비트/64비트 여부, OS 종류, 소프트웨어 등등 세팅 정보가 필요한데, 이런 세팅 정보(템플릿)를 저장한 단위 ⇒ AMI
- AMI 를 사용하여 현재 상태의 EC2 세팅(템플릿)을 복제해서 다른 계정이나 다른 리전에게 전달도 가능

출처 :

[[AWS] 📚 EC2 개념 원리 & 사용 세팅 💯 총정리 (Instance / EBS / AMI)](https://inpa.tistory.com/entry/AWS-📚-EC2-개념-사용-구축-세팅-💯-정리-인스턴스-EBS-AMI#ec2_구축_세팅__사용하기)

# EC2 생성하기

![image-20241107225206689]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225206689.png)

![image-20241107225216702]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225216702.png)

![image-20241107225224432]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225224432.png)

![image-20241107225231625]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225231625.png)

# EC2 보안 그룹 설정

**EC2 인스턴스 > 보안 > 보안그룹 선택하면 아래와 같이 나옴**

프론트와 통신한다면 포트번호 3000번도 열어주자!

![image-20241107225238891]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225238891.png)

**인바운드 규칙 편집 선택**

![image-20241107225246813]({{site.url}}/assets/images/2024-08-13-niornear_project(9)/image-20241107225246813.png)

# EC2 에 프로젝트 배포

## 1️⃣ 깃 설치하기

```bash
sudo apt-get install git
git --version
```

## 2️⃣ git clone 하기

```bash
git clone "리포지토리 주소"
```

## 3️⃣ Java 설치

- 프로젝트 버전에 맞는 JAVA 설치

```bash
sudo apt install openjdk-17-jdk
```

## 4️⃣ 필요한 파일 EC2 서버에 업로드

- 필요하지만, 보안 때문에 올리지 못한 properties 등등 scp 명령어로 업로드
- 로컬 터미널에서 진행

```bash
scp -i [pem파일경로] [업로드할 파일 이름] [ec2-user계정명]@[ec2 instance의 public DNS]:[EC2 복사할 경로]
```

```java
scp -i /Users/onam-ui/Desktop/Project/niornear.pem /Users/onam-ui/Desktop/Project/2024/nior-near/BE/server/src/main/resources/application-iamport.properties ubuntu@54.180.155.131:/home/ubuntu/BE/server/src/main/resources
```

- EC2 에서 복사할 폴더 경로 알고 싶으면 `pwd` 명령어로!

## 5️⃣ 빌드

- 빌드 전에 프로젝트 폴더에 반영할 사항있으면, `git pull origin <branch name>` 로 반영하기
- 프로젝트 폴더 내에 gradlew 파일 있는 위치에서 해당 명령어 실행

```bash
./gradlew clean build
```

### 잠깐! 빌드하는데 너무 오래 걸린다..! 그럼 스왑을 이용해서 ram 을 늘려보자!

```bash
sudo dd if=/dev/zero of=/swapfile bs=128M count=16
```

- AWS 권장 스왑 용량 최대 : 2GB
- 우리는 RDS 를 사용하므로, 하드디스크 용량이 크게 중요하지 않으므로 스왑해서 사용하자

[[AWS] 당신의 EC2 인스턴스가 계속해서 죽는다면?(feat. Swap Memory)](https://velog.io/@pgmjun/AWS-당신의-EC2가-계속해서-죽는다면feat.-Swap-Memory)

[AWS EC2 램 늘리기(feat. 스왑)](https://velog.io/@shawnhansh/AWS-EC2-메모리-스왑)

## 6️⃣ 빌드 후, 빌드 파일(jar 파일) 실행

```bash
cd build/libs
```

```bash
nohup java -jar {jar 파일 이름} &
```

- & 백그라운드에서 실행한다는 의미 → ec2 콘솔 접속이 끊겨도 계속 실행 파일이 돌아감
- 실행하던 거 다시 종료하고, 재배포 해야 한다면 아래 코드대로 하고 다시 5번 부터!
  
    ```bash
    sudo fuser -k -n tcp 8080
    # 8080 포트로 실행되고 있는 프로그램 종료하라는 명령어
    ```
    
    ```bash
    lsof -i :포트번호
    ```
    

출처 : 

[SpringBoot 프로젝트 EC2 배포하기](https://velog.io/@jonghyun3668/SpringBoot-프로젝트-EC2-배포하기)