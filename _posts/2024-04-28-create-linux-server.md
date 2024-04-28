---
layout: post
title: 아메리카노트 서버 구성 <1> Linux 서버 생성
subtitle: 
categories: wiki
tags: [Server, NaverCloudPlatform, Project, Linux]
---
#### 배경
- 비사이드 포텐데이403 프로젝트를 진행하면서 Linux 서버를 생성하기 위해 순서대로 정리해봤습니다.
<br/>
<br/>


#### 1. OS 선택
- 레퍼런스가 많고 지속적으로 업데이트가 되고 있는 **Ubuntu**로 선택함
<br/>
<br/>


#### 2. VPC(Virtual Private Cloud) 설정
- 전용 네트워크
- 필수
    - 이름, IP 주소 범위
<br/>
<br/>


#### 3. Subnet 설정
- VPC 내에 세분화된 격리 공간 제공
- 필수
    - 이름, VPC, IP 주소 범위(VPC 범위 내), 가용 Zone, Network ACL, Internet Gateway 전용 여부(public), 용도
<br/>
<br/>


#### 4. 서버 생성
- 서버 타입: Standard
- 서버 스펙: s2-g3(vCPU 2EA, Memory 8GB)
- 스토리지: CB1 (기본 스토리지, 10GB)
- 서버 인증키(pem)
	- 잘 보관해두기
<br/>
<br/>


#### 5. ACG(방화벽) 설정
- 22
    - 접속할 PC
    - IP 입력
    - **팀원들 IP 입력 필요** ⭐
- 80, 443
    - URL 링크로 접속
    - 클라이언트를 위한 포트
    - 80: HTTP
    - 443: HTTPS
- 81
    - Nginx Proxy Manager의 접속을 위한 포트
- 3306
    - DB 접속
<br/>
<br/>


**참고**
- https://velog.io/@ejh990521/CentOS-vs-Ubuntu
- https://docs.3rdeyesys.com/compute/ncloud_compute_server_vpc_create.html