---
layout: single
title:  "[니어니어 프로젝트] CORS 오류 정리"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# CORS 오류

# CORS 오류 테스트하는 명령어

- 로컬 터미널에서 실행

```bash
curl --verbose --request OPTIONS \
'{백엔드 요청 주소}' \
--header 'Origin: {프론트엔드 주소}' \
--header 'Access-Control-Request-Headers: Origin, Accept, Content-Type' \
--header 'Access-Control-Request-Method: {요청 HTTP 메서드}'
```

```bash
curl --verbose --request OPTIONS \
'http://54.180.155.131:8080/stores/near-company/2' \
--header 'Origin: http://localhost:3000' \
--header 'Access-Control-Request-Headers: Origin, Accept, Content-Type' \
--header 'Access-Control-Request-Method: GET'
```

# CORS 에러 메세지

- 웹 브라우저는 HTTP 요청에 대해 어떤 요청을 하느냐에 따라 다른 특징을 가지기 때문에 일어난다.

## 요청 방식에 따라 다른 CORS 발생 여부

### 1. \<img\>, \<video\>, \<script\>,\<link\> 태그 등

- 기본적으로 Cross-Origin 정책 지원
    - link 태그의 href 에서 다른 사이트의 .css 리소스에 접근하는 것 가능
    - img 태그의 src 에서 다른 사이트의 .png, .jpg 등의 리소스에 접근하는 것이 가능
    - script 태그의 src 에서 다른 사이트의 .js 리소스에 접근하는 것 가능(type=”module” 속성은 제외)

### 2. XMLHttpRequest, Fetch API 스크립트

- 기본적으로 Same-Origin 정책 따름
    - 다른 도메인의 소스에 대해 자바스크립트 ajax 요청 API 호출시
    - 웹 폰트 CSS 파일 내 @font-face 에서 다른 도메인의 폰트 사용 시
- 자바스크립트에서의 요청은 기본적으로 서로 다른 도메인에 대한 요청을 보안상 제한
    - 브라우저는 기본으로 하나의 서버 연결만 허용되도록 설정되어 있기 때문(주로 자신의 서버)

## CORS 에러 이해하기

- CORS = Cross-Origin Resource Sharing(교차 출처 리소스 공유 정책)
    - 교차 출처 → (엇갈린) 다른 출처

![image-20241109211528014]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211528014.png)

- 위의 메시지를 이해하기 위해 차근차근 개념을 살펴보자

### 출처(Origin) 란?

- URL 은 아래와 같은 구성 요소로 이루어져 있다.
  
    ![image-20241109211544310]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211544310.png)
    
- CORS 를 이해하는데 있어 Origin 부분, Protocol + Host + Port 를 알면 된다.
- 출처(Origin) = Protocol + Host + Port 까지의 URL

### 동일 출처 정책(Same-Origin Policy)

- 동일한 출처에서만 리소스를 공유할 수 있다. ⇒ 이 룰을 가진 정책
- 동일 출처 서버에 있는 리소스는 자유로이 가져올 수 있지만, 다른 출처(Cross-Origin) 서버에 있는 이미지나 유튜브 영상 같은 리소스는 상호작용 불가능

![image-20241109211703552]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211703552.png)

**동일 출처 정책(SOP)이 필요한 이유**

- 동일 출처가 아닌 접근을 차단하는 이유는?
    - 출처가 다른 두 어플리케이션이 자유로이 소통할 수 있는 환경은 매우 위험한 환경
    - 해커가 CSRF(Cross-Site Request Forgery) 나 XSS(Cross-Site Scripting) 등의 방법을 이용해서 우리가 만든 어플리케이션에서 해커가 심어놓은 코드가 실행되어 개인 정보 유출의 위험이 있다.
    - 시나리오 예시
        1. 사용자가 악성 사이트에 접속
        2. 이때 해커가 몰래 심어놓은 악의적인 자바스크립트가 실행되어, 사용자가 모르는 사이 어느 포털 사이트에 요청을 보냄
        3. 그럼 포털 사이트에서 해당 브라우저의 쿠키를 이용하여 로그인을 하는 등 상호작용에 따른 개인 정보를 응답 값으로 받은 뒤, 사이트에서 해커 서버로 재차 보냄
        4. 이외에도 사용자가 접속중인 내부망의 아이피와 포트를 가져오거나, 해커가 사용자 브라우저를 프록시처럼 악용할 수도 있음
- 위와 같은 경우를 방지하기 위해, SOP 정책으로 동일하지 않은 다른 출처의 스크립트가 실행되지 않도록 브라우저에서 사전에 방지하는 것

**같은 출처와 다른 출처 구분 기준**

- Protocol, Host, Port 동일하면 동일 출처로 판단
  
    예시
    
    ![image-20241109211558488]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211558488.png)
    

**출처 비교와 차단은 브라우저가 한다.**

- 브라우저가 정책으로 차단을 한다는 말은, 브라우저를 통하지 않고 서버 간에 통신을 할 때는 정책이 적용되지 않는다는 말
    - 즉, 클라이언트 단 코드에서 API 요청을 하는게 아니라, 서버 단 코드에서 다른 출처의 서버로 API 요청을 하면 CORS 에러로부터 자유로워진다.
    - 이를 이용한 프록시(Proxy) 서버라는 것이 있음

**그럼 어떻게 해야 하는가?**

- 몇 가지 예외 조항을 두고 다른 출처의 리소스 요청이라도 이 조항에 해당할 경우 허용하기로 함
- 그게 바로 **CORS 정책을 지킨 리소스 요청!**

## 교차 출처 리소스 공유 (Cross-Origin Resource Sharing)

- 다른 출처의 리소스 공유에 대한 허용 / 비허용 정책

### 우리가 욕했던 CORS 는 사실 해결책이었다.

- SOP 정책을 위반해도 CORS 정책에 따르면 다른 출처의 리소스라도 허용한다는 뜻

### 브라우저의 CORS 기본 동작 살펴보기

1. 클라이언트에서 HTTP 요청의 헤더에 Origin 을 담아 전달
    1. HTTP 프로토콜을 이용하여 서버에 요청을 보내면서, 브라우저는 요청 헤더에 Origin 이라는 필드에 출처를 함께 담아 보냄
2. 서버는 응답헤더에 Access-Control-Allow-Origin 을 담아 클라이언트로 전달
    1. 서버가 클라이언트의 요청에 대한 응답을 할 때, 응답 헤더에 Access-Control-Allow-Origin 필드 추가하고 값으로 ‘이 리소스를 접근하는 것이 허용된 출처 url’을 내려보냄
3. 클라이언트에서 Origin 과 서버가 보내준 Access-Control-Allow-Origin 을 비교
    1. 이후 응답을 받은 브라우저는 자신이 보냈던 요청의 Origin 과 서버가 보내준 응답의 Access-Control-Allow-Origin 을 비교해본 후 차단할지 말지 결정
    2. 유효하지 않다면 그 응답을 사용하지 않고 버린다! (CORS 에러!)
    3. 유효하다면(요청 보내는 주소가 허용되어 있다면) 다른 출처의 리소스를 문제 없이 가져오게 됨

### 결국 CORS 해결책은 서버의 허용이 필요

## CORS 작동 방식 3가지 시나리오

- 실제 CORS 가 동작하는 방식은 세 가지의 시나리오에 따라 변경된다.

### 예비 요청(Preflight Request)

- 브라우저는 요청을 보낼 때 한번에 바로 보내지 않고, 예비 요청을 보내 서버와 잘 통신되는지 확인한 후 본 요청을 보냄
- 예비 요청의 역할 → 브라우저 스스로 안전한 요청인지 미리 확인
- 브라우저가 예비요청을 보내는 것을 Preflight 라고 부르며, 이 예비요청의 HTTP 메소드를 `OPTIONS` 가 사용된다.

![image-20241109211610172]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211610172.png)

- 예비요청 중에 출처(Origin)를 비교한다.

**예비 요청의 문제점과 캐싱**

- 결국은 실제 요청에 걸리는 시간이 늘어나게 되어 어플리케이션 성능에 영향을 미치는 크나큰 단점 존재..!

![image-20241109211617689]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211617689.png)

- 따라서 브라우저 캐시(Cache) 를 이용해 Access-Control-Max-Age 헤더에 캐시될 시간을 명시해주면, 이 Preflight 요청을 캐싱 시켜 최적화 시켜줄 수 있다.
- 즉, 예비 요청으로 온 응답을 설정한 시간만큼 캐싱해놓고 그 값을 활용하는 것이다.

- 예비 요청 캐시는 다른 캐싱 매커니즘과 유사하게 작동
  
    ![image-20241109211625917]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211625917.png)
    
    1. 브라우저는 예비 요청을 할 때마다, 먼저 Preflight 캐시를 확인 → 해당 요청에 대한 응답이 있는지 확인
    2. 만일 응답이 캐싱되어 있지 않다면, 서버에 예비 요청을 보냄
    3. 만일 서버로부터 Access-Control-Max-Age 응답 헤더를 받는다면 그 기간 동안 브라우저 캐시에 결과를 저장
    4. 다시 요청을 보내고 만일 응답이 캐싱되어 있으면, 예비 요청 보내지고 않고 캐시된 응답을 사용

### 단순 요청(Simple Request)

- 예비 요청을 생략하고 바로 서버에 직행으로 본 요청을 보낸 후
  
    서버가 이에 대한 응답의 헤더에 Access-Control-Allow-Origin 헤더를 보내주면
    
    브라우저가 CORS 정책 위반 여부 검사하는 방식
    
    ![image-20241109211634299]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211634299.png)
    
- 다만, 특정 조건을 만족하는 경우에만 예비 요청 생략 가능
  
    대표적으로 아래 3가지 경우 만족할 때
    
    1. 요청의 메소드 → `GET`, `HEAD`, `POST` 중 하나 일때
    2. `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, `DPR`, `Downlink`, `Save-Data`, `Viewport-Width`, `Width` 헤더일 경우에만 적용
    3. Content-Type 헤더가 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 중 하나여야 함
- 까다로운 조건이 많기 때문에, 단순 요청이 일어나는 상황은 드물다고 보면 된다.
- 대부분의 HTTP API 요청은 `text/xml` 이나 `application/json` 으로 통신하기 때문에

**💡 따라서 대부분의 API 요청은 그냥 예비 요청으로 이루어진다 라고 이해하면 된다.**

### 인증된 요청(Credentialed Request)

- 인증된 요청은 클라이언트에서 서버에게 자격 인증 정보(Credential)를 실어 요청할 때 사용되는 요청
    - 자격 인증 정보 → 세션 ID 가 저장되어 있는 쿠키 혹은 Authorization 헤더에 설정하는 토큰 값 등
- 클라이언트에서 일반적인 JSON 데이터 외에도 쿠키같은 인증 정보를 포함해서 다른 출처의 서버로 전달할 때가 인증된 요청의 경우이다.
    - 이 경우 기존의 단순 요청, 예비 요청과는 다른 인증 형태로 통신하게 된다.

1️⃣ **클라이언트에서 인증정보를 보내도록 설정하기**

- 브라우저가 제공하는 요청 API 들은 별도의 옵션 없이는
브라우저의 쿠키같은 인증 관련 데이터를 함부로 요청 데이터에 담지 않도록 되어 있음
- 요청에 인증 관련 정보를 담을 수 있도록 하는 옵션 → `credentials` 옵션
    - 이 옵션에는 3가지 값 사용 가능
      
        ![image-20241109211643339]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211643339.png)
    
- 별도 설정 안해주면 쿠키 등 인증 정보는 절대 자동으로 서버에게 전송되지 않음
- 서버에 인증된 요청을 보내는 방법으로는 fetch 메서드 / axios, jQuery 라이브러리 등 다양함
    - 어떤 메서드를 사용하느냐에 따라 `credentials` 옵션을 지정하는 문법이 다르다

**2️⃣ 서버에서 인증된 요청에 대한 헤더 설정하기**

- 서버도 마찬가지로 인증된 요청에 대해 일반적인 CORS 요청과는 다르게 대응
    1. 응답 헤더의 `Access-Control-Allow-Credentials` 항목을 true 로 설정
    2. 응답 헤더의 `Access-Control-Allow-Origin` 의 값에 와일드카드 문자 (”*”) 는 사용할 수 없음
    3. 응답 헤더의 `Access-Control-Allow-Methods` 의 값에 와일드카드 문자 (”*”) 는 사용할 수 없음
    4. 응답 헤더의 `Access-Control-Allow-Headers` 의 값에 와일드 카드 문자(”*”)는 사용할 수 없음
- 응답의 Access-Control-Allow-Origin 헤더가 와일드카드가 아닌 분명한 Origin 으로 설정되어야 하고, `Access-Control-Allow-Credentials` 헤더는 `true` 로 설정되어야 한다는 뜻

![image-20241109211651481]({{site.url}}/images/2024-08-15-niornear_project(11)/image-20241109211651481.png)

- 인증된 요청 역시, 예비 요청이 먼저 발생한다.
- 위의 그림에서는 단순 GET 요청이기 때문에 예비 요청은 생략

```
# 헤더에 작성된 출처만 브라우저가 리소스를 접근할 수 있도록 허용함.
# * 이면 모든 곳에 공개되어 있음을 의미한다.
Access-Control-Allow-Origin : https://naver.com

# 리소스 접근을 허용하는 HTTP 메서드를 지정해 주는 헤더
Access-Control-Request-Methods : GET, POST, PUT, DELETE

# 요청을 허용하는 해더.
Access-Control-Allow-Headers : Origin,Accept,X-Requested-With,Content-Type,Access-Control-Request-Method,Access-Control-Request-Headers,Authorization

# 클라이언트에서 preflight 의 요청 결과를 저장할 기간을 지정
# 60초 동안 preflight 요청을 캐시하는 설정으로, 첫 요청 이후 60초 동안은 OPTIONS 메소드를 사용하는 예비 요청을 보내지 않는다.
Access-Control-Max-Age : 60

# 클라이언트 요청이 쿠키를 통해서 자격 증명을 해야 하는 경우에 true.
# 자바스크립트 요청에서 credentials가 include일 때 요청에 대한 응답을 할 수 있는지를 나타낸다.
Access-Control-Allow-Credentials : true

# 기본적으로 브라우저에게 노출이 되지 않지만, 브라우저 측에서 접근할 수 있게 허용해주는 헤더를 지정
Access-Control-Expose-Headers : Content-Length
```

# Spring 세팅

```java
// 스프링 서버 전역적으로 CORS 설정
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
        	.allowedOrigins("http://localhost:8080", "http://localhost:8081") // 허용할 출처
            .allowedMethods("GET", "POST") // 허용할 HTTP method
            .allowCredentials(true) // 쿠키 인증 요청 허용
            .maxAge(3000) // 원하는 시간만큼 pre-flight 리퀘스트를 캐싱
    }
}
```

```java
// 특정 컨트롤러에만 CORS 적용하고 싶을때.
@Controller
@CrossOrigin(origins = "*", methods = RequestMethod.GET) 
public class customController {

	// 특정 메소드에만 CORS 적용 가능
    @GetMapping("/url")  
    @CrossOrigin(origins = "*", methods = RequestMethod.GET) 
    @ResponseBody
    public List<Object> findAll(){
        return service.getAll();
    }
}
```

출처:

[🌐 악명 높은 CORS 개념 & 해결법 - 정리 끝판왕 👏](https://inpa.tistory.com/entry/WEB-📚-CORS-💯-정리-해결-방법-👏)

[Inpa Dev 👨‍💻:티스토리]