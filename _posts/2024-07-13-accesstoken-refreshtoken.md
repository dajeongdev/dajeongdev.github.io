---
layout: post
title: AccessToken과 RefreshToken
categories: wiki
subtitle: 
tags: [JWT, AccessToken, RefreshToken Network]
---
### AccessToken과 RefreshToken
`AccessToken`은 사용자에 대한 정보를 담아 서비스에 접근(Access)할 수 있는 토큰이다.

`RefreshToken`은 `AccessToken`과 다른 특별한 기능은 없지만, `AccessToken`을 새로 발급해주는 용도로 사용한다.
<br/>
<br/>


#### AccessToken의 문제
`AccessToken`만 사용하는 기존의 인증 방식은 토큰이 탈취되었을 때 문제가 발생한다. 

또 이러한 상황에서도 내부에서는 해당 토큰이 탈취되었는지 구분할 수 없다. 그래서 유효기간이 필요한 것이다.

그렇다고 유효기간이 너무 짧아도 사용자가 자주 로그인해야 하는 번거로움이 생기고, 유효기간을 늘리는 것 또한 보안에 취약할 수밖에 없다.
그래서 유효기간이 다른 `AccessToken`과 `RefreshToken` 2개를 사용하는 것이다.
<br/>
<br/>


#### RefreshToken 사용
`RefreshToken`은 `AccessToken`처럼 JWT를 사용하지만, AccessToken는 내부 데이터에 접근하고, `RefreshToken`은 `AccessToken`을 재발급하는 역할을 한다.

회원가입이나 로그인에 성공할 때 2가지 토큰을 모두 발급해준다. 
데이터베이스에는 `RefreshToken`을 저장하고, 클라이언트에는 `AccessToken`과 `RefreshToken`을 보내 쿠키/세션/스토리지에 저장하여 토큰이 필요한 API마다 헤더(보통 Authorization)에 담아서 보낸다.

`RefreshToken`은 `AccessToken`보다 유효기간이 길기 때문에, AccessToken이 만료되면 데이터베이스에 저장한 `RefreshToken`을 확인하여 다시 `AccessToken`을 재발급 해준다.
<br/>
<br/>

#### 발급 프로세스
1. 서버는 회원가입/로그인 성공 시 `AccessToken`과 `RefreshToken`를 발급하여 클라이언트에 전달한다.
    - 이때 `RefreshToken`은 데이터베이스에 저장해둔다.
2. 클라이언트는 전달받은 `AccessToken`과 `RefreshToken`을 로컬(쿠키/세션/스토리지 등)에 저장한다.
3. 클라이언트는 토큰이 필요한 API의 헤더(Authorization)에 `AccessToken`을 넣어 요청을 보낸다.
    - 서버에서는 인증된 사용자인지 먼저 검증한 후 요청 API의 로직을 수행하여 응답을 보낸다.
4. 로그아웃 API 요청 시 서버에서는 데이터베이스에 저장해둔 `RefreshToken`을 삭제한다.
<br/>
<br/>


#### 토큰 검증 분기

| AccessToken | RefreshToken | 결과 |
| --- | --- | --- |
| 유효 | 유효 | 정상 상태로 이후 로직 수행 |
| 만료 | 유효 | 클라이언트의 RefreshToken과 데이터베이스의 RefreshToken을 비교하여 AccessToken 재발급 |
| 유효 | 만료 | AccessToken을 검증하여 RefreshToken 재발급 |
| 만료 | 만료 | 재로그인을 요청하여 토큰을 새로 발급받도록 유도 |

<br/>

#### 참고
- [Access Token & Refresh Token 원리]([https://inpa.tistory.com/entry/WEB-📚-Access-Token-Refresh-Token-원리-feat-JWT](https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-Access-Token-Refresh-Token-%EC%9B%90%EB%A6%AC-feat-JWT))
- [Access Token과 Refresh Token이란 무엇이고 왜 필요할까?]([https://velog.io/@chuu1019/Access-Token과-Refresh-Token이란-무엇이고-왜-필요할까](https://velog.io/@chuu1019/Access-Token%EA%B3%BC-Refresh-Token%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%A0%EA%B9%8C))