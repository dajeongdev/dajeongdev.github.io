---
layout: post
title: Windows 명령어
subtitle: 
categories: wiki
tags: [Windows,command]
---

### 이동
``` powershell
cd [이동할 경로]
```

### 드라이브 변경
``` powershell
[드라이브 문자열]: // D:
```

### 현재 위치의 파일과 폴더 목록 조회
``` powershell
dir
```

### 네트워크 설정 상태 조회
``` powershell
ipconfig 
// all 옵션으로 상세 설정 정보 보기 가능
```

### 폴더 생성
``` powershell
md(or mkdir) [폴더명]
```

### 폴더 삭제
``` powershell
rm(rmdir) [폴더명]
```

### 환경 변수 path 조회
``` powershell
path
```

### 파일 삭제
``` powershell
del [파일명]
```

### 파일 복사
``` powershell
copy [복사할 파일명] [복사할 위치]
xcopy [복사할 파일명] [복사할 위치] // 숨김파일도 복사 가능
```

### 파일 이동 
``` powershell
move [파일명] [이동할 위치]
```

### 파일명/폴더명 변경
``` powershell
rename [현재 파일명/폴더명] [변경할 파일명/폴더명]
```

### 현재 날짜 조회
``` powershell
date
```

### 현재 실행 중인 프로세스 모두 조회
``` powershell
tasklist
```

### 명령 프롬프트 창 종료
``` powershell
exit
```