---
layout: post
title: 홈페이지 마이그레이션 회고
subtitle: 
categories: wiki
tags: [Java, Spring, SpringBoot, Maven, Gradle, JSP, Thymeleaf Migration, Review]
---
포트폴리오를 작성하다보니 생각보다 예전에 했던 프로젝트들이 회고해볼 만한 게 많아서 한 번 되짚어보려고 한다.
<br/>
<br/>


입사 당시 첫 주는 거의 일이 없어서 심심했었는데, 2주차에는 드디어 첫 업무를 받았다.
<br/>

Spring 4.3.3, Maven, JSP로 구성되어 있던 프로젝트를 Spring Boot, Gradle, Thymeleaf로 마이그레이션 하는 업무였는데, 마이그레이션 자체도 처음이고 Spring Boot와 Thymeleaf도 처음 써보는 거라 걱정반 기대반으로 조사부터 했었다.
<br/>

다행스럽게도 마이그레이션에 대한 레퍼런스가 많아서 찬찬히 읽어보고 시작했다.
<br/>

일단 Spring 4점대가 이미 개발 지원이 종료된 상태였고, 불필요한 중복 코드나 주석도 많아서 함께 리팩토링하기로 했다. 그리고 한꺼번에 바꾸게 되면 에러날 확률이 높다고 해서 하나씩 바꾸기로 했다.
<br/>
<br/>


#### 1. Spring → SpringBoot
spring boot 의존성도 추가하고, xml 파일들을 yml 혹은 config 파일로 분리했다. 

xml 파일이 적지 않아서 yml 문법에 맞게 옮기는 데 시간이 조금 걸렸고, java config 파일은 웬만하면 만들고 싶지 않아서 yml 문법으로 가능한 건 다 yml로 처리했었다.
<br/>

#### 2. Maven → Gradle
maven에서 gradle로 변경하는 건 생각보다 간단했는데, 일단 gradle을 설치하고 환경변수를 설정한 후 프로젝트 내에서 reload gradle project를 해주면 된다. 

그런데 빌드하면서 에러가 발생했는데 lombok 부분을 annotationProcessor로 변경해주니 금방 해결되었다.
<br/>

#### 3. JSP → Thymeleaf
당연하지만 3가지 순서 중에 가장 오래 걸린 부분이다.
<br/>

사실 이 부분은 내가 하고 싶어서 한 부분인데, JSP를 지양하는 문화로 가기도 했고 Spring Boot를 사용할 때 Thyemelaf를 권고한다고 해서 써보고 싶었다. Thymeleaf는 성능면에서 JSP보다 조금 느리지만, Jar 파일로 빌드할 수 있다. 그리고 무엇보다 HTML과 자바 코드가 섞이지 않아도 되는 점이 가장 마음에 들었다.
<br/>

Thymeleaf는 JSP와 인식되는 폴더 자체가 다르기 때문에 폴더 구조부터 수정한 뒤 jsp 파일을 html 파일로 변경했다. 

이후부터는 Thymeleaf 문법에 맞추는 수작업이었는데, Thymeleaf 설정에서 cache를 false로 설정해주면 브라우저에서 새로고침만으로도 바로 변경사항을 반영해볼 수 있어서 정말 편리했다.
<br/>
<br/>


#### 소감
사실 지금 생각해보면 아주 간단한 프로젝트였고, 모르는 게 있으면 질문을 정리해서 대리님께 물어봤기 때문에 금방 해낼 수 있었다.
첫 업무를 생각보다 빠르게 끝내서 기분 좋기도 했고(당연하지만 지금 하면 더 잘할 수 있을텐데),
회사에서도 뭔가 시켜도 되겠다 싶으셨는지 이후에 업무를 차근차근 받아서 진행하는 계기가 됐었다.
<br/>

지금 생각해보면 일하느라 바빠서 말도 많이 안했던 것 같은데 ㅋㅋ 조용히 열심히 하는 직원으로 생각해주셨으려나~