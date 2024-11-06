---
layout: single
title:  "[니어니어 프로젝트] 요리사 신청하기 기능 구현"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 요리사 신청하기(개인 사업자) 트러블 슈팅

@NoArgsConstructor 가 Builder 를 위한 매개변수가 존재하는 생성자를 만들 때마다 넣어줘야 함.

이유는?

오류 1) Unique index or primary key violation

- DB 에 테스트 데이터를 넣을 때 id 값을 지정해서 넣어주니까, 충돌이 일어남.
- id 를 직접 지정하지 않고, default 값으로 넣어주니 해결됨

[https://velog.io/@galmegi/Repository테스트-Unique-index-or-primary-key-violation-오류](https://velog.io/@galmegi/Repository%ED%85%8C%EC%8A%A4%ED%8A%B8-Unique-index-or-primary-key-violation-%EC%98%A4%EB%A5%98)