---
layout: post
title: API 문서 자동화로 시간 절약하기 (feat.Swagger)
categories: wiki
subtitle: 
tags: [Spring, SpringBoot, Swagger, SpringAPIDocs, 글또, 글또10기]
---
#### 배경
백엔드 개발자에게 API란 무엇일까요. 열심히 설계하고 작성한 코드가 의미가 있으려면, 그걸 받아서 작업하는 프론트 또는 앱 개발자분들에게 잘 전달되어야겠죠. 그렇습니다. API를 잘 작성하는 것도 중요하지만, 그걸 잘 전달하는 것 또한 매우 중요합니다. 그래서 API를 잘 전달하기 위해 API 명세서를 작성하는 방법을 소개해볼까 합니다. 제가 실무에서 작업한 방법을 알려드리는 것이니, 더 좋은 방법이나 틀린 곳이 있다면 언제든지 댓글 부탁드리겠습니다.
<br/>
<br/>


### Swagger
#### Swagger란?

- Swagger는 API 명세서(문서)를 자동으로 생성 및 관리하기 위한 도구로, OpenAPI Specification(OAS)를 기반으로 API 설계 및 문서회에 활용할 수 있습니다.
- Swagger의 주요 장점
    - 개발 속도 향상
    - 문서와 API 코드 간의 동기화 가능
    - Swagger UI로 실시간 API 테스트 가능
<br/>
<br/>


#### Swagger 설치 및 설정 방법

**1. 의존성 추가**
- Swagger를 사용하기 위해 springdoc-openapi 라이브러리를 설치해주겠습니다.
- springfox는 더이상 지원하지 않으므로 사용을 권장하지 않습니다.

```java
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0'
```

**2 Swagger 설정 클래스 추가 (공통)**

```java
@OpenAPIDefinition(
        info = @Info(
                title = "Test API Document",
                version = "v1"
        )
)
@Configuration
class SwaggerConfig {

    @Bean
    public OpenAPI getOpenAPI() {
        return new OpenAPI()
                .components(new Components());
    }
}
```
<img alt="swagger-ui-common-setting" src="https://github.com/user-attachments/assets/4177ac6b-a378-4143-9a42-bcfb2ff081a5">

**3. 그룹별 Swagger 설정 클래스 추가**
- 그룹별로 API를 나눠서 제공하면 사용자가 보다 편리하게 문서를 이용할 수 있습니다.

```java
@Configuration
class UserSwaggerConfig {

    @Bean
    GroupedOpenApi userDocs() {
        return GroupedOpenApi.builder()
                .group("고객 API")
                .packagesToScan("com.test.test.user")
                .build();
    }
}
```
<img alt="swagger-ui-group-setting" src="https://github.com/user-attachments/assets/39bab756-d150-474a-ab4c-1330b385129d">
<br/>
<br/>


#### Swagger 어노테이션 사용 방법

**1. Controller**

```java
@Tag(name = "고객", description = "고객 관련 API입니다.")
@RequestMapping("/api/user")
@RequiredArgsConstructor
@RestController
class UserController {

    private final UserService userService;

    @Operation(summary = "summary : 이메일 인증번호 전송",
            description = """
                    ## 요청 :
                    - String email 이메일 (필수)
                    ## 응답 :
                    - String data "success"
                    """)
    @BasicApiSwaggerResponse
    @PostMapping("/send-email")
    ResponseEntity<CommonResponse<String>> sendEmail(@Valid @RequestBody SendEmailRequest request) {
        userService.sendEmail(request);
        return new ResponseEntity<>(new CommonResponse<>("이메일 인증번호 전송", MessageConstants.SUCCESS_MESSAGE), HttpStatus.OK);
    }
}
```

- **@Tag**: API 그룹 설정
    - name: 태그명
    - description: 태그에 대한 설명
- **@Operation**: API에 대한 설명
    - summary: API 요약
    - description: API 상세 설명  

**2. Request class**

```java
@Getter
public class SaveNoticeRequest {

    @NotNull
    @Schema(name = "language", description = "언어", example = "ko")
    private Language language;

    @NotNull
    @Schema(name = "category", description = "카테고리", example = "공지")
    private String category;
    
    @NotNull
    @Schema(name = "title", description = "제목", example = "제목")
    private String title;

    @NotNull
    @Schema(name = "content", description = "내용", example = "내용")
    private String content;
}
```

- **@Schema**: 해당 Schema에 대한 설명
    - name: 스키마 이름
    - description: 설명
    - example: 예시
- @Schema를 작성하여 요청값에 대한 정확한 설명을 적어주면 API 명세서가 더 완성도 있게 작성될 수 있습니다.
    - 이외에도 allowValues, defaultValue 같은 옵션이 있으니 잘 적용해보면 좋겠습니다.
<br/>
<br/>


#### Swagger의 장단점
**장점**
1. API 문서의 자동화
2. 협업 및 커뮤니케이션 강화
3. 실시간 테스트 및 디버깅 가능

**단점**
1. 코드와 문서가 따로 관리될 경우 실제 동작과 불일치 가능
2. 어노테이션으로 인해 코드 복잡성 증가
<br/>
<br/>


#### Swagger 실무 활용 팁
**1. @BasicApiSwaggerResponse 설정**
위의 코드 예제에서 사용한 것처럼  @BasicApiSwaggerResponse 같은 응답 상태 코드를 설명해주는 어노테이션을 만들어두고 사용하면 편리합니다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "API 호출 성공"),
        @ApiResponse(responseCode = "400", description = "잘못된 요청"),
        @ApiResponse(responseCode = "404", description = "존재하지 않는 API"),
        @ApiResponse(responseCode = "403", description = "인가 실패(권한 없음)"),
        @ApiResponse(responseCode = "500", description = "서버 에러")
})
public @interface BasicApiSwaggerResponse {}
```

**2. JWT 설정**

```java
@Configuration
class SwaggerConfig {

        private static final String TOKEN_PREFIX = "Bearer";
        
        @Bean
        public OpenAPI getOpenAPI() {
                String securityJwt = "JWT";
                SecurityRequirement securityRequirement = new SecurityRequirement().addList(securityJwt);
                Components components = new Components()
                        .addSecuritySchemes(securityJwt, new SecurityScheme()
                                .name(securityJwt)
                                .type(SecurityScheme.Type.HTTP)
                                .scheme(TOKEN_PREFIX)
                                .bearerFormat(HttpHeaders.AUTHORIZATION)
                        );

                return new OpenAPI()
                        .addSecurityItem(securityRequirement)
                        .components(components);
        }
}
```