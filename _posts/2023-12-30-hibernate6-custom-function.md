---
layout: post
title: hibernate6 custom function(MySQL Dialect)
subtitle: 
categories: wiki
tags: [Hibernate6, MySQLDialect, SpringBoot3, QueryDSL, JPQL]
---

### 배경
현재 프로젝트에서 QueryDSL을 사용 중인데 여기에 집계함수를 사용하고 싶어서 사용자 함수 추가를 위해 서치 후 적용해보려고 했다. 그런데 이게 웬일.. Spring boot 3점대에서는 hibernate 6.1을 기본으로 사용하고 있어서 당장 올해 3월까지 올라온 블로그 글의 코드도 적용되지 않았다.. 그래서 한시간을 고생해서 적용해보고 고치고 하다가 김영한님 JPA 강의의 커뮤니티에 어떤 친절한 분이 해결방안을 올려주셨다.. 정말 천사,,

<img src="https://dajeongdev.github.io/assets/images/posts/spring-boot-3.2-hibernate-version.png">
[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M4-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M4-Release-Notes)
<br/>
<br/>   

### 해결
1.  FunctionContributor의 구현체를 만들어준다.
    ```java
    public class CustomFunctionContributor implements FunctionContributor {
    
        @Override
        public void contributeFunctions(FunctionContributions functionContributions) {
            functionContributions.getFunctionRegistry()
                .register("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
        }
    }
    ```
2.  src/main/resources 하위에 META-INF/services/org.hibernate.boot.model.FunctionContributor 파일을 생성한다.
3.  해당 파일에 1번의 구현체를 등록한다.
    
    -  `패키지명.구현체` 이름의 형태로 등록
    - com.example.com.config.~Contributor
-   yml 파일의 dialect 변경은 필요없음
<br/>   
<br/>   
   
   
### 시도
-   [https://discourse.hibernate.org/t/migrate-hibernate-5-to-6-with-spring-boot-2-7-x-to-3/7787/2](https://discourse.hibernate.org/t/migrate-hibernate-5-to-6-with-spring-boot-2-7-x-to-3/7787/2)
-   [https://discourse.hibernate.org/t/migration-of-dialect-to-hibernate-6/6956/16](https://discourse.hibernate.org/t/migration-of-dialect-to-hibernate-6/6956/16)
-   [https://thisiswoo.github.io/development/using-jpa-querydsl-groupconcat-func.html](https://thisiswoo.github.io/development/using-jpa-querydsl-groupconcat-func.html)
<br/>
<br/>   

   
**참고**
-   [](https://www.inflearn.com/questions/1096265/hibernate-6-custom-%ED%95%A8%EC%88%98-%EB%93%B1%EB%A1%9D-%EB%B0%A9%EB%B2%95-%EA%B3%B5%EC%9C%A0)[https://www.inflearn.com/questions/1096265/hibernate-6-custom-함수-등록-방법-공유](https://www.inflearn.com/questions/1096265/hibernate-6-custom-%ED%95%A8%EC%88%98-%EB%93%B1%EB%A1%9D-%EB%B0%A9%EB%B2%95-%EA%B3%B5%EC%9C%A0)
-   [https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M4-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M4-Release-Notes)
-   [](https://velog.io/@ttomy/%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%95%EC%9D%98-dialectmatch-against%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)[https://velog.io/@ttomy/사용자-정의-dialectmatch-against사용하기](https://velog.io/@ttomy/%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%95%EC%9D%98-dialectmatch-against%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)