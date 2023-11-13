---
layout: post
title: Thymeleaf와 @RestController
subtitle: 
categories: wiki
tags: [Java,Spring,Annotation,Thymeleaf]
---

# Thymeleaf와 @RestController

## 개요
테스트를 해보려고 테스트 컨트롤러에 @RestController을 설정한 후 String으로 view 이름을 리턴했더니 화면에 입력한 view 이름이 그대로 나왔다.
```java
@RestController
public class TestController {

    @GetMapping("/sample")
    public String hello() {
        return "sample";
    }
}
```
<br/>


### 왜 그럴까?
- `@RestController`에는 `@Controller` 애노테이션과 `@ResponseBody` 애노테이션이 포함되어 있다. Controller에서 페이지 렌더링과 Json 데이터를 함께 리턴해야 할 때 사용한다.
- `@Controller`는 리턴 값이 String이라면 view 이름으로 인식하여 view를 찾아 렌더링 한다.
    - 페이지 이동(`@Controller`), Json 데이터 리턴(`@ResponseBody`)
- `@ResponseBody`: 자바 객체를 HTTP 요청의 body 내용으로 매핑하는 역할을 하는 애노테이션
<br/>


### 형태
`@Controller`
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller
```
`@RestController`
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController
```
<br/>


### 해결 방법
`ModelAndView`를 사용하여 `setViewName()`에는 이동할 view 이름, `addObject()` 등에는 사용할 변수명을 입력해주면, `@RestController`나 `@Controller` 애노테이션 둘 다 상관없이 동작한다.