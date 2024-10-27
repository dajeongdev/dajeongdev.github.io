---
layout: post
title: Validation 그거 어떻게 하는건데
categories: wiki
subtitle: 
tags: [Spring, SpringBoot, Validation, 글또, 글또10기]
---
### 배경
백엔드 작업에서 Validation은 꽤나 중요도가 높은 작업으로, 클라이언트가 보내는 데이터가 서버에서 기대하는 형식과 규칙을 충족하는지 확인하는 과정을 의미합니다. 
주로 웹 애플리케이션에서 클라이언트가 API를 통해 데이터를 전송할 때 백엔드에서는 이 데이터가 정확한지, 요구사항을 만족하는지 검증을 해야 데이터베이스에 저장하거나 다음 처리를 진행하게 됩니다.
<br/>
<br/>


### Validation이 중요한 이유
1. **데이터 무결성 보장**
- 검증되지 않은 데이터가 서버로 들어오면 데이터베이스의 일관성이 깨질 수 있습니다. 
- 예를 들어, 생년월일 필드에 알파벳이 포함되거나 전화번호에 특수 문자가 포함되는 경우, 데이터의 신뢰성을 확보하기 어렵습니다. 이때 Validation을 통해 이러한 잘못된 데이터를 사전에 차단할 수 있습니다.
    
2. **보안 강화**
- 유효하지 않은 데이터는 SQL 인젝션, 스크립트 인젝션(XSS) 등과 같은 보안 취약점을 노출시킬 수 있습니다.
- Validation을 통해 이러한 악성 입력을 막음으로써 백엔드 시스템을 보호할 수 있습니다.
    
3. **애플리케이션 안정성 유지**
- Validation을 통해 예상하지 못한 데이터 형식으로 인한 에러를 줄일 수 있습니다. 
- 검증된 데이터만을 처리하기 때문에 애플리케이션의 안정성과 성능을 유지할 수 있습니다.
    
4. **비즈니스 로직 보호**
- 유효성 검사는 단순히 데이터 형식을 확인하는 것뿐만 아니라, 비즈니스 로직에 맞는 데이터인지 검토할 수 있습니다.
<br/>
<br/>


### 사용 방법
#### **1. 의존성 추가**
Spring Boot를 사용한다면 아래처럼 javax.validation 또는 jakarta.validation 패키지를 기본적으로 제공하지만, 프로젝트에 따라 Hibernate Validator 의존성을 명시적으로 추가해야 할 수도 있습니다.
```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

#### **2. 어노테이션 추가**
Validation은 주로 어노테이션을 기반으로 동작하기 때문에 DTO에 검증 어노테이션을 추가하여 Spring이 자동으로 검증을 수행하게 할 수 있습니다.
```java
public class UserRequest {

	@NotBlank(message = "이름을 필수입니다.")
	private String name;
	@Email(message = "이메일 형식을 확인해주세요.")
	private String email;
	@Size(min = 8, message = "비밀번호는 최소 8자 이상이어야 합니다.")
	private String password;

	// ...
}
```

#### **3. 컨트롤러에서 적용**  
컨트롤러에서 @Valid 어노테이션을 사용하여 DTO를 검증할 수 있습니다.
```java
@RestController
public class UserController {

    @PostMapping("/users")
	ResponseEntity<String> saveUser(@Valid @RequestBody UserRequest userRequest) {
		return ResponseEntity.ok("successfully saved User!");
	}
}
```

#### **4. 검증 에러 처리**
검증에 실패하면 Spring Boot는 기본적으로 MethodArgumentNotValidException을 발생시킵니다. 이를 처리하기 위해 @ExceptionHandler를 사용하거나 @ControllerAdvice를 통해 글로벌하게 처리할 수 있습니다.
```java
@RestControllerAdvice
class CommonExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    ResponseEntity<CommonResponse<List<String>>> handleValidationException(MethodArgumentNotValidException ex) {
        return new ResponseEntity<>(new CommonResponse<>(ErrorCode.INVALID_REQUEST_FORMAT.getErrorMessage(), null), HttpStatus.BAD_REQUEST);
    }
}
```
<br/>
<br/>


### 많이 쓰는 애노테이션과 팁
#### [기본 애노테이션]
`@NotNull`, `@NotEmpty`, `@NotBlank`
- `@NotNull`은 null이 아닌 값으로, 모든 타입에 사용 가능
- `@NotEmpty`은 null이거나 비어있지 않은 값으로, CharSequence, Collection, Map, 배열만 사용 가능
- `@NotBlank`는 null, 빈 문자열(`””`), 하나 이상의 공백이 아닌 문자만 허용

`@Size`
- 문자열, 배열, 컬렉션의 길이나 크기를 제한
- min, max 속성을 사용해 범위를 설정할 수 있음
<br/>

#### [숫자 관련 애노테이션]
`@Min`
- 숫자 필드의 최소값 지정

`@Max`
- 숫자 필드의 최댓값 지정

`@Positive`, `@PositiveOrZero`
- 값이 양수인지(0 포함 여부 선택)

`@Negative`, `@NegativeOrZero`
- 값이 음수인지(0 포함 여부 선택)

`@Digits`
- 정수와 소수의 자리수를 지정할 수 있음
- `@Digits(integer = 3, fraction = 2, message = “정수 3자리, 소수 2자리까지 가능합니다.”)`
<br/>

#### [문자열 관련 애노테이션]
`@Email`
- 문자열이 이메일 형식인지 검증

`@Pattern`
- 정규표현식을 사용하여 값이 특정한 패턴을 만족하는지 검증
- `@Pattern(regexp = "^[0-9]{10}$", message = "전화번호는 10자리 숫자여야 합니다.")`
<br/>

#### [날짜 관련 애노테이션]
`@Past, @PastOrPresent`
- 값이 과거 날짜인지, 현재 날짜를 포함한 과거인지 검증

`@Future`, `@FutureOrPresent`
- 값이 미래 날짜인지, 현재 날짜를 포함한 미래인지 검증
<br/>
<br/>