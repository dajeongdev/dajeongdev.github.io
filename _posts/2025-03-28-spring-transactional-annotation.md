---
layout: post
title: Spring @Transactional을 정말 제대로 쓰고 있는 걸까?
categories: wiki
subtitle: 
tags: [Spring, Transactional]
---
우리는 수많은 데이터를 저장하고 수정하며, 그 과정에서 데이터의 무결성을 보장하는 것이 얼마나 중요한지 깨닫게 된다. 특히 JPA와 Spring을 활용하는 환경에서는 트랜잭션을 제대로 관리하지 않으면 예상치 못한 문제가 발생할 수 있다.  
JPA를 처음 접했을 때, 엔티티의 값을 바꾸면 자동으로 UPDATE 쿼리가 나가는 게 꽤 신기했다. 그런데 막상 실무에서 코드를 작성하다 보면 "왜 업데이트가 안 되지?", "왜 데이터가 반영되지 않았지?" 하는 순간들을 겪게 된다.  
그런 경험을 통해 JPA의 변경 감지가 트랜잭션 범위 내에서만 동작한다는 점을 알게 되고, 결국 @Transactional 애노테이션의 필요성을 체감하게 된다. 또 @Transactional(readOnly = true) 옵션을 활용하면 조회 성능을 최적화할 수 있다는 것도 알게 됐다.  
결국 중요한 건 단순히 @Transactional을 붙이는 게 아니라, 트랜잭션이 어떻게 동작하는지를 이해하고, 상황에 맞는 옵션을 적용하는 것이다.
<br/>
<br/>


### Trasaction
Spring에서는 트랜잭션 처리를 간편하게 하기 위해 `@Transactional` 애노테이션을 제공한다. 본격적으로 이 애노테이션을 살펴보기 전에, 먼저 트랜잭션 자체에 대해 간단히 짚고 넘어가자.
트랜잭션(Transaction)은 사전적으로는 '거래'라는 뜻이지만, 개발에서는 데이터를 일관성 있게 처리하기 위한 작업 단위로 이해하면 된다.
<br/>

#### 예시
예를 들어, 손님이 키오스크로 아메리카노를 한 잔 주문했다고 해보자. 
내부적으로는 다음과 같은 작업이 순서대로 일어난다.
1. 아메리카노 재고 확인
2. 손님의 카드 계좌에서 잔액 확인
3. 계좌에서 아메리카노 가격만큼 차감
4. 아메리카노 재고 차감
5. 주문 완료, 영수증 출력

이 모든 작업은 **모두 성공하거나, 하나라도 실패하면 전체가 취소**되어야 한다. 
그래야 손님의 돈만 빠져나가고 음료가 나오지 않는 상황을 방지할 수 있다. 
이처럼 하나의 단위로 묶여 처리돼야 하는 작업을 ‘**트랜잭션’**이라고 한다.  
<br/>  

#### 트랜잭션의 4가지 특징 (ACID)
1. **원자성(Atomicity)**
<br/>
\- 모든 작업은 전부 성공하거나 전부 실패해야 한다.    
2. **일관성(Consistency)**
<br/>
\- 트랜잭션 전후로 데이터는 항상 유효한 상태여야 한다. (예: 재고가 음수가 되면 안 됨)    
3. **격리성(Isolation)**
<br/>
\- 동시에 여러 트랜잭션이 실행되어도 서로 간섭하지 않아야 한다. 
4. **지속성(Durability)**
<br/>
\- 트랜잭션이 성공적으로 완료되면 그 결과는 영구적으로 반영되어야 한다.
<br/>
<br/>


### @Transactional 애노테이션
Spring에서는 `@Transactional`을 통해 트랜잭션을 선언적으로 관리할 수 있다. 이 애노테이션을 붙이면, 메서드 실행 전후로 트랜잭션을 시작하고 커밋하거나 롤백하는 동작을 자동으로 처리해준다.

#### 동작 방식
Spring의 `@Transactional`은 AOP(관점 지향 프로그래밍) 기반으로 동작한다. 정확히는 프록시 객체가 생성되고, 이 프록시가 메서드 실행 전후로 트랜잭션을 관리한다.
1. `@Transactional`이 붙은 메서드 호출
2. 프록시가 트랜잭션 시작
3. 메서드 실행
4. 예외 없으면 커밋, 예외 발생 시 롤백

단, 이 구조 때문에 **자기 자신 내부에서 메서드를 호출하면 트랜잭션이 적용되지 않는** 문제가 있다.
<br/>
<br/>


### 주요 옵션

| 옵션 | 설명 |
| --- | --- |
| `propagation` | 트랜잭션 전파 방식 (예: `REQUIRED`, `REQUIRES_NEW`, `NESTED` 등) |
| `isolation` | 트랜잭션 격리 수준 (`READ_COMMITTED`, `SERIALIZABLE` 등) |
| `rollbackFor` | 어떤 예외 발생 시 롤백할지 지정 (기본은 `RuntimeException`과 `Error`만 롤백) |
| `readOnly` | 읽기 전용 트랜잭션 여부 (쿼리 최적화에 사용) |
| `timeout` | 트랜잭션 제한 시간 초과 시 롤백 |

#### readOnly = true
조회만 수행하는 메서드라면 `@Transactional(readOnly = true)` 설정을 통해 성능을 최적화할 수 있다. 이 설정이 있으면 JPA는 변경 감지(Dirt Checking)를 위한 스냅샷을 저장하지 않으므로, 불필요한 리소스를 줄일 수 있다.

```java
@Transactional(readOnly = true) // 조회 성능 최적화
public Book findBook(Long bookId) {
    return bookRepository.findById(bookId).orElseThrow();
}
```

클래스 전체에 설정할 수도 있고, 메서드 단위로 오버라이드도 가능하다.

```java
@Service
@Transactional(readOnly = true) // 모든 메서드 조회 전용
public class BookService {

    public Book findBook(Long bookId) { ... }

    @Transactional // 이 메서드는 readOnly 적용되지 않음
    public void updateBook(Long bookId, String title) { ... }
}
```
<br/>
<br/>


### **실무에서 주의할 점**
#### 1. **자기 자신을 호출하는 메서드는 트랜잭션이 적용되지 않음**

```java
@Service
public class OrderService {

    @Transactional
    public void outerMethod() {
        innerMethod(); // 트랜잭션 적용되지 않음
    }

    @Transactional
    public void innerMethod() {
        // 실제 트랜잭션 적용되지 않음 (프록시를 거치지 않음)
    }
}
```
❓ @Transactional은 프록시 기반이기 때문에, 이 프록시가 메서드 호출을 가로채서 트랜잭션을 관리한다. 하지만 동일한 클래스 내에서 메서드가 자기 자신을 호출하는 경우에는 자기 호출(Self-Invocation)로 인식되어 프록시를 거치지 않기 때문에 innerMethod()의 트랜잭션은 처리되지 않는다.
**➡️** innerMethod를 별도 클래스/빈으로 분리하거나, `AopContext.currentProxy()`로 직접 호출 (비추천)

#### 2. 기본 롤백 대상은 RuntimeException만 가능하다.

```java
@Transactional
public void doSomething() throws Exception {
    throw new Exception(); // 롤백되지 않음
}
```
❓ 일반적인 Exception이나 CheckedException은 기본적으로 롤백 대상이 아니다.
**➡️** `@Transactional(rollbackFor = Exception.class)` 같이 명시적으로 지정

#### 3. 중첩 트랜잭션 (Nested Transaction)
`REQUIRES_NEW`는 기존 트랜잭션을 중단하고 새 트랜잭션을 시작하는 방식이다. 이는 실제 중첩 트랜잭션(NESTED)과는 다르며, 구현체에 따라 지원 여부가 달라질 수 있다. 성능이나 롤백 범위에 주의가 필요하다.

#### 4. readOnly = true 설정
readOnly 설정이 있는 트랜잭션 안에서 `INSERT`, `UPDATE` 등의 작업을 하면 DB 종류에 따라 에러가 나거나 무시될 수 있다. 꼭 조회 전용 메서드에서만 사용하는 것이 좋다.
<br/>
<br/>


### insight & thoughts
처음에는 그냥 애노테이션 하나 붙이면 되는 거 아닌가?라고 생각했지만, 실무에서 직접 트랜잭션 이슈를 겪다 보면 이게 왜 필요한지를 체감하게 된다. 특히, JPA의 변경 감지, 트랜잭션 경계, readOnly 최적화, 롤백 동작 방식을 이해하면 단순히 동작하는 코드를 넘어서 신뢰할 수 있는 코드를 만들 수 있다. 이제는 트랜잭션을 단순한 기술 요소가 아니라, 데이터 정합성을 지키기 위한 개념으로 받아들이게 된 것 같다.
<br/>
<br/>
<br/>


**참고**
- [Spring 공식 문서](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
- [JPA Transactional 잘 알고 쓰고 계신가요?](https://tech.kakaopay.com/post/jpa-transactional-bri)
- [@Transactional의 동작 방식 & 중첩이 일어났을 때 어떻게 될까?](https://m42-orion.tistory.com/160)