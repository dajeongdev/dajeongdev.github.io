---
layout: post
title: Dangling meta character '?' near index 0
subtitle: 
categories: wiki
tags: [Java,Error]
---

# Dangling meta character '?' near index 0

## 에러
<img src="../../../../assets/images/posts/java-regex-pattern-syntax-error.png">
<br>

## 코드
``` java
String[] split = question.split("?");
```
<br>

## 원인
* split() 메소드 사용시 특정한 문자를 regex로 사용하는 경우 동작하지 않는다.
	* 특정한 문자: 이미 문법 상 사용 중인 특수문자 (|, &, ^, *, ?, . 등)
<br>

## 해결 방법
* regex 앞에 **백슬래시(\\)**를 **두 번** 넣어주면 코드가 정상적으로 동작한다.
``` java
String[] split = question.split("\\?");
```
