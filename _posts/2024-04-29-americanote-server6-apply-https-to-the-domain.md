---
layout: post
title: 아메리카노트 서버 구성 <6> HTTPS 적용
subtitle: 
categories: wiki
tags: [NaverCloudPlatform, Project, MySQL, DataBase]
---
- 이전 단계에서 진행했던 방화벽에서의 80, 443 포트 허용과 도메인이 있다면 이제 기간은 짧지만 무료로 받을 수 있는 Letsencrypt의 SSL 인증서를 발급 받아보겠습니다.
<br/>
<br/>


#### SSL 인증서 발급받기
- 인증서를 발급받기 위해 letsencrypt의 certbot을 설치해주고, 인증서를 발급받습니다.

```bash
sudo apt update # 패키지 업데이트
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d 도메인이름 -d www.도메인이름
```

- 첫 실행 시에는 이메일을 입력하고 약관에 동의해야 합니다.
- 이후 certbot이 letsencrypt 서버와 통신 중 도메인이 유효한치 확인하는 과정을 거칩니다.
    - 저는 이 부분에서 에러가 참 많이 발생했는데요 ㅎㅎ 하나씩 살펴보겠습니다.
        1. **/well-known/acme-challenge/ 폴더가 허용되지 않았습니다.**
            - 이 부분은 Nginx에서 적용하는 부분이라 이전 단계에서 만들었던 `/etc/nginx/sites-available/도메인이름` 파일에서 추가해주었습니다.

                ```bash
                server {
                        listen 80 default_server;
                        listen [::]:80 default_server;
                
                        server_name 도메인이름 www.도메인이름;
                
                        location ~ /\.well-known/acme-challenge/ {
                        allow all;
                        root /var/www/letsencrypt;
                        }
                }
                ```
                
        2. **도메인에 A 레코드를 적용되어 있지 않았습니다.**
            - A 레코드 적용을 위해 NCP에서 Global Domain에 도메인을 추가하고, A 레코드를 추가해주었습니다.
                <img width="679" alt="https1" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/3abce503-be85-43fe-8531-612c95578743">
            - 하지만 이 Global Domain의 네임서버를 카페24의 도메인에 설정해주는 과정에서 24시간이 걸린다는 소식에 아주 멘탈이 탈탈 털릴 뻔 했지만… 다행히 12시간 정도 뒤에 사용이 가능했습니다😂
                <img width="544" alt="https2" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/8f6df708-93fa-4e43-8709-e2ad85b0c689">
            
<img width="770" alt="https3" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/f52a2ee6-ea91-475c-ba0e-8252c28a2993">
- 그리하여 드디어 간절히 기다리던 발급 성공 화면을 보게 되면 SSL 발급에 성공한 겁니다.
- 이제 `/etc/nginx/sites-available/도메인이름` 파일에 접근해보면 자동으로 SSL이 적용된 것을 확인할 수 있습니다.

```bash
server {
        listen  443 ssl;
        server_name americanote.store www.americanote.store;

        #access_log /var/log/nginx/proxy/access.log;
        #error_log /var/log/nginx/proxy/error.log;

        ssl_certificate /etc/letsencrypt/live/americanote.store/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/americanote.store/privkey.pem;

        location / { # 추가
                proxy_pass http://127.0.0.1:8009;

        }
}
```

- 참고로 `location / { … }` 부분까지 설정해주어야 현재 서버의 8009로 실행되고 있는 프로세스로 프록시가 요청을 전달해줍니다.
<br/>
<br/>


#### 참고
- [https://velog.io/@curiosity806/nginx-Lets-Encrypt-SSL-인증서-발급](https://velog.io/@curiosity806/nginx-Lets-Encrypt-SSL-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%B0%9C%EA%B8%89)
- [Ubuntu 20.04에서 Nginx 설치 및 설정하기](https://jaehyeon48.github.io/nginx/configure-nginx-on-ubuntu-2004)