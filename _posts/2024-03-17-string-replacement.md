---
layout: post
title: 문자열 대치
subtitle: 
categories: wiki
tags: [Java,String,Method]
---
### 종류
1. `replace()`
2. `replaceAll()`
3. `replaceFirst()`
<br/>
<br/>

### replace(CharSequence target, CharSequence replacement)
- 일치하는 문자열(target)을 새로운 문자열(replacement)로 치환한다.
```java
String str = "가/나/다/라/마/바/사";
System.out.println(str.replace("/", ",")); // 가,나,다,라,마,바,사
```
<br/>

### replaceAll(String regex, String replacement)
- 일치하는 정규 표현식(regex)을 새로운 문자열(replacement)로 **전부** 치환한다.
    - replace()와 유사하지만, `.`(온점) 같은 경우 정규 표현식으로 모든 문자를 의미하기 때문에 모든 문자열이 치환될 수 있다.
```java
String str = "가.나.다.라.마.바.사";
System.out.println(str2.replace(".", ",")); // 가,나,다,라,마,바,사
System.out.println(str2.replaceAll(".", ",")); // ,,,,,,,,,,,,,
```
<br/>

### replace()와 replaceAll()
- 두 메소드는 일반 문자열 리터럴을 비교할 땐 실행 결과에 차이가 없다.
- 하지만 replaceAll()은 문자열 대치에 정규식을 사용하여 조금 더 구체적인 검증이 가능하다.
<br/>
<br/> 
   
### replaceFirst(String target, String replacement)
- 첫번째로 일치하는 문자열(target)를 대체할 문자(replacement)로 치환한다.
```java
String str = "가/나/다/라/마/바/사";
System.out.println(str.replaceFirst("/", ",")); // 가,나/다/라/마/바/사
```
<br/>

### 실무 사용 예시
- 해시태그가 포함되어 있는 내용에 바꾸고 싶은 내용이 해시태그마다 다를 때 사용한다.
```java
String contents = reportMapper.findReportTemplate(60001);
contents = contents.replaceFirst("#", childName)
                .replaceFirst("#", String.valueOf(progress.getCount()))
                .replaceFirst("#", progress.getPercent())
                .replaceFirst("#", previous)
                .replaceFirst("#", average);
```
<br/>

#### 참고
- [[Java] 문자열 치환(Replace) 사용법 & 예제](https://coding-factory.tistory.com/128)
- 031724 추가 [String 클래스 메서드의 replace()와 replaceAll()의 차이점](https://bada744.tistory.com/16)