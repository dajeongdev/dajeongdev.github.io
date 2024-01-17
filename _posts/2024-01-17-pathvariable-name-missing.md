---
layout: post
title: @PathVariable 이름 미지정 시 에러
subtitle: 
categories: wiki
tags: [Spring, SpringBoot]
---
### 배경
- 팀원이 작업한 코드를 pull 받아서 실행시켰는데 아래처럼 에러가 발생했다. 그런데 팀원이 올리기 전에 실행시켰을 땐 발생하지 않았던 에러가 왜 내 로컬에서는 발생했을까?
```java
java.lang.IllegalArgumentException: Name for argument of type [java.lang.String] not specified, and parameter name information not found in class file either.
```
<br/>


### 원인
- Spring Boot 3.2 이전에는 경로의 변수명와 @PathVariable의 변수명이 같을 때 생략할 수 있었다.
```java
@GetMapping("/{id}")
public String testMethod(@PathVariable String id) {...}
```
 <img src="https://dajeongdev.github.io/assets/images/posts/parameter-name-retention.png">
- Spring Boot 3.2 전에는 바이트코드를 파싱해서 매개변수 이름을 추론하려고 시도했지만 3.2 부터는 이 부분이 사라져서 @PathVariable을 포함한 몇몇 어노테이션(@RequestParam, @Autowired, @ConfigurationProperties)에서 매개변수 인식 문제가 발생하고 있다.
- [공식 문서 참고](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-6.x#parameter-name-retention)
<br>


### 해결방법
1. **어노테이션에 이름을 생략하지 않고 항상 적는다.**
	```java
	@GetMapping("/{id}")
	public String testMethod(@PathVariable("id") String id) {...}
	```
1. **컴파일 시점에 -parameters 옵션을 적용한다.**
	1. IntelliJ IDEA에서 File -> Settings를 연다. (Mac은 IntelliJ IDEA -> Settings)
	2. Build, Execution, Deployment → Compiler → Java Compiler로 이동한다.
	3. Additional command line parameters라는 항목에 다음을 추가한다. (`-parameters`)
	4. out 폴더를 삭제하고 다시 실행한다. **꼭 out 폴더를 삭제해야 다시 컴파일이 일어난다.**
1. **Gradle로 빌드하고 실행한다.**
	- 참고로 이 문제는 Build, Execution, Deployment -> Build Tools -> Gradle에서 Build and run using를 IntelliJ IDEA로 선택한 경우에만 발생한다.
	- Gradle로 선택한 경우에는 Gradle이 컴파일 시점에 해당 옵션을 자동으로 적용해준다.
<br>


#### 나의 의견
- 결국 김영한님 강의를 많이 봤던 나는 프로젝트 초반에 Build and run using 설정을 IntelliJ로 바꾸고 사용 중이었고, 팀원은 Gradle로 빌드 및 실행을 하고 있어서 발생한 문제였다.
- 나 또한 설정을 바꿨지만, Spring Boot 3.2부터 이름을 붙이지 않으면 에러가 발생할 수도 있는 부분이기 때문에 우리 팀은 해결방법 1번을 선택하여 이름을 일관적이게 붙여주기로 결정했다.
<br>


#### 참고
- [https://www.inflearn.com/questions/1088283](https://www.inflearn.com/questions/1088283/pathvariable-%EB%B3%80%EC%88%98%EB%AA%85-%EA%B0%99%EC%9D%84%EB%95%8C-%EC%83%9D%EB%9E%B5%EC%8B%9C-%EC%98%A4%EB%A5%98-%EB%B9%8C%EB%93%9C-%EC%84%A4%EC%A0%95%EC%9D%84-gradle%EB%A1%9C-%ED%95%98%EB%A9%B4-%ED%95%B4%EA%B2%B0%EB%90%98%EB%8A%94-%EA%B2%83-%EA%B0%99%EC%8A%B5%EB%8B%88%EB%8B%A4)