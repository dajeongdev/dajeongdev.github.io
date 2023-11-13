---
layout: post
title: 자바로 피보나치 수열 이해하기
subtitle: 
categories: wiki
tags: [Java,Algorithm]
---

# 피보나치 수열
- 피보나치 수열은 재귀 함수의 대표적인 예시로, 첫째 및 둘째 항이 1이며 그 뒤의 모든 항은 바로 앞 두 항의 합인 수열이다.
    - F(0) = 0, F(1) = 1일 때, 1 이상의 n에 대하여 F(n) = F(n-1) + F(n-2)
    - **재귀 함수**: 자기 자신을 다시 호출해 작업을 수행하는 함수
```java
public int fibonacci(int n) {
		if(n == 1 || n == 2) {
			return 1;
		} else if (n >= 3) {			
			return fibonacci(n - 2) + fibonacci(n - 1);
		}
}
```
<br/>

## 프로그래머스 피보나치 수 풀이
**원래 코드**
```java
public int solution(int n) {
	int answer = 0;
	int first = 1;
	int second = 1;
	if (n > 2) {
		for (int i = 3; i <= n; i++) {
			answer = (first + second);
			first = second;
			second = answer;
		}
	} else {
		return 1;
  }
	return answer;
}
```
- 이 코드를 제출했더니 7번부터 오답이 나왔다.. 그래서 한참 고민하다가 질문하기 페이지를 봤는데 좋은 답변을 달아주셔서 참고했다.
    <img src="../../../../assets/images/posts/fibonacci-reference.png">
    - **정리**: 자료형의 크기에 제한이 있는 언어를 사용할 경우 (A + B) % C == ((A % C) + (B % C)) % C라는 모듈러 연산의 성질을 이용하여, 계산 결과에 1234567으로 나눈 나머지를 대신 넣는 것으로 int 범위 내에 항상 값이 존재하도록 보장할 수 있다.

**수정 코드**
```java
public int solution(int n) {
	int answer = 0;
	int first = 1;
	int second = 1;
	if (n > 2) {
		for (int i = 3; i <= n; i++) {
			answer = (first + second) % 1234567;
			first = second;
			second = answer;
		}
	} else {
		return 1;
  }
	return answer;
}
```