---
layout: single
title:  "[니어니어 프로젝트] AWS EBS 정리"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# EBS (Elastic Block Storage)

- EBS 는 클라우드에서 사용하는 가상 하드디스크(HDD) 라고 말할 수 있다.
- EBS 는 AWS 클라우드의 Amazone EC2 인스턴스에 사용할 **영구 블록 스토리지 볼륨**을 제공
- 프로비저닝(빌리는 행위)한 부분에 대해서만 저렴한 비용을 지불할 수 있다.

## EBS ↔ EC2 연결 특징

- EC2 인스턴스가 종료되어도 별개로 작동하여 유지가 가능하다는 점
- 컴퓨터 본체가 꺼지면 하드디스크를 사용하지 못하지만, EBS 는 네트워크로 연결된 별개의 서비스이기 때문에 가능한 것
- 저장 장치 기능만 필요할 때는 EBS 는 독립적으로 살아있기 때문에 스토리지 기능만 이용하는데, 인스턴스의 추가 요금을 내지 않아도 된다.
- 이렇게 컴퓨팅을 구성하면 장점이 있다.
    1. 네트워크로 연결된 인스턴스와 EBS 는 단순히 인스턴스만 다른 걸로 바뀌어 껴주면 언제든 인스턴스의 성능 등을 바꿀 수 있다.(재연결)
    2. 인스턴스 입장에서도 여러 개 EBS 붙일 수 있다.
- EBS 는 EC2 와 같은 가용영역(AZ) 에 존재
    - AZ 가 같아야 연결 및 통신이 빠르기 때문
    - 다른 AZ 로 생성해서 EC2 에 붙이려고 하면 에러가 난다.

## EBS 볼륨(Volume) 이란?

- EBS 로 생성한 디스크 하나하나 저장 단위를 말한다.
- EBS 볼륨을 인스턴스에 연결하다는 말 = EC2 에 물리적 하드 드라이브처럼 사용하겠다는 뜻

### EBS 볼륨 유형 타입

- EBS 는 총 5가지 타입 제공
    1. 범용 : SSD
    2. 프로비저닝 된 IOPS : SSD
    3. 쓰루풋 최적화
    4. 콜드 HDD
    5. 마그네틱

![image-20241107225410171]({{site.url}}/images/2024-08-14-niornear_project(10)/image-20241107225410171.png)

- 하드의 성능은 용량과 MAX IOPS(Input/Output Per Second) 를 보면 된다

## EBS 와 Instance Storage

- EC2 인스턴스 저장 타입은 대표적으로 두가지 존재
    1. EBS 기반
    2. 인스턴스 저장 기반
    

![image-20241107225416239]({{site.url}}/images/2024-08-14-niornear_project(10)/image-20241107225416239.png)

### EBS 기반

- EC2 가 EBS 와 네트워크로 연결
- 그래서 속도가 느림
- 대신 인스턴스 삭제되더라도 EBS 는 남아있음(영구적 스토리지)
- 하나의 인스턴스에 연결한 EBS 볼륨을 따로 분리해서 다른 인스턴스에 연결 가능(USB 처럼)

### 인스턴스 저장 기반(Instance Storage)

- EC2 안에 storage 가 들어있음(네트워크 연결 X)
- 그래서 속도가 빠름
- 안에 들어있는 형태, 인스턴스가 삭제되면 storage 도 같이 삭제
- EBS 처럼 스토어 분리해서 다른 인스턴스에 연결 불가능
- 보통 영구적이지 않는 데이터를 저장
  
    ex) 캐시 데이터
    

출처 : 

[[AWS] 📚 EBS 개념 & 사용법 💯 정리 (EBS Volume 추가하기)](https://inpa.tistory.com/entry/AWS-📚-EBS-개념-사용법-💯-정리-EBS-Volume-추가하기)