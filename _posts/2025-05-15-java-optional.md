---
layout: post
title: null-safe하게 Optional 사용하기
categories: wiki
subtitle: 
tags: [Java, Java8, Optional]
---
### 배경
프로젝트 작업 중 Optional을 자주 사용하고 있다는 걸 인지하게 되었다. 기본 개념부터 주의할 점을 다시 살펴보니, 생각보다 다양한 방식으로 null-safe한 처리가 가능하다는 걸 알게 되었고, 흐름을 더 명확하게 표현할 수 있다는 점도 인상 깊었다. 이번 기회에 Optional을 어떻게 쓰는 것이 바람직한지, 실무에서 자주 하는 실수는 무엇인지 정리해보고자 한다.
<br/>
<br/>


## Optional
<img src="https://dajeongdev.github.io/assets/images/posts/optional.png" />

- Java에서 NullPointerException(NPE)은 가장 흔한 런타임 오류 중 하나다.
- Java 8 이전에는 메서드가 null을 반환할 경우, 사용하는 쪽에서 null 여부를 주석이나 문서로만 유추해야 했다.
- Java 8에서 도입된 Optional을 사용하여 null 여부를 명시적으로 표현할 수 있어, map, filter, flatMap 등 함수형 메서드를 지원하여 null-safe한 코드 작성이 가능하다.
<br/>
<br/>


## Optional 주요 메서드 요약

| 메서드 | 설명 |
|---|-------|
| map() | 내부 값을 가공하고 Optional로 감싸서 제공 |
| flatMap() | 내부 값이 또 다른 Optional일 때 중첩 방지 |
| filter() | 조건을 만족할 경우만 값을 유지 |
| ifPresent() | 값이 있으면 실행 (void 리턴) |
| ifPresentOrElse() (Java 9) | 값이 있으면 A 실행, 없으면 B 실행 |
| orElse() | 값이 없을 경우 기본값 제공 |
| orElseGet() | 값이 없을 경우 Supplier 실행 |
| orElseThrow() | 값이 없을 경우 예외 발생 |
| isPresent()/isEmpty()<br/>(Java 11) | 값 존재 여부 확인 (→ 가급적 사용 자제, ifPresent() 등 권장) |
| or() (Java 9) | 비어 있을 경우 대체 Optional 제공 |
| stream (Java 9) | Optional을 Stream으로 변환 |

### orElse와 orElseGet

```java
// orElse는 항상 실행
String name = opt.orElse(getExpensiveDefault());

// orElseGet은 필요할 때만 실행(lazy)
String name = opt.orElseGet(() -> getExpensiveDefault());
```

⭐️ `orElse()`는 기본값을 항상 먼저 계산하다. 불필요한 리소스 낭비가 되지 않도록, 기본값이 필수가 아니라면 `orElseGet()`을 사용하는 것이 좋다.

### isPresent() 사용 시 주의사항

```java
if (optionalUser.isPresent()) {
	User user = optionalUser.get(); // 여전히 예외 위험 있음
}
```

- Optional의 목적은 null-safe한 코드를 작성하는 건데, isPresent()를 써서 if문으로 다시 체크한다면 결국 이전 방식(if (value ≠ null)과 다를 바 없기 때문에 Optional을 쓰는 이유가 불분명해지게 된다.
    - 또한, isPresent()가 true라고 해도 중간에 다른 로직이 추가되면 get() 호출 시점에 Optional이 달라질 수 있어 위험하다.
- 이럴 때는 아래처럼 명시적인 함수형 스타일을 사용하면 좋다.

```java
User user = optionalUser
	.orElseThrow(() -> new NotFoundException("사용자를 찾을 수 없습니다."));
```

- ifPresent()도 비어있지 않을 때만 수행하는 작업을 표현하기 좋다.

```java
optionalUser.ifPresent(user -> System.out.println(user.getName());
```

### filter

- 조건에 맞는 값만 유지

```java
Optional<String> name = Optional.of("파인애플");
name.filter(n -> n.length() > 3)
		.ifPresent(n -> System.out.println("이름이 너무 깁니다. : " + n));
```

### map

- 값을 null-safe하게 변환

```java
Optional<String> name = Optional.of("파인애플");
Optional<Integer> nameLength = name.map(String::length);
log.info("결과: {}", nameLength.orElse(0)); // 결과: 4
```

### flatMap

- 중첩된 Optional 해제

```java
// User- > Optional<Profile>를 반환하는 메서드가 있을 경우
Optional<User> user = Optional.of(new User());

String email = user
	.flatMap(User::getProfile) // Optional<Profile>
	.map(Profile::getEmail)    // Optional<String>
	// map을 안 쓰면 Optioanl<Optional<String>>이 되어버림
	.orElse("이메일 정보 없음");
```
<br/>
<br/>


## **Optional 지양해야 할 사용법**

### 1. 무분별한 Optional.get() 사용
```java
Optional<User> user = userRepository.findById(1L);
**User u = user.get();** 
```
- `Optional.get()`은 내부에 값이 없을 경우 런타임 예외(NoSuchElementException) 발생하며, null-safe하게 코드를 작성하려고 Optional을 쓰는데, get()을 쓰면 그 의미가 사라진다.
- isPresent()와 함께 쓰더라도, 유지보수 중에 변경되면 예외 발생 가능성 존재한다.

**👍 대안:**
```java
User user = userRepository.findById(1L)
		.orElseThrow(() -> new CommonException(ErrorCode.NOT_FOUND));
```
- `orElse()`이나 `orElseThrow()` 사용

### 2. 필드에  사용
```java
public class User {
	private Optional<String> name;
}
```
- JPA, Jackson, Hibernate 등 직렬화/역직렬화/매핑 과정에서 오류가 발생할 수 있다.
- 필드는 일반적으로 객체의 상태를 표현하는데, Optional은 계산 결과나 조건적 반환을 위한 용도라 맞지 않는고, 메모리 낭비 및 불필요한 래핑으로 가독성이 저하될 수 있다.

**👍 대안:**
```java
public class User {
	private String name;
	public Optional<String> getNameOptional() {
		return Optional.ofNullable(name);
	}
}
```
- 필드는 일반 타입으로 선언, getter에서 Optional로 감싸는 방식 권장한다.

### 3. 파라미터나 리턴값으로 사용
```java
public void updateName(Optional<String> name) // 잘못된 설계
```
- Optional은 메서드 결과를 감싸기 위한 목적이지, 인자로 전달하는 용도가 아니며, 오히려 사용하는 쪽에서 of(), empty() 등을 강제하게 되어 복잡도가 증가한다.

**👍 대안:**
```java
public void updateName(String name) {
	// 내부에서 Optional.ofNullable(name)으로 처리
}
```
- 메서드 인자는 기본 타입으로 받고, 내부에서 Optional.ofNullable()로 처리한다.
```java
public Optional<User> findById(Long id) {
	return Optional.ofNullable(...);
}
```
- 리턴 타입으로 Optional을 쓰면 “값이 없을 수도 있다”라는 의도가 명확하게 드러나므로 좋다.
<br/>
<br/>


## Optional 단점

### 1. 성능 오버헤드
- Optional은 결국 하나의 래퍼 객체로, 값 하나를 감싸기 위해 객체가 추가로 생성된다. 특히 빈 Optional 객체는 재사용되지만, 값이 있을 경우 매번 Optional.of()로 새 인스턴스를 생성한다.
- 자주 호출되는 메서드에서 Optional을 남발하면 GC에 부담이 증가하고, 성능 저하 가능성이 높아진다.

### 2. 체이닝이 지나치면 가독성 저하
```java
Optional.ofNullable(user)
    .flatMap(User::getProfile)
    .map(Profile::getEmail)
    .filter(email -> email.endsWith(".com"))
    .orElse("이메일 없음");
```
- 함수형 스타일은 깔끔해 보일 수 있지만, 익숙하지 않은 개발자에겐 오히려 난해하다. 특히 예외 처리나 조건 분기가 복잡할 수록 명령형 코드보다 읽기 어려워질 수 있다.
<br/>
<br/>


## Optional을 사용하는 바람직한 위치
| 계층 | 사용 여부 | 이유 |
| --- | --- | --- |
| Repository | 리턴값 | 조회 결과가 없을 수 있다는 의도를 명확히 드러냄 |
| Service | 내부 처리/변환 시 | Optional을 바로 처리해서 도메인 객체로 변환하거나 예외 처리 |
| Controller | ❌ | 응답 객체는 명확하고 단순한 타입이 바람직하므로 Optional 노출 지양 |

```java
// Repository
Optional<User> findById(Long id);

// Service
public User findUserOrThrow(Long id) {
	return userRepository.findById(id)
			.orElseThrow(() -> new CommonException(ErrorCode.NOT_FOUND_USER));
}

// Controller
@GetMapping("/users/{id}")
ResponseEntity<UserResponse> getUser(@PathVariable("id") Long id) {
	User user = userService.findUserOrThrow(id);
	return ResponseEntity.ok(UserResponse.from(user));
}
```
➡️ Repository는 Optional로 감싸서 리턴하고, Service에서 처리하고, Controller에서는 노출하지 않는 것이 바람직하다.
<br/>
<br/>


## Optional과 Validation/Exception 처리

**1. Optional로 조건 분기 없이 깔끔한 예외 처리 가능하다.  **

```java
User user = userRepository.findById(id)
	.orElseThrow(() -> new CommonException(ErrorCode.NOT_FOUND_USER));
```
- orElseThrow()를 활용하면 null 체크와 예외 처리를 한 줄로 처리 가능하다.

**2. Validation과의 연계는 Optional 밖에서 수행한다.**
- 입력값이 Optional인 경우, Spring Validation에서는 제대로 처리되지 않을 수 있으므로, 요청 DTO에는 기본 타입으로 선언하고, 이후 변환 시 Optional을 사용한다.  
```java
public class UserRequest {
		@NotBlank
		private String name;
}
 ```    
- 받은 값이 null일 수 있다면, Service 계층에서 Optional.ofNullable(request.getName())으로 감싸서 처리하는 것이 낫다.
<br/>
<br/>


## 마무리
Optional은 제대로 사용하면 강력한 null-safe 도구지만, 목적에 맞지 않게 사용하거나 남용할 경우 오히려 가독성과 성능을 저하시킬 수 있으므로, 많이 사용할 수록 바람직한 방식으로 사용하는 것이 중요하다.

✅ Optional은 리턴에만 적절하게 사용<br/>
❌ get(), isPresent() 남용 금지<br/>
🧩 orElseGet(), orElse 차이 유의<br/>