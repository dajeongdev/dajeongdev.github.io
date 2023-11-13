---
layout: post
title: 문자열 치환
subtitle: 
categories: wiki
tags: [Java,String,Method]
---

# 문자열 치환

## 종류
1. replace()
2. replaceAll()
3. replaceFirst()
<br>

## replace(CharSequence target, CharSequence replacement)
- 일치하는 문자열(target)을 대체할 문자(replacement)로 치환한다.
```java
String str = "가/나/다/라/마/바/사";
System.out.println(str.replace("/", ",")); // 가,나,다,라,마,바,사
```
<br>

## replaceAll(String regex, String replacement)
- 일치하는 정규 표현식(regex)을 대체할 문자열(replacement)로 **전부** 치환한다.
    - replace()와 유사하지만, `.`(온점) 같은 경우 정규 표현식으로 모든 문자를 의미하기 때문에 모든 문자열이 치환될 수 있다.
```java
String str = "가.나.다.라.마.바.사";
System.out.println(str2.replace(".", ",")); // 가,나,다,라,마,바,사
System.out.println(str2.replaceAll(".", ",")); // ,,,,,,,,,,,,,
```
<br>

## replaceFirst(String target, String replacement)
- 첫번째로 일치하는 문자열(target)를 대체할 문자(replacement)로 치환한다.
```java
String str = "가/나/다/라/마/바/사";
System.out.println(str.replaceFirst("/", ",")); // 가,나/다/라/마/바/사
```
<br>

**실무 사용 예시**
→ 해시태그가 포함되어 있는 내용에 바꾸고 싶은 내용이 해시태그마다 다를 때 사용함
```java
String contents = "...";
contents = contents.replaceFirst("#", str1)
                .replaceFirst("#", str2)
                .replaceFirst("#", str3)
                .replaceFirst("#", str4)
                .replaceFirst("#", str5);
```
<br>

참고
- [https://coding-factory.tistory.com/128](https://coding-factory.tistory.com/128)