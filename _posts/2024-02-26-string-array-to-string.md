---
layout: post
title: String 배열 -> String
subtitle: 
categories: wiki
tags: [Java, String]
---
### 배경
- 코딩테스트 문제를 풀면서 String 배열을 String으로 변환하는 작업이 자주 나와서 기록해두려고 한다.   
<br/>  
   

### 1. String.join(CharSequence delimiter, CharSequence... elements)
- `join()` 메서드는 delimiter(구분자)를 사용하여 elements(대상 배열)을 문자열을 리턴할 수 있다.
- String.join() 메서드는 Java 1.8 이상부터 사용 가능
```java
String[] strArr = {"hello", "java"};
System.out.println(String.join(",", strArr));
// 결과 : hello,java
```
<br/> 

### 2. Arrays.toString(Object[] a)
- Arrays 클래스의 `toString()` 메서드를 사용하여 배열 모양의 문자열을 리턴할 수 있다.
```java
String[] strArr = {"hello", "java"};
System.out.println(Arrays.toString(strArr));
// 결과 : [hello, java]
```
<br/> 

### 3. Stream - Collectors.joining()
- Stream의 `Collectors.joining()` 메서드를 사용하여 문자열을 리턴할 수 있다.
- Stream은 Java 1.8 이상부터 사용 가능
```java
String[] strArr = {"hello", "java"};
System.out.println(Arrays.stream(strArr).collect(Collectors.joining()));
// 결과 : hellojava
```
<br/> 

### 4. StringBuilder.append(String str)
- 가장 기본적인 방법 중에는 StringBuilder를 생성하여 `append()` 메서드를 사용하여 문자열을 리턴할 수 있다.
- 빠르지만 for문을 사용하기 때문에 코드가 조금 길어진다는 단점이 있다.
```java
String[] strArr = {"hello", "java"};
StringBuilder sb = new StringBuilder();
for (int i = 0; i < strArr.length; i++) {
	sb.append(strArr[i]);
}
System.out.println(sb);
// 결과 : hellojava
```
<br/> 

### 5. Apache Commons Lang 3  라이브러리
- [Apache Commons Lang 3](https://commons.apache.org/proper/commons-lang) 라이브러리는 Java의 유틸리티 클래스 패키지로, 문자열, 배열, 숫자 등 매우 다양하고 편리한 기능을 제공한다.
- 그 중 StringUtils 클래스의 `join()` 메서드를 사용하여 문자열 배열을 문자열로 리턴할 수 있다.
```java
String[] strArr = {"hello", "java"};
System.out.println(StringUtils.join(strArr, ","));
// 결과 : hello,java
```