---
layout: single
title:  "[니어니어 프로젝트] 요리사 조회하기 기능 구현"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 요리사(상점) 조회하기 트러블 슈팅

오류 1) scale has no meaning for SQL floating point types

`@Column(nullable = false, precision = 10, scale = 1)`

- double, 원시 타입으로 변경하여 위의 어노테이션이 적용될 수 없었던 것

피드백 1) ResponseDto 에 접근 제어자 지정해주기

- 사실 객체지향을 배우면서 접근 제어자 지정에 대해 setter, getter 가 있으면 무슨 소용일까 라는 생각을 했다.
- 그러나 아래 게시물을 보고 왜 있어야 하는지 알았다.
- 은닉 개념에서 훨씬 좋다. 협업을 할 때, private 으로 지정해놓은 것들은 직접 접근해서 사용할 필요가 없음을 명시하는 등의 역할을 한다는 것을 알게 됨

[OKKY - 자바 객체지향에 대해 공부중인데 접근제어자 private가 꼭 필요한가요?](https://okky.kr/questions/1416385)