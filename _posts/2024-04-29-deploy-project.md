---
layout: post
title: 아메리카노트 서버 구성 <3> 프로젝트 배포
subtitle: 
categories: wiki
tags: [Server, NaverCloudPlatform, Project, Deploy]
---
서버와 데이터베이스를 생성했다면 이번에는 서버에 프로젝트를 배포해보겠습니다.
<br/>
<br/>


#### 1. 서버에 접근하기
```bash
ssh username@public-ip -p 22(허용 포트)
# password
```
<br/>


#### 2. 서버에 JAVA 설치
```bash
sudo apt install openjdk-17-jdk
java --version # 자바 버전 확인
```
<br/>


#### 3. 프로젝트 가져오기
```bash
git clone --branch 가져올브랜치 https://github.com/깃헙아이디/레포지토리명.git # 프로젝트 복제
```
<br/>


#### 4. 별도로 필요한 yml 파일이 있다면 (optional)
```bash
cd Americanote # 리포지토리로 들어가기
cd src/main/resources # yml 파일이 위치할 resources 폴더로 이동
cat > ~.yml # yml 파일 생성 및 편집
# yml 파일 내용 붙여넣기
# 필요한 yml 파일을 모두 만들었으면 다시 리포지토리 홈으로 돌아갑니다.
```
<br/>


#### 5. Gradle 빌드 후 실행하기
```bash
./gradlew clean build -x test # 테스트 없이 처음부터 빌드
cd build/libs # jar 파일이 존재하는 폴더로 이동
ll 혹은 ls # 폴더 보기
nohup java -jar [이름-SNAPSHOT.jar] &
```
- nohup
    - no hang up의 약자, 말그대로 끊지마!
    - 사용자가 세션과 연결을 종료해도, 데몬 형태로 실행하게 하여 해당 프로세스가 종료되지 않습니다.
- &
    - 프로그램 실행 시 마지막에 붙여주면 백그라운드에서 해당 프로세스가 실행됩니다.
<br/>
        → 어떤 프로그램을 종료없이 백그라운드에서 실행시킬 때 `nohup ~ &` 조합을 사용하는 게 가장 확실하고 안전합니다.
<br/>
<br/>


#### 6. 로그 확인
```bash
# 실행 후 ll 이나 ls로 확인해보면 nohup.out이 생성됩니다.
tail -f nohup.out # 실시간 업데이트 로그
cat nohup.out     # 현재까지 로그
```
- 제대로 실행되었는지 확인하기 위해 로그를 실행 후 로그를 반드시 확인하는 편입니다.
<br/>
<br/>


#### 7. 만약 이미 프로젝트가 실행 중이라면? 종료하고 실행하자
```bash
netstat -tulpn # 실행 중인 프로세스 확인
kill -9 [PID 번호]
```
- Gradle 빌드 후 실행하기 전에 한번쯤 확인하면 좋겠습니다.
<br/>
<br/>


#### 참고
- [https://1.theapplemango.com/entry/네이버-클라우드-플랫폼NCP-서버-구축-및-스프링-배포까지-2](https://1.theapplemango.com/entry/%EB%84%A4%EC%9D%B4%EB%B2%84-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%ED%94%8C%EB%9E%AB%ED%8F%BCNCP-%EC%84%9C%EB%B2%84-%EA%B5%AC%EC%B6%95-%EB%B0%8F-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B0%B0%ED%8F%AC%EA%B9%8C%EC%A7%80-2)
- [https://joonyon.tistory.com/entry/쉽게-설명한-nohup-과-백그라운드-명령어-사용법](https://joonyon.tistory.com/entry/%EC%89%BD%EA%B2%8C-%EC%84%A4%EB%AA%85%ED%95%9C-nohup-%EA%B3%BC-%EB%B0%B1%EA%B7%B8%EB%9D%BC%EC%9A%B4%EB%93%9C-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%82%AC%EC%9A%A9%EB%B2%95)