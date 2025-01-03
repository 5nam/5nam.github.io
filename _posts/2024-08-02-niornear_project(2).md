---
title:  "[니어니어 프로젝트] DB 설계(2차)"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# DB 설계 보완 및 완성

![Untitled]({{site.url}}/assets/images/2024-08-02-niornear_project(2)/Untitled.png)

## 보완한 점

1. 요리사와 사용자로 나눠서 관리하는 것이 아니라, 사용자는 사용자대로 관리하되 사용자 중에 요리사 신청하기를 통해 요리사가 되는 사람들을 Store 테이블에 따로 관리하도록 설계 변경
    - User 테이블에 회원 권한 컬럼 추가
        - 회원, 비회원, 요리사, 관리자
2. 요리사가 여러 인증을 가질 수 있으므로 다대다 매핑
3. 요리사의 요리 이미지도 관리하기