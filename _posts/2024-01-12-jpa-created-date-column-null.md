---
layout: post
title: update 시 @CreatedDate를 붙인 컬럼에 null이 들어가는 문제와 해결 방법
subtitle: 
categories: wiki
tags: [Spring, JPA]
---
### 배경
- update문의 결과를 확인하는데 생성일 컬럼이 null로 update 되는 문제를 발견했다.   
<br/>   
   

### 원인
- 정확히는 생성일 컬럼에 이름을 설정하기 위해 붙인 @Column 어노테이션의 옵션 중 하나인 updatable의 기본값이 true라서 값을 설정해주지 않은 생성일 컬럼에 null이 update 된 것이다.

```java
@Target({METHOD, FIELD}) 
@Retention(RUNTIME)
public @interface Column {
	    /**
	     * (Optional) Whether the column is included in SQL UPDATE 
	     * statements generated by the persistence provider.
	     */
	    boolean updatable() default true;
  }
 ```
<br/>


### 해결 방법
- 그래서 @Column 어노테이션의 updatable 옵션을 false로 지정해주어야 한다.

```java
@CreatedDate
@Column(name = "created_date_time", updatable = false)
private LocalDateTime createdDateTime;
```
<br/>


#### 참고
- [JPA @CreatedDate Column Update시 Null 되는 현상 해결방법](https://wakestand.tistory.com/935)