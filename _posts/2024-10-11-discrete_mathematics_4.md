---
layout: single
title:  "[이산수학] 04 알고리즘"
published : true
categories: discrete_mathematics
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 학습 내용

1. 알고리즘
2. Big-Oh 표기(중요)
3. 몇가지 알고리즘과 시간 복잡도
   1. 다항식 계산 알고리즘
   2. 검색 알고리즘
   3. 정렬 알고리즘
   4. 재귀 알고리즘

학습목표

- 알고리즘의 개념을 이해하고 표현방법 등을 아라보고 Big-Oh 표기법을 통해 복잡성을 측정하여 알고리즘의 효율성을 판단한다.
- 검색, 정렬, 재귀 알고리즘을 학습한다.

# 1. 알고리즘

**Def**. 알고리즘은 계산을 수행하거나 문제를 푸는 명확한 명령어의 유한한 순서 있는 나열

- 알고리즘을 묘사할 때 아래의 특징을 고려

1. 입력(input), 출력(output) 을 갖는다
2. 각 단계가 명확히 정의되어야 한다
3. 문제 해결 과정과 출력은 논리적이고 정확해야 한다.
4. 어떤 입력에 대해서도 유한한 수의 단계를 거치도록 해야 한다
5. (중요) 각 단계의 효율성이 있어야 한다(최대한 짧은 시간)
6. 같은 유형의 문제에 대해 항상 적용될 수 있는 일반성이 있어야 함

## 알고리즘의 표현

1. 의사코드(Pesudocode)

   - 알고리즘을 프로그램 언어와 유사한 형태로 풀어 써 놓은 것으로 정형화된 문법적 측면을 배제하고 사고의 흐름을 간결하고 효과적으로 전달하는 표현

   ![image-20241012002004163]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241012002004163.png)

2. 순서도(flow chart)

   - 처리 순서를 단계화하고, 상호간의 관계를 표준 기호를 사용하여 입력, 처리, 결정, 출력 등의 박스와 연결선으로 일목요연하게 나타낸 도표

# 2. Big-O 표기

**Def**. x > k 일 때, |f(x)| <= C|g(x)| 를 만족하는 상수 C 와 k 가 존재하면 f(x) = O(g(x)) 라 한다. 이때 f(x)는 g(x)의 big-oh 라고 한다.

![image-20241012002246558]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241012002246558.png)

예시 1)

7x^2 = O(x^2) : OK

7x^2 = O(x^3) : OK, 차수가 더 크게 표현하는 것은 괜찮음

x^3 = O(7x^2) : NO, 차수가 더 작게 표현하는 것은 안됨



예시 2)

n^2 = O(n) : NO, 차수가 맞지 않음(차수가 더 작게 표현됨)



**Thm.** 두 양수 m, n(m < n) 에 대하여 f(x) = x^m, g(x) = x^n 일 때, f(x) = O(g(x)) 은 성립하고, g(x) = O(f(x)) 은 성립하지 않는다.

예) x^2 = O(x^3) : OK, x^3 != O(x^2) : NO



**Thm.**

![image-20241012003137883]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241012003137883.png)



![image-20241012010834455]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241012010834455.png)

![image-20241012011822638]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241012011822638.png)

![image-20241012011836699]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241012011836699.png)