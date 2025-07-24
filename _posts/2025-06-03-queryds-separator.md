---
layout: post
title: QueryDSL에서 group_concat SEPARATOR 없이 구분자 처리하기
categories: wiki
subtitle: 
tags: [QueryDSL]
---

### 💡 문제 상황

종종 테이블의 여러 개로 들어가 있는 데이터를 List<String>의 형태로 보내야 할 때가 있다. QueryDSL을 사용할 때 이를 위해 보통 group_concat을 사용해 하나의 문자열로 묶은 뒤, DTO에서 String으로 받아 `String.split(”,”)`을 사용하여 List로 변환하하는 방식을 사용하곤 했다.
```java
Expressions.stringTemplate("group_concat({0})", book.title);
```
그런데 이 방식엔 치명적인 문제가 있다.
<br/>
<br/>


### 💫 데이터에 쉼표가 들어간다면?
예를 들어, 다음 같은 책 제목이 있다고 할 때
```
"첫 여름, 완주"
"다섯 번째 계절"
```
이 값들을 기존 방식대로 group_concat으로 묶은 뒤 DTO에서 `split(”,”)` 처리를 해보자.

```java
String titles = "첫 여름, 완주, 다섯 번째 계절";
List<String> aresult = Arrays.asList(titles.split(","));
// ["첫 여름", " 완주", "다섯 번째 계절"]
```

제목 안의 쉼표가 String.split()의 기준이 되어 값이 쪼개져버리는 문제가 발생한다.
<br/>
<br/>


### 🤔 그럼 SEPARATOR를 쓰면 되지 않을까?
물론 MySQL에서는 `group_concat(… SEPARATOR ‘|||’)` 같은 방식으로 구분자를 지정할 수 있다.

하지만 **QueryDSL 내부의 Hibernate에서 이 SEPERATOR를 SQL이 아니라 JPQL로 파싱하려고 하기 때문에 SyntaxException을 발생**하게 된다. 그래서 MySQL 전용 문법인 SEPARATOR를 직접 쓸 수 없다.

참고로 `group_concat`을 사용하면 기본적으로 `,(쉼표)`를 구분자로 사용한다.
<br/>
<br/>


### 해결: QueryDSL에서 구분자 수동 삽입

`group_concat`의 기본 구분자를 바꿀 수는 없지만, **각 값 뒤에 원하는 구분자(`|||`)를 직접 붙일 수 있다.**

```java
// QueryDSL
Expressions.stringTemplate(
    "group_concat(distinct concat({0}, '|||'))", book.title
)

// DTO
public static List<String> convertTitles(String titles) {
		// 쉼표로 분리된 값들을 '|||' 기준으로 나눔
    return Arrays.stream(titles.split("\\\\|\\\\|\\\\|")) 
	    // 앞에 붙을 수 있는 쉼표 제거
	  .map(s -> s.startsWith(",") ? s.substring(1) : s)
      .filter(s -> !s.isBlank())
      .toList();
}
```

이렇게 하면 `group_concat`은 여전히 쉼표로 값을 붙이지만, 각 값의 내부에는 우리가 지정한 구분자(`|||`)가 붙어 있어 안전하게 분리할 수 있다.
<br/>
<br/> 
<br/>


### 추가: 데이터 안에 반드시 쉼표 포함된다면?

만약 데이터 안에 쉼표가 반드시 포함된다면, 다음과 같이 한 번 더 escape 처리하는 것이 안전하다.

```java
// QueryDSL
Expressions.stringTemplate(
    "replace(group_concat(distinct replace({0}, ',', '##COMMA##')), ',', '|||')",
    book.title
)

// DTO
public static List<String> convertTitles(String titles) {
    return Arrays.stream(titles.split("\\\\|\\\\|\\\\|"))
	    .map(s -> s.replace("##COMMA##", ","))
      .toList();
}
```
<br/>
<br/>


### ✅ Native Query를 사용하지 않은 이유

Native Query를 사용하면 `SEPARATOR`나 `ORDER BY` 같은 SQL을 QueryDSL보다 자유롭게 쓸 수 있다.

```java
@Query(value = "select group_concat(distinct title SEPARATOR '|||') " +
	"FROM book", nativeQuery = true)
```

하지만 나는 여러 이유로 QueryDSL 방식을 선택했다.
-   사용해야 하는 조건이 많음
-   **타입 안정성**이 유리함
-   Native Query는 유지보수 어렵고, 타입 매핑이 불편함
-   문자열 조립을 잘못하면 **SQL Injection 위험**도 있음
    -   QueryDSL에서 파라미터는 내부적으로 `PreparedStatement`로 바인딩되기 때문에 **사용자 입력이 직접 쿼리 구조에 들어가지 않으며, SQL Injection 공격으로부터 안전**
<br/>
<br/>
<br/>


### 📌 마무리
더 좋은 방법이 있을 수 있겠지만 (있다면 댓글 부탁드립니다!)  
QueryDSL은 group_concat에서 SEPERATOR를 직접 지정할 순 없지만, 우회적으로 구분자를 삽입하고 후처리하는 방식으로 문제를 충분히 해결할 수 있었다.