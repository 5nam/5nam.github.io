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

# 3. 몇 가지 알고리즘과 시간복잡도

- 알고리즘의 효율성은 시간복잡도와 공간복잡도로 체크한다.

## [다항식 값 구하기]

다항식 

![image-20241020224625569]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020224625569.png)

에 대하여 f(a) 값을 구해보자.



algorithm Horner's method

```java
procedure Horner(a,a0,a1,a2, ... ,an)
b = an // 최고차항
for i = 1 to n
	b = a_(n-1) + ba
output b
```

예) 

![image-20241020224853642]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020224853642.png) : 총 10번의 곱하기 연산 수행

![image-20241020225012337]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020225012337.png) : 총 4번의 곱하기 연산 수행



다항식 코딩을 할 때 아래가 훨씬 효율적!

위 과정에서 곱셈 횟수 n, 덧셈횟수 n 이다. O(n) 의 시간복잡도

## [탐색 알고리즘(searching algorithm)]

- 목록에 있는 원소중에서 특정성질의 원소를 찾는 알고리즘

**선형탐색 알고리즘**

```java
procedure linear(x, a1, a2, ..., an)
i=1
while(i<=n and x!=ai)
	i = i+1
if i<=n then location=i
else location=0
output location
```

예) **6**	3, 2, 8, 1, 4, 5, 6, 9, 0, 7

위 과정에서 최대 비교 횟수는?

2n + 2 -> O(n)



**이진탐색 알고리즘**

```
procedure binary(x,a1,a2,...,an) // ** 오름차순 정렬된 데이터
i=1 // **왼쪽 끝점
j=n // **오른쪽 끝점
while(i<j)
	m = (i+j) / 2 // ** 바닥함수, (n+1)/2 이하 정수들 중 최대정수
	if x > a_m then i = m+1
	else j = m

if x = a_i then location = i
else location = 0
output location
```

예) **19**	1,2,3,5,6,7,8,10,12,13,15,16,18,19,20,22

n 은 16, ![image-20241020225938500]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020225938500.png)

1. i=1, j=16
   1. 17/2 = 8, x 랑 8번째 숫자랑 비교. x가 더 큼
2. i=8+1, j=16
   1. (9+16)/2 = 12, x 랑 12번째 원소 비교. x 가 더 큼.
3. i=12+1, j=16
   1. (13+16)/2 = 14, x 랑 14번째 숫자 비교. x랑 19 일치

즉, 시간복잡도는 ![image-20241020230109853]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020230109853.png)



## [정렬 알고리즘(sorting algorithm)]

순서대로(일렬로 비교가능) 배열(예. 성명순, 학번순, 성적순 등)



**버블정렬**

```
procedure bubble(a1, a2, ..., an)
for i=1 to n-1
	for j=1 to n-i
		if a_j > a_{j+1} then interchange a_j and a_{j+1}
```

예)

![image-20241020230842880]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020230842880.png)

이중 for 문

![image-20241020231011018]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020231011018.png)

![img]({{site.url}}/images/2024-10-11-discrete_mathematics_4/img.png)

즉, 시간복잡도는 ![image-20241020231114722]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020231114722.png)



**삽입정렬**

```
procedure insertion(a1,a2,...,an)
for j=2 to n-1
	i=1
	while a_j > a_i
		i=i+1
	m=a_j
	for k=0 to j-i-1
		a_{j-k} = a_{j-k-1}
	a_i = m
```

예) 3,2,4,1,5 를 삽입정렬을 이용하여 정렬하시오

![image-20241020231250411]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020231250411.png)

즉, 시간복잡도는 ![image-20241020231330109]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020231330109.png)



**병합정렬**

정렬된 두 목록을 합쳐서 정렬하는 방법

![image-20241020231411656]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020231411656.png)

예) L1 : 3,5,8 / L2 : 1,7,8

첫 번째 원소를 비교, 더 작은 것을 제거 후 다른 변수에 저장. 이걸 계속 반복

1. L1 : 3,5,8 / L2 : 7,8
2. L1 : 5,8 / L2 : 7,8
3. L1 : 8 / L2 : 7,8
4. L1 : 8 / L2 : 8
5. 최종 배열 = {1, 3, 5, 7, 8, 8}

대소비교 횟수는 최대 m+n-1 이다.

예) 정렬되지 않은 데이터의 합병. {6,5,3,1,8,7,2,4}, n=8

L1 = 6, L2 = 5, L3 = 3, L4 = 1, L5 = 8, L6 = 7, L7 = 2, L8 = 4

1. L1, L2 병합정렬 -> L1 = {5, 6}
2. L1, L3 병합정렬 -> L1 = {3, 5, 6}
3. L1, L4 병합정렬 -> L1 = {1,3,5,6}
4. L1, L5 병합정렬 -> L1 = {1,3,5,6,8}
5. L1, L6 병합정렬 -> L1 = {1,3,5,6,7,8}
6. L1, L7 병합정렬 -> L1 = {1,2,3,5,6,7,8}
7. L1, L8 병합정렬 -> L1 = {1,2,3,4,5,6,7,8}



원소개수 n = 2^k 일 때 병합정렬의 최대 대소비교 횟수는

![image-20241020233028319]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020233028319.png)

## [재귀 알고리즘(recursive algorithm)]

어떤 함수가 자기 자신을 다시 호출해서 사용하는 알고리즘

![image-20241020233303392]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020233303392.png)



**n! 계산에 대한 재귀 알고리즘**

```java
procedure factorial(n)
if n=0 then return 1
else return n*factorial(n-1)
```



**a^n 계산을 위한 재귀 알고리즘**

```java
procedure power(a,n)
if n=0 then return 1
else return a*power(a,n-1)
```



**피보나치 수에 대한 재귀 알고리즘**

```java
procedure fibonacci(n)
if n=0 then return 0
else if n=1 then return 1
else return fibonacci(n-1)+fibonacci(n-2)
```

앞의 두 개를 더해서 값으로 지정하는 식의 알고리즘 f_n = f_{n-1} + f_{n-2}

![image-20241020233654566]({{site.url}}/images/2024-10-11-discrete_mathematics_4/image-20241020233654566.png)



```java
procedure iterative fibonacci(n)
if n=0 then return 0
else x=0, y=1
	for i=1 to n-1
		z=x+y, x=y, y=z
	return y
```

