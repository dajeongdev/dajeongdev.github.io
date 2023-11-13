---
layout: post
title: 자바로 소수 판별
subtitle: 
categories: wiki
tags: [Java,Method,Algorithm]
---

# 소수 판별

## 소수
- 1과 자기 자신으로만 나누어지는 숫자
<br/>
<br/>

## 0부터 N까지 소수 구하기
```java
public int solution(int N) {
    int result = 0;

    int[] check = new int[N + 1];
    for (int i = 2; i < N; i++) {
        if (check[i] == 0) {
            result++;
            for (int j = i; j <= N; j += i) {
                check[j] = 1;
            }
        }
    }
    return result;
 }
```
<br>

## 해당 숫자가 소수인지 판별
```java
public boolean isPrime(int num) {
	if (num == 1) return false;

	// 2부터 시작해서 해당 숫자까지 약수가 있으면 false
	for (int i = 2; i < num; i++) {
		if (num % i == 0) return false;
	}

	// 약수가 없으면 true
	return true;
}
```
<br>

## Math 클래스의 sqrt() 메소드
- `sqrt()`: 2부터 루트 `n`까지의 범위에서 소수를 체크해주는 함수

```java
public int solution(int N) {
	for (int i = 2; i <= Math.sqrt(N); i++) {
		if (N % i == 0) return 0;
	}
	return 1;
}
```