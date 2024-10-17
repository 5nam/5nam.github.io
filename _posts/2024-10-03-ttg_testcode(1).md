---
layout: single
title:  "[또또가 프로젝트] 테스트 코드 작성 "
published : false
categories: ttottoga_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

## 리포지토리 테스트 코드 작성

- @Sql 어노테이션 활용하면 다른 리포지토리를 선언할 필요 없어짐

- 리포지토리 테스트 코드 작성하면서, findByMember 등 객체로 선언하면 테스트 코드에 MemberRepository 를 추가적으로 선언해줘야 한다는 점에서 불편함을 겪음.
- 그래서 리포지토리 관련 코드는 id 로 찾겠끔 findByMemberId 이런 식으로 적용해주는 것이 좋겠다는 생각을 함

- 테스트 코드 다 작성한 후 수정할 것. 그래야 수정 후에 틀린지 안 틀린지를 테스트 코드로 신속하게 알아볼 수 있기 때문
