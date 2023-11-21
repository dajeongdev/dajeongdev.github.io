---
layout: post
title: 문자열 자르기
subtitle: String.split(), StringTokenizer
categories: wiki
tags: [Java,Method]
---
### 1. String.split()
- 지정한 구분자(정규표현식)로 문자열을 나눠 문자열 배열에 저장한다.
- 공백 포함   
<br/>

### 2. StringTokenizer
- 지정한 구분자(문자열)로 문자열을 나눈다.
- 구분자 생략 시 공백이 디폴트
- **생성자**
   
```java
StringTokenizer(String str) // 공백 구분자 사용
StringTokenizer(String str, String delim) // 구분자 지정
StringTokenizer(String str, String delim, boolean returnDelims) // 구분자도 토큰에 포함
```
- **메소드**
   
```java
nextToken()     // 다음 토큰 반환 (이전 토큰은 삭제)
hasMoreTokens() // 리턴할 토큰이 남아있으면 true, 없으면 false
countTokens()   // 남아있는 토큰의 갯수
```
<br/>   

#### 공통점
- 지정한 구분자를 통해 문자열을 나눈다.   
<br/>

#### 차이점

|  | String.split() | StringTokenizer |
| --- | --- | --- |
| 구분자 | 정규표현식 | 문자/문자열 |
| 빈 문자열 인식 | O | X |
| 결과값 | String[] | StringTokenizer → 반복문으로 출력 가능 |
