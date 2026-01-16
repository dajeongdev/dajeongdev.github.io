---
layout: post
title: Docker + Github Actions 기반 자동 배포 구축하기
categories: wiki
subtitle: 
tags: [CICD, Docker, GithubActions]
---

### 배경

프로젝트가 절반 정도 진행되니 수동 배포하는 게 생산성이 너무 떨어진다는 걸 체감하기 시작했다.
그래서 이전처럼 Github Actions 기반의 자동 배포를 하려고 했지만, 이왕 하는 김에 요즘 많이 사용하는 Docker + Github Actions 조합을 적용해보기로 했다.
<br/>
<br/>


### 과정
#### 1. 로컬에서 먼저 Docker 확인하기
자동 배포 전에 Docker 실행 흐름을 알고 싶어 로컬에서 먼저 확인했다.
- [Docker](https://www.docker.com/products/docker-desktop/) 설치
- **⭐️ Dockerfile는 반드시 프로젝트 루트에 위치**
   
    ```docker
    FROM amazoncorretto:21.0.4
    WORKDIR /app 
    
    ARG JAR_FILE=build/libs/*.jar
    COPY ${JAR_FILE} app.jar
    
    ENTRYPOINT ["java", "-jar", "app.jar"]
    ```
- 로컬에서 Dockerfile 실행
    
    ```bash
    # jar 생성
    ./gradlew build
    
    # 이미지 생성
    docker build -t <사용할 이미지 이름> . # 현재 디렉토리 전체를 빌드 컨텍스트로 사용
    
    # 이미지 확인
    docker images
    
    # 컨테이너 실행
    docker run -d -p <포트번호>:<포트번호> --name <컨테이너 이름> <이미지 이름>
    ```
    
    - 여기까지 진행하면 로컬에서 docker를 성공적으로 띄웠다는 걸 확인할 수 있다.
<br/>
<br/>


#### 2. docker-compose.yml 생성
- docker-compose.yml 파일은 여러 개의 `docker run` 명령어를 파일 하나로 관리해준다.
- 하지만 매번 아래처럼 명령어를 입력해서 실행시키는 게 위험하고 귀찮았다.
    ```docker
    docker run -d -p <포트번호>:<포트번호> -e <프로파일 설정> --name <컨테이너 이름> <이미지 이름>
    ```
- 그래서 docker-compose.yml로 관리하도록 변경했다.
    ```yaml
    services:
      app:
        image: <이미지 이름>
        container_name: <컨테이너 이름>
        ports:
          - "<포트번호>:<포트번호>"
        environment:
          SPRING_PROFILES_ACTIVE: prod
    ```
    
    **실행**
    
    ```bash
    docker compose up -d
    ```
    
    **중지**
    
    ```bash
    docker compose down
    ```
<br/>
<br/>


#### **3. 민감 정보 분리 (application-secret.yml → .env)**

**기존 방법**
- `application-secret.yml`을 `.gitignore` 처리
- 자동 배포 시 서버에 동일한 설정을 다시 만들어야 함

**개선 방향**
- application-secret.yml에는 환경 변수 placeholder만 남김
- 실제 값은 `.env` 파일과 Github Repository Variables로 관리

**application-secret.yml**
```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

**.env**
```
DB_URL=...
DB_USERNAME=...
DB_PASSWORD=...
```

**docker-compose.yml**
```yaml
services:
  app:
    image: <이미지 이름>
    container_name: <컨테이너 이름>
    ports:
      - "<포트번호>:<포트번호>"
    environment:
      SPRING_PROFILES_ACTIVE: prod
    env_file:
	    - .env
```
<br/>
<br/>


#### 4. 서버에 Docker 설치
- Ubuntu 기준으로 Docker를 설치했다.

```bash
# 기존 패키지 업데이트
sudo apt update
sudo apt upgrade -y

# 필요 패키지 설치
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common lsb-release

# Docker GPG key 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Docker repository 추가
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker 설치
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Docker 권한 설정 (sudo 없이 실행)
sudo usermod -aG docker $USER

# Docker Compose 설치
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 버전 확인
docker-compose --version

# Docker Compose로 실행
docker compose up -d --build

# 로그 확인
docker compose logs -f
```
<br/>
<br/>


#### 5. Github Actions 자동 배포

**전체 흐름**
1. feature 브랜치에 push
2. self-hosted runner에서 실행
3. Gradle 빌드
4. Docker 이미지 빌드
5. 기존 컨테이너 종료
6. docker compose 재실행

**진행**
- 서버에 먼저 Github Actions Runner 설치한다.
- 그리고 프로젝트 루트에 /.gihub/workflows 폴더 만들고, 배포할 yml 파일을 작성한다.

```yaml
name: Auto Deploy

on:
  push:
    branches:
      - feature

jobs:
  build:

    runs-on: self-hosted
    environment: prod

    steps:
      # 코트 체크아웃
      - name: Checkout
        uses: actions/checkout@v4

      # JDK 설정
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'zulu'

      # env 파일 생성
      - name: Create prod env file
        run: |
          mkdir -p /home/ubuntu/project/env
          
          cat <<EOF > /home/ubuntu/project/env/prod.env
          SPRING_PROFILES_ACTIVE=prod
          
          # ... .env 파일에 Variables 매핑 설정
          EOF
          
          chmod 600 /home/ubuntu/project/env/prod.env

      # Gradle 권한 부여
      - name: Grant permission for gradlew
        run: chmod +x ./gradlew

      # 테스트 없이 빌드
      - name: Build with Gradle
        run: ./gradlew clean build -x test

      # 기존 컨테이너 중지 및 제거
      - name: Stop & Remove Existing Containers
        run: |
          docker compose down
          echo "Existing containers stopped and removed."

      # 컨테이너 실행
      - name: Run Docker Compose
        run: |
          docker compose up -d --build
          echo "Docker containers running."
```
<br/>
<br/>


### 💫 트러블슈팅

#### 1. docker-compose.yml 위치
- 처음에는 `docker-compose.yml` 파일이 설정 파일 같아서 /src/main/resources 폴더에 넣어뒀더니 계속 에러가 발생했다.
- 여러 번의 에러를 통해 Dockerfile이 있는 프로젝트 루트에 넣어야 한다는 걸 알았다.
<br/>

#### 2. private key과 .env 파일의 값 매핑 이슈
**문제 상황**
1. 애플 로그인의 private key 값을 기존 application-secret.yml에서는 멀티라인으로 사용
2. 하지만 `.env` 파일은 멀티라인 미지원
<br/>

**시도**
1. 줄바꿈을 \n로 치환
2. Base64 인코딩
3. BEGIN / END 구문 제거 후 인코딩
4. 줄바꿈을 \n로 치환 + BEGIN / END 구문 제거 → 성공  
<br/>

**정리**  
겨우 해결하고 나니 이게 안전한 방법인지 의문이 들어서 다음에는 AWS Parameter Store / Secrets Manager나 GitHub OIDC + 외부 Secret 관리 같은 방식도 검토해보려고 한다.