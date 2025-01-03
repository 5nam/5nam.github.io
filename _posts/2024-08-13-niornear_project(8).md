---
title:  "[니어니어 프로젝트] AWS RDS 정리 및 연동"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# RDS 구성 아키텍처

- AWS RDS 는 DB 를 보다 효율적으로 관리할 수 있게 하는 인프라 아키텍쳐 방법 3가지 제공

## 1. RDS Multi AZ 구조

![image-20241107224603056]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224603056.png)

- 두 개 이상의 AZ(가용영역)에 걸쳐 데이터베이스를 구축
- 원본과 다른 DB(StandBy[임시])를 자동으로 동기화하는 구조
- 이 구조는 AWS 에 의해서 자동으로 관리가 이루어짐
- 현재 사용 중인 DB 에 문제가 생기면 RDS 는 즉시 다른 AZ 에 만들어진 복제본을 원본 DB 로 승격시켜 그대로 사용함 → Disaster Recovery(재해복구)

### Multi AZ 재해 복구 원리

- DNS 주소로 RDS Primary 인스턴스에 접속하여 사용
- 기본적으로 Standby DB 는 DNS 가 부여되지 않으므로 접근 불가능
- 만약 RDS Primary 인스턴스에 장애가 발생하면, 다음 과 같이 장애 복구 이루어짐
    1. RDS Primary 에 문제가 생기면, 자동으로 DNS 를 Standby 인스턴스로 연동
    2. 기존에 RDS Primary 와의 연결을 끊음
    3. 이를 fail over(fail 났을 때 복구하는 과정)
    4. EC2 입장에서는 동일한 Address 로 바라보고 있으므로 장애를 느끼지 못함 → 빠른 속도로 장애 복구 가능

### Multi AZ 용도

- 멀티 AZ 구조의 단점
    - 예비용 인스턴스가 하나 더 돌고 있어서 비용이 두 배가 든다는 점
    - Multi AZ 구조는 퍼포먼스의 상승 효과가 아닌 안정성을 위한 서비스
      
        → 성능 개선이 주 목적이라면, Read Replicas 구조를 이용해야함
        

## 2. Read Replica(읽기 전용 복제본)

![image-20241107224617972]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224617972.png)

- Read Replica, RDS Primary 를 복제해서 DB 쓰기는 RDS Primary 에서 처리하고, DB 읽기는 복제한 복제본에서 처리하는 구조
- 쓰기와 읽기를 나눠으면, 서비스에서 DB 읽기 위주의 작업이 많은 경우 Read Replica 를 여러 개 만들어서 부하를 분산할 수 있다.
- RR 의 복제본들은 같은 AZ 에 있을 수도 있고, 다른 AZ 심지어 다른 리전에 존재할 수 있다
- RR 은 멀티 AZ 와는 달리 각각의 복제본 인스턴스에 DNS 가 각각 부여되어서 접근 가능하다.
- 이렇게 DB 를 복제하여 읽기 쓰기의 트래픽을 분산하는 기술 → 데이터베이스 Replication
    - 서버를 이중화하듯 데이터베이스를 이중화 한다고 이해하면 된다.

### 데이터 Replication

- Replication
    - 백업과 성능 향상을 위해서 데이터베이스를 여러 대의 서버에 복제하는 행위
    - 원본 데이터 위치하는 서버 → 마스터
    - 원본을 복제한 서버 → 슬레이브
- 쓰기 작업같은 경우 저장된 데이터가 변경되므로, 만일 복제된 서버에 쓰기 작업을 맡기게 된다면 복제된 서버들 간에 동일한 형태를 유지하는데 많은 비용이 듦
  
    → 그래서 한대의 서버에만 쓰기 작업을 하고, 그 서버의 데이터를 복제해서 여러 대의 슬레이브 서버를 만든 후에 슬레이브에서는 읽기 작업만ㅇ르 수행하게 구성
    

## Read Replica VS Multi-AZ

### 1. 복제의 목적

- Multi-AZ 복제 → 서비스가 항상 가동해야 하는 가용성이 목적
- Read Replica → 퍼포먼스를 위한 서비스

### 2. 복구 방법

- Read Replica → fail over(자동복구) 불가능
    - RDS Primary 고장 → 관리자가 직접 수동으로 쓰기 DNS를 복제본에 연결해 복구

### 3. 복제 데이터 일관성

- Multi-AZ → 쓰기 작업을 실시한 즉시 자동으로 예비 인스턴스에 복제, 데이터가 항상 일치
- Read Replica → 쓰기 작업을 실시하면 약간의 시간차를 가지고 데이터가 복제, 데이터가 일치하지 않을 수 있다.

> S3 에서의 PUT 동작과 UPDATE/DELETE 동작간의 데이터 일관성, 최종 일관성과 같은 원리
> 

## 3. RDS Multi Region

- 다른 리전에 지속적으로 동기화 시키는 DB 클러스터 생성 (Async 복제)
- 예시
    - 유럽 국가에 서비스를 한다고 가정
    - 서울에 마스터 리전을 두고 유럽에 Read Replica 를 두면 유럽 현지에서 RR 로 읽기 때문에 latency(지연시간)이 줄어든다
    - 우리나라에서는 읽기 전용 복제본에 데이터 복제만 해주면 된다.
- RDS 멀티 리전은 주로 로컬 퍼포먼스 또는 DR(disaster recovery) 시나리오로 활용
    - 로컬 퍼포먼스 : 그 지역에 있는 사람들이 빨리 읽을 수 있게 해줌
    - DR 시나리오 : 만약 서울에 폭탄이 터져서 리전이 fail 된다면 다른 리전을 마스터로 만드는 구성
- 각 리전별로 자동 백업 가능
- 리전별로 Multi-AZ 가능

출처 :

[[AWS] 📚 RDS 개념 & 아키텍쳐 정리 [이론편]](https://inpa.tistory.com/entry/AWS-📚-RDS-개념-아키텍쳐-정리-이론편)

# RDS 생성 및 연동

1. 파라미터 그룹 생성 및 설정
   
    ![image-20241107225002376]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107225002376.png)
    
    ![image-20241107224723341]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224723341.png)
    
    ![image-20241107224733466]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224733466.png)
    
    ![image-20241107224741334]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224741334.png)
    
2. RDS 생성
   
    ![image-20241107224817458]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224817458.png)
    
    ![image-20241107224823054]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224823054.png)
    
    ![image-20241107224643878]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224643878.png)
    
    ![image-20241107224831293]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224831293.png)
    
    ![image-20241107224656876]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224656876.png)
    
    ![image-20241107224838795]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224838795.png)
    
    ![image-20241107224848445]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224848445.png)
    
    ![image-20241107224708021]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224708021.png)
    
    ![image-20241107224858302]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224858302.png)
    
    ![image-20241107224905176]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224905176.png)
    
3. 보안 그룹 설정
   
    **초기상태**
    
    ![image-20241107224919018]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224919018.png)
    
    **RDS > 내가 생성한 RDS 인스턴스 > 보안 > VPC 보안 그룹 선택**
    
    ![image-20241107224755913]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224755913.png)
    
    ![image-20241107224926701]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224926701.png)
    
4. DataGrip 으로 외부접속 해보기
   
    ![image-20241107224934866]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224934866.png)
    
5. SpringBoot 연동
   
    **build.gradle**
    
    ![image-20241107224942595]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224942595.png)
    
    **.gitignore**
    
    ![image-20241107224950115]({{site.url}}/assets/images/2024-08-13-niornear_project(8)/image-20241107224950115.png)
    
    **application-rds.properties**
    
    ```bash
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.url=jdbc:mysql://{RDS 엔드포인트}:{3306과 같은 포트번호}/{설정한 초기 데이터베이스 이름}
    spring.datasource.username={RDS 에서 설정한 username} 
    spring.datasource.password={RDS 에서 설정한 password}
    
    spring.jpa.hibernate.ddl-auto=create
    spring.jpa.properties.hibernate.format_sql=true
    
    logging.level.org.hibernate.SQL=debug
    logging.level.org.hibernate.type.descriptor.sql=trace
    ```
    
    **실행**
    
6. DataGrip 에서 확인
   
    `show databases;` 로 생성되어 있는 데이터베이스들 조회
    
    `use {설정한 초기 데이터베이스 이름};` 로 데이터베이스 사용 설정
    
    `show tables;` 로 테이블들 잘 생성되었는지 조회
    

## 실제 운영 목적일 경우 RDS

[[AWS] 📚 RDS 실전 구축하기 [Multi AZ / Read Replica]](https://inpa.tistory.com/entry/AWS-📚-RDS-실전-사용하기-Multi-AZ-Read-Replica)

- 내 개인프로젝트에 적용해보면 좋을듯 ㅎㅎ(유료니까..)