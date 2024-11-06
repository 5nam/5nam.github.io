---
layout: single
title:  "[니어니어 프로젝트] 메뉴 등록하기 기능 구현"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 메뉴 등록하기 트러블 슈팅

문제 1) @ColumnDefault 어노테이션 작동 안함

[[Trouble-Shooting] Spring JPA @ColumnDefault 오류](https://jaehee1007.tistory.com/16)

문제 2) Handler 작동 안함 문제

- MasterExcpetionHandler 를 적용 안해서 발생한 문제. 적용하니 사라짐

문제 3) Type definition error

- 직렬화 오류 → DTO 를 직렬화할 때 Getter 를 사용하기 때문에
- @Getter 가 없어서 발생한 문제

[DTO 직렬화 에러 / DTO와 @Getter 개념/Type definition error: --No serializer found for class  --and no properties discovered to create BeanSerializer](https://velog.io/@won0104/DTO-직렬화-에러-DTO와-Getter-개념Type-definition-error-No-serializer-found-for-class-and-no-properties-discovered-to-create-BeanSerializer)

피드백 1) 추후에 변경해야 할 코드의 주석은 TODO, FIXME 이런 식으로 달아주기

피드백 2) null 값이 들어가지 않는 데이터의 경우 참조 타입(Long, Integer) 보다는 원시 타입(long, int)으로 사용하기

- 원시 타입은 값을 직접 저장하기 때문에 성능이 더 좋다.

[[Java] Long과 long의 차이](https://bluemoon-clover.tistory.com/129)

피드백 3) 검증 로직은 필수다.

- 메뉴 등록하기 기능을 만들 때, store 의 요리사인지 따로 검증하지 않았는데 검증하는 편이 훨씬 더 확실하다!