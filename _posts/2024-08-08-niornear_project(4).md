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

# 요리사 신청하기 기능 설명

- 요리사는 니어 컴퍼니 소속과 개인 소속으로 나뉜다.
- 니어 컴퍼니 소속 요리사 신청과 개인 소속 요리사 신청은 인증 부분에 다른 컬럼이 들어가기 때문에 저장할 때 구분이 필요하다.

# 요리사 신청하기(니어 컴퍼니 소속) 트러블 슈팅

오류 1) ModelAttribute 적용 안됨

- ModelAttribute 는 해당 RequestDto 를 바인딩하기 위해서는 @Setter 가 필요

오류 2) No serializer found for class → @Getter 가 필요

- serializer 하는 과정에서 기본으로 접근 제한자가 public 이거나 getter/setter를 이용하기 때문에 인스턴스 필드를 private 등으로 선언하면 json으로 변환 과정에서 에러가 발생

출처 : [https://sharonprogress.tistory.com/273](https://sharonprogress.tistory.com/273)