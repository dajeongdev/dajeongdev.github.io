---
layout: post
title: [JAVA] 배열 자르기
subtitle: 
categories: wiki
tags: [Java,Array,Method]
---

# 배열 자르기

## 1. Stream의 skip과 limit
**Arrays.stream.skip(num)**
- skip은 숫자(num)만큼 아이템을 건너뛴 다음 그 뒤의 아이템으로 새로운 스트림을 생성한다.
**Arrays.stream.limit(num)**
- limit은 지정한 개수(num)만큼 가져와서 새로운 스트림을 리턴한다.

```java
int[] array = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

int[] skip = Arrays.stream(array)
						.skip(4)
						.toArray(); // [5, 6, 7, 8, 9, 10]

int[] limit = Arrays.stream(array)
							.limit(4)
							.toArray(); // [1, 2, 3, 4]
```
<br/>


## 2. copyOfRange
- i번째 숫자부터 j번째 숫자까지 자를 때 사용할 수 있다.

```java
int[] array = {1, 2, 3, 4, 5};
int[] temp = Arrays.copyOfRange(array, 2, 4); // [3, 4, 5]
```