---
title:  "[니어니어 프로젝트] Long 과 long 의 차이"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# Long 과 long 의 차이

- 이전에 NULL 값을 할당할 수 없던 원시타입으로 인해 DB 를 다시 create 했던 경험때문에, Entity 의 값은 습관적으로 참조 타입으로 했었다.
- 그러나 코드 리뷰 때 받은 <🤔 왜 Null 값이 들어가지 않는 컬럼을 참조 타입으로 설정하셨나요? 원시 타입이 더 좋지 않을까요?> 라는 피드백으로 공부해봐야겠다고 생각했다.

## long 은 원시타입(Privitive Type), Long 은 참조 타입(Reference Type)

- 원시 타입 → 실제 메모리에 데이터 값 자체를 직접 저장하는 타입
    - boolean, char, byte, short, int, long, double
- 참조 타입 → 객체의 주소를 저장하는 타입으로 메모리 주소 값을 통해 객체를 참조하는 타입
    - 원시 타입을 제외한 문자열, 배열, enum, 클래스 인터페이스 등
    - 원시 타입에 대비되는 것들은 래퍼 클래스에 있는 Boolean, Character, Byte, Short, Integer, Long, Float, Double 등이 있다.

## 참조 타입을 사용하는 경우는?

- 매개 변수로 객체를 요구할 때, 기본형 값이 아닌 객체로 저장해야 할 때, 객체 간의 비교가 필요할 때 등등의 경우에 사용한다.
- 또한, Null 값을 할당해야 할 때 사용한다. 원시타입은 null 값을 할당할 수 없기 때문이다.

## 하지만, 원시 타입은 성능이 좋다.

- 원시 타입은 직접 값을 할당한다는 점에서 참조 타입에 비해 성능이 좋다.
- 원시 타입 → ‘Stack’ 영역에 값이 존재
- 참조 타입 → ‘Stack’ 영역에는 참조 주소 정보만 있고, 실제 데이터는 ‘Heap’ 영역에 존재

<aside>
💡 값을 가져오는 데에 있어 원시 타입이 속도 상 빠르고, 메모리 측면에서도 원시 타입을 사용하는 것이 좋다!

</aside>

## 도메인에서 id 값에 Long 을 사용하는 이유는?

- id 값은 대체로 DB 테이블의 auto increment 값을 의미하는 경우가 많다.
- 즉, 데이터가 생성되는 시점에서 해당 값이 할당된다는 것이며 도메인 객체의 id 는 특정 시점에 존재할 수도 있고, 존재하지 않을 수도 있다.

<aside>
💡 결론적으로, 변수가 not null 이 보장된다면 원시 타입을 사용하고 변수가 특정 시점에 따라 not null 이 보장되지 않는다면 참조 타입을 사용한다.

</aside>

## ⭐ 원시 타입과 참조 타입을 혼용해서 사용하지 말자!

- 원시 타입 → 참조 타입 : Boxing
- 참조 타입 → 원시 타입 : UnBoxing
- 자바에서는 이런 변환을 명시적으로 선언하지 않아도, Auto Boxing 과 Auto UnBoxing 기능을 제공
- 이 경우 불필요한 객체 생성으로 인한 성능 저하가 일어날 수 있다!

```java
public class Sum {
    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++) {
            sum += i;
        }
        return sum;
    }
}
```

- long 으로 선언된 i 값이 Long 으로 선언된 sum 변수와 연산
  
    → Auto Boxing 을 함
    
    → long 을 Long 으로 변환하는 과정에서 불필요한 Long 인스턴스 생성
    
    → long 으로 선언했을 때와 비교하면 약 3배의 실행 속도 차이 확인 가능
    
- 최대한 원시 타입을 사용, 원시 타입과 참조 타입이 혼용되지 않도록 주의!

출처 :

[[Java] Long과 long의 차이](https://bluemoon-clover.tistory.com/129)