---
layout: post
title: 아메리카노트 서버 구성 <5> 도메인 연결
subtitle: 
categories: wiki
tags: [Server, NaverCloudPlatform, Project, DNS]
---
- 자동 배포까지 했으니 어느정도는 백엔드 코드를 서버에 올리는 것까지 했다고 할 수 있겠습니다.
- 저도 여기가 마지막인 줄 알았으나..
![domain1](https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/9ba32892-8688-4f23-aa75-806a8df95b37)
- 첨부한 이미지는 프론트분이 API가 요청되지 않는다고 캡쳐해주신 에러 메세지입니다. 읽어보면 HTTPS인 프론트 도메인에서 HTTP인 백엔드 도메인으로 요청하는 부분에서 XMLHttpRequest 에러가 발생했습니다. 바로 CORS 문제입니다 ㅎㅎ
- 검색해보니 프론트 쪽에서 간단하게 `<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">` 같은 HTML 코드를 넣어서 해결하는 방법도 있었지만, 그래도 저희는 해결되지 않아서 결국 도메인을 구매하기로 결정했습니다. (심지어 마감 전날 토요일에 ㅎㅎ)
<br/>
<br/>


#### 1. 카페24에서 도메인 구매
- 아주 익숙한 카페24에서 도메인을 구매하기로 결정했습니다.
- 좋은 이름이면 참 좋겠지만 프론트에서 API로 요청할 때만 필요하기 때문에 1년에 단돈 500원에 구매할 수 있는 `.store`를 선택했습니다.

<img width="875" alt="domain2" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/b015dcca-441f-4fe1-9067-0577379b6bc6">
<br/>
<br/>


### 2. 서버에 Nginx 설치하기
- 이제 도메인과 서버를 연결해주기 위해서 서버에 nginx를 설치해보겠습니다.
- 먼저 설치하기 전에 패키지 목록을 업데이트 해줍니다.
```bash
sudo apt update
sudo apt upgrade
```
- 그리고 nginx를 설치하겠습니다.
```bash
sudo apt install nginx
```
- 계속할지 묻는 메시지가 표시되면 ‘Y’를 누르고 enter 키를 치다보면 설치가 완료됩니다.
    <img width="970" alt="domain3" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/4943ca29-1701-4786-9015-11ec1efd5ed1">
	- 저는 이렇게 하다가 중간에 에러가 발생했는데요, 이유는 IPV6 설정때문이었습니다. 해결해주기 위해 `/etc/nginx/sites-available/default` 파일에 접근해서 아래처럼 수정해줍니다.
    ```bash
    server {
            listen 80 default_server;
            # listen [::]:80 default_server; # 이 부분 주석 처리
    
            ...
    
            root /var/www/html;
    
            # Add index.php to the list if you are using PHP
            index index.html index.htm index.nginx-debian.html;
    
            server_name _;
    
            location / {
                    # First attempt to serve request as file, then
                    # as directory, then fall back to displaying a 404.
                    try_files $uri $uri/ =404;
            }
    				...
    }
    ```
    
- 이제 설치를 마쳤으면 nginx를 시작합니다.
```bash
sudo service nginx start
sudo service nginx status # nginx 상태 확인
```
<img width="1045" alt="domain4" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/5447c65d-2df6-4850-945c-e19f8dd219b6">
<br/>
<br/>


#### 3. 방화벽 설정
```bash
sudo ufw app list
```
- 위의 명령어를 입력하여 방화벽 설정을 확인하면 방화벽 설정이 가능한 애플리케이션 목록이 나옵니다.
```bash
Available applications:
  Nginx Full # 80, 443 포트 허용
  Nginx HTTP # 80 포트만 허용
  Nginx HTTPS # 443 포트만 허용
  OpenSSH
```
- 저희는 이 다음에 HTTPS까지 적용하기 위해서 ‘Nginx Full’ 방화벽을 허용해줍니다.
```bash
sudo ufw allow 'Nginx Full'
sudo ufw status # 상태 체크
```
```bash
Status: inactive

To                         Action      From
--                         ------      ----
Nginx HTTP                 ALLOW       Anywhere
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```
- 그리고 위처럼 상태가 inactive로 되어있으면 아래 명령어를 입력하여 활성화 시켜줍니다.
    - ***tip***: 아래 명령어를 입력하면 ssh 연결을 방해할 수도 있다는 경고 문구가 나오는데, 이때 y를 하기 전에 반드시 22번 포트를 허용하거나 제한해줍니다.
    <img width="608" alt="domain5" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/429317de-249e-49fb-b516-7ebe01647d45">
    ```bash
    sudo ufw limit/allow 22
    ```
    - 처음 설정할 때는 이 부분을 몰라서 ssh에 들어갈 수 없게 되는 사태가 벌어졌지만, NCP 서버 콘솔에 원격으로 접속하는 곳으로 들어가 22번 포트를 허용하는 방식으로 겨우 해결했습니다…💦💦💦
```bash
sudo ufw enable
```
- 이제 public ip로 접근해보면 nginx 화면이 성공적으로 뜨는 것을 확인할 수 있습니다.

<img width="745" alt="domain6" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/d9ee761f-1926-4fce-82dd-85382f1eb292">
<br/>
<br/>


#### 4. 도메인 연결
- public ip로 뜨는 걸 봤으니 이제 도메인과 연결해주겠습니다.
- 먼저 기본 nginx의 화면이 아닌 다른 화면으로 출력하기 위해 index.html 파일을 만들어줍니다.
```bash
sudo mkdir -p /var/www/도메인이름/html # 폴더 생성
sudo chown -R $USER:$USER /var/www/도메인이름/html # 소유자 설정
sudo chmod -R 755 /var/www/도메인이름 #권한 설정
sudo vi /var/www/도메인이름/html/index.html # index.html 파일 생성
```
- 위에서 수정했던 `/etc/nginx/sites-available/default` 파일 대신 이번에는 도메인 이름으로 된 파일을 만들어서 아래처럼 설정해줍니다.
```bash
sudo vi /etc/nginx/sites-available/도메인이름
# 위 파일로 자동으로 이동한 뒤 아래 내용 입력
server {
    listen 80;

    root /var/www/도메인 이름/html; # 위에서 만든 html 폴더
    index index.html index.htm index.nginx-debian.html;

    server_name 도메인 이름 www.도메인 이름;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
- 위 파일을 nginx에서 연결해주기 위해 해당 파일을 sites-enabled 폴더에 심볼릭 링크 설정을 해줍니다.
```bash
sudo ln -s /etc/nginx/sites-available/도메인이름 /etc/nginx/sites-enabled/
```
- 이제 브라우저에서 도메인을 입력하면 위에서 생성한 index.html이 확인할 수 있습니다.
<br/>
<br/>


#### 참고
- https://jaehyeon48.github.io/nginx/configure-nginx-on-ubuntu-2004/
- [https://velog.io/@xangj0ng/Linux-Ubuntu-Nginx-설치](https://velog.io/@xangj0ng/Linux-Ubuntu-Nginx-%EC%84%A4%EC%B9%98)