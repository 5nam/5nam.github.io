---
title:  "[니어니어 프로젝트] 주문하기 기능 응답 시간 단축 - 비동기 처리 with @Async"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



# @Async 어노테이션 적용

- 스프링에서는 `@Async` 어노테이션을 제공하여 로직의 비동기 처리 지원



## 간단하게 사용하기

```java
@EnableAsync
@SpringBootApplication
public class MySpringApplication {
    ...
}
public class AsyncService {
    @Async
    public void asyncMethod() {
        ...
    }
}
```

- `@EnableAsync` 어노테이션을 통해 스프링이 비동기 처리를 위한 프록시 객체를 생성
- 이렇게 설정하면 Thread Executor 로 기본적으로 스프링은 SimpleAsyncTaskExecutor를 사용
- SimpleAsyncTaskExecutor
    - `org.springframework.core.task.SimpleAsyncTaskExecutor` 에 존재
    - `SimpleAsyncTaskExecutor` 는 매 요청마다 새로운 스레드를 생성
    - `ThreadPoolTaskExecutor` 는 미리 생성된 스레드 풀을 사용하기 때문에 오버헤드를 줄이고 성능을 향상시킬 수 있음
- ThreadPoolTaskExecutor 를 사용하는 예제 코드
  
    ```java
    @Configuration
    public class AsyncConfig {
        @Bean
        public Executor taskExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            executor.setCorePoolSize(10);
            executor.setMaxPoolSize(20);
            executor.setQueueCapacity(500);
            executor.initialize();
            return executor;
        }
    }
    ```
    



## AOP 와 @Async 어노테이션의 관계

- 스프링의 AOP(Aspect-Oriented Programming) 는 `@Async` 어노테이션의 비동기 처리를 구현
- `@Async` 가 붙은 메서드를 호출할 때, 스프링은 AOP 프록시를 통해 실제 메소드 호출을 가로채고
- 설정된 `AsyncTaskExecutor` 를 사용하여 메소드를 비동기적으로 실행
- 메소드 실행은 별도의 스레드에서 이루어지고, 메인 스레드는 블로킹되지 않고 다른 작업 계속 수행 가능
- `@Async` 어노테이션을 효과적으로 사용하기 위해서는 스프링의 AOP 동작 원리를 이해하는 것이 중요



## 유의사항

1. method 접근 지정자 private 사용 불가
2. self-invocation(자가 호출) 불가, 즉 inner method 사용 불가

왜 두 가지 사항이 생겨났을까? 작동 방식을 살펴보자



### 작동 방식

@Async 는 Spring AOP 에 의해 프록시 방식으로 작동

Spring Context 에 등록되어 있는 Async Bean 호출

→ Spring 이 개입하여 해당 Async Bean 을 프로식 객체로 Wrapping(프록시 객체화)

→ 호출한 객체는 실질적으로 AOP 를 통해 만들어진 프록시 객체화된 Async Bean 을 참조하게 됨

→ 즉, 비동기 함수를 호출하는 함수는 프록시 객체로 감싸진 비동기 함수를 호출하게 되는 것

✅ 그럼 비동기 함수로 처리되기 위해서는 AOP 가 비동기 함수로 지정된 것을 가로채서 프록시 객체로 만들어야 하는데, private 접근 제어자일 경우 접근할 수 없다.

✅ 비동기 함수는 프록시 객체로 감싸져서 프록시 객체를 꼭 거쳐서 호출되어야 하는데, 자가 호출의 경우 프록시 객체를 거치치 않고 직접 비동기 함수를 호출하기 때문에 Async 가 동작하지 않는다.



## 내가 적용한 코드

- 니어니어 서비스에서 주문하기 기능 2초 소요
- 딜레이 되는 부분을 찾다가, 문자 전송 기능을 주석 처리하니 빨라짐
- 그래서 문자 전송 기능을 비동기 처리 하면 좋겠다고 생각
    1. 반환 타입이 없고, 그냥 주문 정보를 통해 데이터를 넣어서 문자 전송한다.
    2. 다른 메서드와 상호 작용하는 메서드가 아니다.
    3. 외부 클래스에 존재한다.
    4. public 메서드이다.
    5. 빈에 등록된 Service 를 사용하여 호출하는 것이다.(생성자를 통해 생성하는 것 X)

```java
@Configuration
@EnableAsync
public class AsyncConfig {
}
@Async
public void sendMessage(Order order) {
    ...
}
```

- 비동기 처리를 하니 응답 시간이 1초 정도로 감소했다.

---

출처

[출처 1](https://velog.io/@think2wice/Spring-Async-Thread-Pool%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC-Async)

[출처 2](https://f-lab.kr/insight/understanding-spring-async-annotation-and-thread-pool)