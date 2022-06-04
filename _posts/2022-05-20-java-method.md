---
layout: post
title: Java method
subtitle: 
categories: wiki
tags: [Java]
---

### charAt(int index)
- 문자열의 index 위치에 있는 글자를 char 타입으로 리턴한다.
- index는 0부터 시작한다.
- 마지막 문자의 index는 문자열 길이 -1이다.

```java
String str = "Hello World";
System.out.println(str.charAt(0)); // H
System.out.println(str.charAt(str.length - 1)); // d
```
<br>

---

### 문자열에서 특정 문자 개수 세기
#### 1. 반복문
```java
public int count(String str, char c) {
    int count = 0;
    for (int i = 0; i < str.length(); i++) {
        if (str.charAt(i) == c) {
            count++;
        }
    }
    return count;
}
```
#### 2. replace()
```java
public int count(String str, char c) {
    return str.length() - str.replace(String.valueOf(c), "").length();
}
```
<br>

---

### 소수점 n번째 자리까지 반올림
#### 1. Math.round()
- 실수의 소수점 첫번째 자리를 반올림하여 정수로 리턴한다.
- 그 이상의 자리를 나타내고 싶다면 100.0 등을 곱해서 사용할 수 있다.
- 참고
    1. Math.ceil(): 올림
    1. Math.floor(): 버림

#### 2. String.format()
- String 클래스의 format 메소드는 리턴되는 문자열 형태를 지정할 수 있어 Math.round 메소드처럼 소수점 n번째 자리까지 반올림하여 나타낼 수 있다.

```java
double pie = 3.14159265358979;
System.out.println(String.format("%.2f", pie)); // 3.14
System.out.println(String.format("%.3f", pie)); // 3.142
```

#### 3. Math.round()와 String.format()
- 두 메소드는 실수를 소수점 n번째까지 잘라서 나타낸다는 공통점이 있다.
- 그러나 Math.round()는 소수점 아래가 0일 경우 절삭하지만, String.format()은 절삭하지 않고 그대로 리턴한다는 차이점이 있다.

```java
double money = 10000.000;
System.out.println(Math.round(money * 1000) / 1000); // 10000
System.out.println(String.format("%.3f", money)); // 10000.000
```
<br>

---
#### 참고
- https://coding-factory.tistory.com/250