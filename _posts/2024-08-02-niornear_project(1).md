---
layout: single
title:  "[니어니어 프로젝트] DB 설계(1차)"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 객체설계

## 필요한 객체

- 사용자
- 요리사
- 주문
- 편지
- 계좌

## 상황

#1 주문하기

사용자가 주문 객체에게 음식을 요청한다.

주문 객체는 사용자가 주문한 음식에 대한 주문서를 응답한다.

사용자는 주문한 음식의 주문서에 정보를 기입하고, 계좌에 결제하기를 요청한다.

계좌 객체는 해당 주문서의 가격을 결제할 수 있는 확인한 후, 결제 완료를 사용자에게 응답한다.

사용자 객체는 결제 완료를 확인하고, 주문 객체에게 이를 주문 완료하기로 요청한다.

주문 객체는 사용자의 요청을 받아 요리사에게 전달한 후, 주문 완료를 사용자에게 응답한다.

#1-1 주문 완료하기

요리사는 주문을 접수하여 음식을 한 뒤, 주문 객체에게 픽업 대기를 요청한다.

주문 객체는 주문의 상태를 픽업 대기 상태로 변경한 뒤, 사용자에게 전달한다.

#2

사용자는 편지 객체에게 편지 보내기를 요청한다.

편지 객체는 요리사에게 생성한 편지를 전달한다.

#3

사용자는 편지 객체에게 조회하기를 요청한다.

편지 객체는 편지 정보를 응답한다.

#4

사용자는 계좌 객체에게 포인트 충전을 요청한다.

계좌 객체는 포인트 충전 후(API 요청을 통해 진행, 여기서는 임의로 진행)에 완료된 내역을 응답한다.

#5

사용자는 주문 객체에게 주문 내역 조회를 요청한다.

주문 객체는 주문 내역을 응답한다.

## 객체 별 역할

### 사용자

- 사용자는 주문에게 음식 요청을 한다
- 사용자는 계좌에게 결제 요청을 한다
- 사용자는 편지에게 편지 보내기 요청을 한다
- 사용자는 편지에게 편지 조회 요청을 한다.
- 사용자는 계좌 객체에게 포인트 충전을 요청한다.
- 사용자는 주문 객체에게 주문 내역 조회를 요청한다.

### 주문

- 주문은 주문서를 생성한다.
- 주문은 주문 내역을 응답한다.
- 주문은 주문서를 요리사에게 전달한다.
- 주문 객체는 주문 상태를 바꾼다.

### 편지

- 편지 객체는 요리사에게 편지를 전달한다.
- 편지는 편지를 생성한다.
- 편지 객체는 편지 정보를 응답한다.

### 계좌

- 계좌는 포인트 충전을 한다.
- 계좌는 결제를 한다.

### 요리사

- 요리사는 음식을 준비한다.

## 객체 별 상태

### 사용자

- 닉네임
- 이메일
- 전화번호
- 지역

### 주문

- 주문서(메뉴, 메뉴 수량, 총 가격)
- 주문 상태
- 요리사
- 사용자

### 편지

- 편지 사진
- 편지 받는 사람
- 편지 보내는 사람

### 계좌

- 포인트
- 예금주(사용자 or 요리사)

### 요리사

- 이름
- 메뉴판
- 온도

# DB 설계

## DB Entity

- 빨간배경은 외래키이자 기본키(식별자 관계) / 파란 배경은 왜래키(비식별자 관계)

### User

- id
- name
- profile_image
- email
- phone
- point([https://heavening.tistory.com/85](https://heavening.tistory.com/85))
- status(ENUM : 회원 상태, 가입/탈퇴/휴면 등)
- create_at
- update_at
- region_id(FK)

### Order

- id
- total_price
- request_message
- phone(픽업 전화번호)
- status(주문 상태 Enum : 주문접수 - 상품제작 - 픽업)
- create_at
- update_at
- place_id(픽업 장소)
- user_id(주문자)
- chef_id(요리사)

### Letter

<aside>
📍 편지를 보낼 때 user 에게 받은 텍스트를 이미지로 변환해서(손편지 글씨체를 사용해서 손편지 느낌나도록) 셰프에게 문자로 전송하는 기능으로 하시는 건 어떤가요?
</aside>

- id
- image_link
- create_at

### Account

<aside>
📍 사용자에게도 요리사에게도 계좌는 필요할듯
</aside>

- id
- bank_name
- account_number
- balance
- create_at

### Chef

- id
- name
- profile_image(link)
- thumbnail(link)
- title
- introduction
- point(구매자가 구매하면 여기 포인트에 입금)
- temperature(DECIMAL : [지정 소수점 타입](https://velog.io/@dnjscksdn98/Database-%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%83%80%EC%9E%85))
- message(주문 완료 후 메시지)
- certification(Enum : 경력 5년 이상, 니어셰프 등)
- 최저가
- create_at
- update_at

### Menu

- id
- name
- image_link
- price
- introduction
- chef_id(요리사)

### Region(계층형) - 사용자가 설정할 수 있는 지역

- id
- name
- 상위지역(서울, 부산 등등 시 단위)

### Place - 픽업 장소(니어니어에서 제공하는)

- id
- name
- address

### ChefRegion(지역별로 셰프를 조회하기 위해, 셰프별로 가능한 지역을 조회하기 위해) -  다대다

- chef_id
- region_id

### PossiblePickUp - 지역별로 픽업 가능한 장소를 조회하기 위해 - 다대다

<aside>
📍 한 지역은 여러 픽업 장소를 가질 수 있고, 한 픽업 장소는 여러 지역에 속할 수 있다고 생각해서 이렇게 함.
  의논이 필요한 지점. 만약 한 픽업 장소는 한 지역에만 속할 수 있다면 Place 에 region_id 만 추가하면 됨</aside>

- region_id
- place_id

### LikeChef(하트 누른 셰프) - 다대다

- user_id
- chef_id

### OrderMenu(주문 별로 주문한 메뉴를 저장하기 위해) - 다대다

- order_id
- menu_id
- 수량

### UserLetterBox - 다대다

<aside>
📍 송신자와 수신자를 편지 객체에 두지 않는 이유는, 편지는 사용자도 보낼 수 있고 요리사도 보낼 수 있기 때문에 id 를 받고, 조회할 때 user 에서 조회해야 하는지 chef 에서 조회해야하는지 알아야 하기 때문에
</aside>

- user_id(receiver)
- letter_id
- chef_id(sender)

### ChefLetterBox → 조회할 때는 필요 없지만,, 혹시나 필요한 데이터일 수도 있으므로..

<aside>
📍 송신자와 수신자를 편지 객체에 두지 않는 이유는, 편지는 사용자도 보낼 수 있고 요리사도 보낼 수 있기 때문에 id 를 받고, 조회할 때 user 에서 조회해야 하는지 chef 에서 조회해야하는지 알아야 하기 때문에
</aside>

- chef_id(receiver)
- letter_id
- user_id(sender)

### UserAccount - 다대다

- user_id
- account_id

### ChefAccount(셰프도 입금받을 계좌를 여러 개 설정할 수 있다고 가정) - 다대다

- chef_id
- account_id

![Untitled]({{site.url}}/images/2024-08-02-niornear_project(1)/Untitled.png)

![Untitled 1]({{site.url}}/images/2024-08-02-niornear_project(1)/Untitled 1.png)

![Untitled 2]({{site.url}}/images/2024-08-02-niornear_project(1)/Untitled 2.png)

- 사용자에게 권한 부여하고, 그 다음 게시물 테이블 만들어서 일대일 매핑하는게 좋을듯
- 사용자 테이블에 회원 권한(회원, 비회원, 요리사, 관리자)