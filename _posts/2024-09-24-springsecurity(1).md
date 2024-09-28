---
layout: single
title:  "[SPRING SECURITY] SecurityBuilder, SecurityConfigurer"
published : false
---

## 개념 및 구조 이해

**SecurityBuilder**

- 빌더 클래스로서 웹 보안을 구성하는 빈 객체와 설정 클래스(초기화 관련)들을 생성하는 역할
- 종류 : WebSecurity, HttpSecurity

![image-20240924163821231](/Users/onam-ui/Library/Application Support/typora-user-images/image-20240924163821231.png)

<img src="/Users/onam-ui/Library/Application Support/typora-user-images/image-20240924164004869.png" alt="image-20240924164004869" style="zoom: 67%;" />

- build 메서드 하나 존재

**SecurityConfigurer**

- Http 요청과 관련된 보안처리 담당하는 **필터들을 생성하고 초기화 설정에 관여**

![image-20240924163831193](/Users/onam-ui/Library/Application Support/typora-user-images/image-20240924163831193.png)

<img src="/Users/onam-ui/Library/Application Support/typora-user-images/image-20240924164152569.png" alt="image-20240924164152569" style="zoom:50%;" />

- init 과 configure 메서드 2개 존재

**SecurityBuilder 와 SecurityConfigurer**

- SecurityBuilder 는 SecurityConfigurer 를 포함
- 인증 및 인가 초기화 작업은 SecurityConfigurer 에 의해 진행
- 실제로 초기화 작업이 진행되면 자동 설정에서
  - SecurityBuilder 클래스의 build 메서드 실행
  - build 메섣 실행되는 과정 속에서 설정 클래스 SecurityConfigure의 init 과 configure 메서드가 호출
  - init 과 configure 메서드 안에서 실제로 초기화 설정 작업 진행

**시퀀스(간단)**

-- auto-configuration 시작 --

1. 빌더 클래스 생성 : 자동 설정(auto-configuration)에 의해서 SecurityBuilder의 `build()` 메서드 호출
2. SecurityBuilder 가 설정 클래스(SecurityConfigurer) 생성
3. 초기화 작업 진행(SecurityConfigurer 내의 init, configure 을 통해)
   - `init(B builder)`, `configure(B builder)`

결국은, SecurityBuilder 클래스가 SecurityConfgiure 의 init 과 configure 메서드를 호출해 주면서 초기화 작업을 진행

![image-20240924190717656](/Users/onam-ui/Library/Application Support/typora-user-images/image-20240924190717656.png)

- FilterChainProxy 와 SecurityFilterChain 은 참조 관계이다.
- FilterChainProxy 는 WebSecurity 의 build 가 진행된 결과값이고
- SecurityFilterChain 은 HttpSecurity 의 build 가 진행된 결과값이다.
