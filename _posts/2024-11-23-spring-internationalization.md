---
layout: post
title: Spring Boot에서 국제화로 언어 장벽 허물기!
categories: wiki
subtitle: 
tags: [Spring, SpringBoot, Internationalization, Message, 글또, 글또10기]
---
### 배경
최근 회사에서 진행하는 프로젝트가 글로벌 사용자를 타겟으로 두고 있어서 다국어 처리를 사용해 보게 되었는데요, 이에 대해 제가 알아본 방법과 작업하고 있는 방법을 정리해보겠습니다.
<br/>
<br/>


### Spring에서의 다국어 처리(국제화)
Spring에서는 다국어 처리에 대해 국제화라는 기능을 제공하고 있습니다. messages.properties 파일의 messages라는 이름 뒤에 Locale만 붙이면 내부에서 해당 파일을 찾아 처리해 줍니다.

**messages_ko.properties**
```java
greeting=안녕하세요
greeting_name=안녕하세요, {0}님
```

**messages_en.properties**
```java
greeting=hello
greeting_name=hello, {0}
```
<br/>

#### Locale
위에서 사용하는 로케일이란 무엇일까요? 로케일은 컴퓨터 시스템에서 특정 지역이나 문화권에 맞는 설정을 정의하는 개념으로, 주로 언어, 국가, 날짜와 시간 형식, 숫자 형식, 통화, 정렬 순서 등과 같은 지역화(Localization) 정보를 포함합니다.

**Locale 표기 방식 (UNIX/LINUX 시스템)**
```java
language[_territory][.codeset][@modifier]
```

한국어를 사용하는 경우 ko_KR.UTF-8을 주로 사용합니다. ko는 한국어, KR은 지역을 의미하고, 코드셋에는 UTF-8이나 EUC-KR 같은 인코딩 타입을 지정해 줍니다.  
일반적으로는 추가 문자열을 의미하는 @modifier 없이 언어, 지역, 코드셋 3개의 값으로 형성된 하나의 로케일을 사용합니다.
<br/>
<br/>


## 설정 방법

#### 1. application.yml(properties)
그래서 Spring에서 messages 파일을 어떻게 인식하는 걸까요? 기존에는 MessageSource를 직접 Bean으로 등록해야 했습니다. 그러나 SpringBoot에서는 따로 등록할 필요 없이 설정 파일(properties/yml)에서 설정할 수 있습니다. 이렇게 설정해두면 자동으로 `messages_*.properties`의 형식으로 추가한 파일들을 인식합니다.

```java
spring:
  messages:
    basename: messages/messages # 메시지 파일명, default messages
    encoding: UTF-8 # 인코딩 설정
    always-use-message-format: false # 시스템 로케일로 fallback 하지 않도록 설정
```

#### 2.LocaleInterceptor
현재 프로젝트에서는 경로에서 locale을 받아오기 때문에 로케일이 포함된 경로일 경우 로케일을 읽어 프로젝트의 기본 로케일로 설정해주고 있습니다.

```java
@Component
public class LocaleInterceptor implements HandlerInterceptor {

    @Getter
    private final String[] includePatterns = {"/api/**"};

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String requestURI = request.getRequestURI();
        // URL에서 locale 추출
        String[] pathSegments = requestURI.split("/");

        // locale이 포함된 경로가 /api/{locale}/... 일 때
        if (pathSegments.length > 2) {
            String localeStr = pathSegments[2];

            // locale이 유효한 형식일 때만 설정
            Locale locale = Locale.forLanguageTag(localeStr);
            Locale.setDefault(locale);
        }
        return true;
    }
}
```

🔼 그런데 지금 생각해보니 이렇게 처리하는 것보다 AcceptLanguage로 받는 게 더 낫지 않았을까 하는 생각이 듭니다. 프로젝트가 이미 어느 정도 진행 중이라 바꾸기 쉽지 않겠지만, 시도는 해보겠습니다.
<br/>

#### 3. SecurityConfig
작업한 환경에 따라 다르겠지만, 프로젝트에서 토큰을 사용 중이기 때문에 SecurityConfig에 2번에서 설정한 인터셉터를 추가해주었습니다.

```java
@RequiredArgsConstructor
@EnableWebSecurity
@Configuration
public class SecurityConfig implements WebMvcConfigurer {

  private final LocaleInterceptor localeInterceptor;

  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    // 다국어 인터셉터
    registry.addInterceptor(localeInterceptor).
        addPathPatterns(localeInterceptor.getIncludePatterns());
  }
  ...
}
```

#### 4. Controller/Service
저희 프로젝트에서는 다국어를 에러 메세지 위주로 사용하기 때문에 Service단에서 사용했는데요, 이또한 작업 환경에 따라 Controller에서도 사용할 수도 있습니다.

```java
messageSource.getMessage(EXISTING_EMAIL, null, locale);
messageSource.getMessage(EXISTING_EMAIL, new Object[]{"name"}, locale); // 동적 메세지 사용
```

보통 code, args, locale 세 가지의 파라미터를 사용한 getMessage() 메소드를 사용했습니다.

- **code**: messages에 key 값으로 넣어둔 문자열
- **args**: 동적 메세지를 사용하고 싶은 경우 값을 넣을 Object 배열
- **locale**: 프론트로부터 받거나 기본적으로 설정된 locale
<br/>
<br/>


#### 마무리
Spring에 대해 공부할 때부터 한 번쯤 제대로 경험해보고 싶었던 국제화 기능에 대해 자세히 알아보고 실무에서 바로 사용해보니 정말 재미있었습니다. 강력한 SpringBoot 덕분에 많은 설정 없이 원하는 언어별로 제공할 수 있네요.
<br/>
<br/>


#### 참고
- [다국어 처리의 모든 것](https://velog.io/@maketheworldwise/%EB%8B%A4%EA%B5%AD%EC%96%B4-%EC%B2%98%EB%A6%AC%EC%9D%98-%EB%AA%A8%EB%93%A0-%EA%B2%83)