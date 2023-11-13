---
layout: post
title: 진수 변환
subtitle: 
categories: wiki
tags: [Java,Method]
---

# 진수 변환

## 10진수 → 2/8/16진수

| return type | class | method | 설명 |
| --- | --- | --- | --- |
| static String | java.lang.Integer | toBinaryString(int i) | 10진수 → 2진수 |
| static String | java.lang.Integer | toOctalString(int i) | 10진수 → 8진수 |
| static String | java.lang.Integer | toHexaString(int i) | 10진수 → 16진수 |

➕ 위의 세 메소드는 파라미터로 정수 뿐만 아니라 정수 두 개를 비트 연산자(OR)로 계산할 수도 있다.

## 10진수 → N진수

| return type | class | method | 설명 |
| --- | --- | --- | --- |
| static String | java.lang.Integer | toString(int n, int radix) | 10진수 → N진수 |

**예시**

```java
int decimal = 10;
String binary = Integer.toBInaryString(10);
String octial = Integer.toOctalString(10);
String hexa = Integer.toHexaString(10);

System.out.println("10진수: " + decimal); // 10
System.out.println("2진수: " + binary);  // 1010
System.out.println("8진수: " + octial); // 12
System.out.println("16진수: " + hexa); // a

/* 비트 연산자로 계산하기 */
Integer.toBinaryString(10 | 12); // 1110
```
<br/>

## 2/8/16진수 → 10진수

| return type | class | method | 설명 |
| --- | --- | --- | --- |
| static int | java.lang.Integer | parseInt(String s) | 문자열 → 10진수 int (default 10진수) |
| static int | java.lang.Integer | parseInt(String s, int radix) | 문자열을 변환할 진수(radix)로 읽어 int로 리턴 |

## N진수 → 10진수

| return type | class | method | 설명 |
| --- | --- | --- | --- |
| static int | java.lang.Integer | parseInt(String, s, int radix) | N진수 → 10진수 |

**예시**

```java
int binary = Integer.parseInt("1010", 2);
int octial = Integer.parseInt("12", 8);
int hexa = Integer.parseInt("A", 16);

System.out.println("2진수 -> 10진수: " + binary); // 10
System.out.println("8진수 -> 10진수: " + octial); // 10
System.out.println("16진수 -> 10진수: " + hexa); // 10
```