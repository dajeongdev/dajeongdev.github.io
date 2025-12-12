---
layout: post
title: Swagger UI에서 발생한 Failed to road remote configuration 원인 찾아내기
categories: wiki
subtitle: 
tags: [SpringBoot, Swagger, Springdoc]
---

### 배경

현재 작업중인 사이드 프로젝트에서 Springdoc 기반으로 Swagger를 사용하고 있다. 새로운 API 그룹을 만들기 위해 GroupedOpenApi 설정을 추가하면서 SwaggerConfig를 분리하다가, Swagger에서 화면이 갑자기 아래 이미지처럼 오류를 출력하며 로딩되지 않는 문제가 발생했다.

<img width="788" alt="github-actions2" src="https://dajeongdev.github.io/assets/images/posts/swagger-error.png">

혹시 몰라 Explore 버튼을 누르니 아래 같은 에러가 출력되었다.

<img width="788" alt="github-actions2" src="https://dajeongdev.github.io/assets/images/posts/swagger-error2.png">

처음에는 Springdoc 의존성과 Spring Boot 버전의 호환을 의심했는데, Springdoc의 스펙을 높이거나 Spring Boot의 버전을 낮춰봐도 문제는 그대로였다.
<br/>
<br/>


### 과정

Swagger UI는 기본적으로 다음 경로에서 화면에 띄워줄스펙(JSON)을 조회한다.

1. `/swagger-ui/**`
2. `/v3/api-docs/**`

그래서 다음 부분을 차례로 확인해봤다.

**1. GroupedOpenApi 설정**
- 경로도 확인하고, 해당 설정을 주석 처리해도 화면이 생성되지 않음
    
**2. YML에서 Springdoc의 paths-to-match 설정**
- paths-to-match에 `/api/**` 경로만 스캔하도록 설정해두었기 때문에 혹시 스펙 생성이 필터링 되었는지 컨트롤러를 확인해봤지만, 이것도 문제가 아니었음
<br/>
<br/>


### 원인 발견

오류 화면을 보다가 Network 탭을 볼 생각이 들어 확인해보니 요청 경로가 `/api-docs/swagger-config`였고, **403** 상태코드를 출력하고 있었다.

그리고 프로젝트에서는 Spring Security를 적용해두었는데, 허용 경로에는 위에 언급했던 default 경로뿐이었다. 즉, **Swagger UI 페이지 자체는 로딩**되었으나, **UI가 내부적으로 요청하는 실제 스펙 경로(`/api-docs/swagger-config`) 요청이 Security에 의해 차단**되고 있던 것이다.
<br/>
<br/>


### 해결

그래서 SecurityConfig에 누락된 `/api-docs/**` 경로를 허용하도록 추가했다.

**SecurityConfig.java**

```java
public class SecurityConfig implements WebMvcConfigurer {

    private final String[] permitUrls = new String[]{
            "/swagger-ui.html",
            "/swagger-ui/**",
            "/api-docs/**", // application.yml에 명시해둔 경로
            "/v3/api-docs/**",
    };
    
		@Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(authorizeRequest -> authorizeRequest // 인증/인가 URL 지정
                        .requestMatchers(permitUrls).permitAll() // 허용 URL
                        .anyRequest().authenticated()) // 나머지는 인증 필요
			...
        return http.build();
    }
}
```

**application.yml**

```java
springdoc:
  swagger-ui:
    ...
  api-docs:
    path: /api-docs # openAPI 접근 경로
```

추가 후에는 Swagger UI가 정상적으로 JSON 스펙을 로딩했고, 모든 API가 잘 표시되었다.
<br/>
<br/>


### 결론

이번 문제는 Swagger 설정 오류로 보였지만, 사실 커스텀으로 OpenAPI 접근 경로를 설정해놓고 SecurityConfig에는 누락한 게 원인이었다.

Springdoc에서 OpenAPI 기본 경로(`/v3/api-docs`)를 대신 커스텀 경로를 사용하는 경우에는 SecurityConfig에서도 반드시 동일한 경로를 허용해주어야 한다.

이번 케이스는 작은 실수였지만, 생각보다 시간을 잡아먹은 삽질이었다. 앞으로는 Swagger 관련 이슈가 있을 때는 YML 설정과 Security를 가장 먼저 검토해볼 것 같다.
<br/>
<br/>


#### 참고
- https://yundevnote.tistory.com/71