---
layout: default
title: Eclipse 초기 세팅
parent: JAVA 
grand_parent: TIL
---

# Eclipse 초기 세팅
{: .no_toc }

## TABLE OF CONTENTS
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Eclipse 단축키
이클립스는 VSCode에 비하면 정말 불편하고 불친절하다...  
그래서 그나마 편하게 하기 위해 몇 가지 단축키는 꼭 알아둬야 한다.
### 중요도⭐⭐⭐
* `ctrl + space`: 입력 중에 완성 기능 호출
* `ctrl + shift + O`: 자동 import / 필요 없는 import 삭제
* `ctrl + alt + ↑/↓`: 한 줄 복사
* `alt + shift + R`: 같은 변수명 전체 변경


### 중요도⭐⭐
* `ctrl + D`: 한 줄 삭제
* `ctrl + shift + F`: 소스 정렬
* `alt + ↑/↓`: 줄 이동
* `ctrl + N`: 새 파일/프로젝트 생성
* `ctrl + W`: 파일 닫기
* `shift + alt + J`: 이클립스 document 주석 달기


## Eclipse 초기 세팅
단축키보다 더 중요한 초기 세팅!  
자잘하게 불편한 점을 줄일 수 있다.

### 1. System.out.println(); 자동입력 되게 하기
* Windows -> Preferences -> Java -> Editor -> Templates 
* sysout 항목을 Java statement -> Java로 변경
* 이클립스 껐다 키기
* `sysout` 치고 `ctrl + space` 입력하면 자동완성!

### 2. Dynamic Web Project 생성하기
* Help -> Install New Software
* work with: Eclipse 버전 선택 -> Web, XML, Java EE and OSGi Enterprise Development
* 로딩이 다 끝난 후 이클립스 껐다 키기
* File -> New -> Project
* Web -> Dynamic Web Project 생성가능해진다!

### 3. 새로운 워크스페이스 UTF-8 설정하기
* Window -> Preferences -> encoding 검색
* Workspace 하단의 Text file encoding을 Other: UTF-8로 바꾼다
* 왼쪽 목록에서 CSS Files, HTML Files, JSP Files, XML Files 에서 죄다 UTF-8로 바꾼다
* 만약 톰캣 서버를 사용한다면 프로젝트 폴더 있는 곳에서 우클릭 -> New -> Other -> Server에서 톰캣 선택
