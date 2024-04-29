---
layout: post
title: 아메리카노트 서버 구성 <4> 자동 배포
subtitle: 
categories: wiki
tags: [Server, NaverCloudPlatform, Project, Deploy]
---
---
layout: post
title: 아메리카노트 서버 구성 <4> 자동 배포
subtitle: 
categories: wiki
tags: [Server, NaverCloudPlatform, Project, Deploy, GithubActions]
---
- 프로젝트를 서버에 배포해봤다면 번거롭다는 생각을 한번쯤 해봤을 것 같습니다. 그래서 이번에는 그 배포를 자동으로 진행해보겠습니다.
- 자동 배포는 Github Actions의 self-hosted runner를 사용했습니다.
<br/>
<br/>


#### 1. 프로젝트 리포지토리에서 Github Actions Runner 생성하기
- 프로젝트 리포지토리로 이동하여 Settings - Actions - Runners - New self-hosted runner 화면으로 이동합니다.
<img width="1344" alt="github-actions1" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/c576842e-5e21-4301-a477-b84727d1d967">
<br/>
<br/>


#### 2. 서버에서 명령어 입력하기
- 서버에 Github Actions runner를 생성해주는 명령어들로 하나씩 차근차근 입력해 줍니다.
- 마지막 명령어 실행 중 **Must not run with sudo** 라는 에러 메시지가 발생하면 `export RUNNER_ALLOW_RUNASROOT="1”`를 입력한 뒤 재시도하면 됩니다.
<img width="788" alt="github-actions2" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/43dad816-a9ed-4eaa-b137-d76272e537b6">
- 설치 완료 이후 화면
<img width="671" alt="github-actions3" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/9bf0bd39-e89e-468b-9755-6debf9c54ac6">
- 이후 아래 명령어를 입력하면 Github Actions를 수신할 준비가 완료됩니다.
```bash
sudo ./svc.sh install
sudo ./svc.sh start
```
<br/>
<br/>


#### 3. 리포지토리에서 Actons workflow 생성
- 다시 리포지토리로 돌아가서 Actions 탭에서 왼쪽 상단의 New workflow를 클릭한 뒤 저희 프로젝트에 맞는 ‘Java with Gradle’을 선택하여 워크플로우를 생성합니다.
- 이 워크플로우가 바로 Github Actions가 서버에서 수행할 업무들입니다.
- 이전 페이지([<3> 프로젝트 배포](https://dajeongdev.github.io/wiki/2024/04/29/deploy-project.html))에서 직접 설치 및 작성한 것과 동일하게 진행합니다.
    - JDK 17 설치
    - YML 파일 생성
    - Gradle 빌드

```bash
name: Auto Deploy

on:
  push:
    branches:
      - develop

jobs:
  build:

    runs-on: self-hosted

    steps:
      # checkout
      - name: Checkout
        uses: actions/checkout@v4

      # JDK setting
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'zulu'

      # secret yml file 생성
      - name: Set secret yml
        run: |
          mkdir -p src/main/resources
          echo "${{ secrets.DATABASE_YML }}" | base64 --decode > src/main/resources/application-database.yml
          echo "${{ secrets.NAVER_TOKEN_YML }}" | base64 --decode > src/main/resources/application-naver-token.yml
          echo "${{ secrets.SECURITY_YML }}" | base64 --decode > src/main/resources/application-security.yml
          find src
        shell: bash

      # Gradle 권한 부여
      - name: Grant permission for gradlew
        run: chmod +x ./gradlew

      # 테스트 없이 빌드
      - name: Build with Gradle
        run: ./gradlew clean build -x test

      # 포트가 사용 중이라면 종료
      - name: Kill 8009 port if exist
        run: |
            if sudo lsof -i :8009; then
              echo "Port 8009 is already in use. Killing the process..."
              sudo kill -9 `sudo lsof -t -i:8009`
            fi

      # build/libs 폴더 속 jar 파일 실행
      - name: Execute Jar File
        run: |
          sudo nohup java -jar build/libs/americano-0.0.1-SNAPSHOT.jar 1>/root/nohup/output.log 2>/root/nohup/error.log &
          sleep 10
```
- secret yml 파일의 경우 별도로 필요한 yml들을 Base64로 인코딩하여 리포지토리 Settings → Secrets and variables → Actions에서 Repository Secrets로 설정해주었습니다.
<img width="1340" alt="github-actions4" src="https://github.com/dajeongdev/dajeongdev.github.io/assets/61612976/f2771b69-b97f-4255-9e94-6e3c1307921d">
- 또한, Github Actions self-hosted runner에서는 nohup이 적용되지 않아 자꾸 `Unable access to jarfile`가 발생하여, 결국 이 부분을 제거했더니 성공적으로 실행되었습니다.
    - ~~nohup이 적용되지 않는 부분은 추후 원인을 찾아내면 꼭 작성해두도록 하겠습니다!~~ **방법을 찾아서 추가하겠습니다. 또한 어이없게도 제가 nohup 이후에 설정한 게 없기 때문에 로그를 확인할 수 없어 필요하기도 했습니다..🥲**
        - nohup의 출력은 프로젝트 내에서 권한 밖이기 때문에 발생한 에러라고 확인이 됩니다. 그래서 프로젝트 밖의 경로로 설정하여 표준 출력과 에러를 구분하여 로그를 전달하는 방식으로 스크립트를 추가했습니다.      
   
```bash
// 이전
sudo nohup java -jar build/libs/americano-0.0.1-SNAPSHOT.jar &
// 현재
 sudo nohup java -jar build/libs/americano-0.0.1-SNAPSHOT.jar 1>/root/nohup/output.log 2>/root/nohup/error.log &
```
<br/>
<br/>
     

#### 참고
- https://e-room.tistory.com/145
- https://be-student.tistory.com/75
- [https://velog.io/@bjk1649/github-Action-만으로-지속적-배포CD-못하나](https://velog.io/@bjk1649/github-Action-%EB%A7%8C%EC%9C%BC%EB%A1%9C-%EC%A7%80%EC%86%8D%EC%A0%81-%EB%B0%B0%ED%8F%ACCD-%EB%AA%BB%ED%95%98%EB%82%98)
- https://green-joo.tistory.com/26
- [https://velog.io/@chlee4858/20211205-github에서-CICD-적용하기-github-action](https://velog.io/@chlee4858/20211205-github%EC%97%90%EC%84%9C-CICD-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-github-action)