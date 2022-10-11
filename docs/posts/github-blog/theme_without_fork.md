---
layout: default
title: 2. fork 할 필요 없이 블로그 테마 입히기
last_modified_date: 2022-10-10
parent: Github Blog
grand_parent: Posts
---

# fork 할 필요 없이 블로그 테마 입히기
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 들어가며
Github 블로그를 하기로 마음먹고 나서 다양한 블로그들을 찾아봤지만 거의 모든 블로그들이 원하는 블로그 테마의 Github repository를 fork해서 테마를 사용하더라. 그렇게 하면 내 repo가 너무 복잡해지고 보기 어려워서 fork 없이 블로그에 테마를 적용하는 법을 열심히 찾아봤다.  
나는 블로그 테마들 중 [just-the-docs](https://github.com/just-the-docs/just-the-docs)를 사용하고 있고 앞으로 블로그 커스터마이징을 할 때 just-the-docs를 기준으로 할 것이다. 어차피 코드를 뜯어보고 내가 직접 커스터마이징 할 것이기에 다른 테마에서도 응용할 수 있을 것 같다.


## 매번 기다릴 필요 없이 서버에서 페이지 구동 확인하기
* 터미널에 `bundle exec jekyll serve`를 입력하면 <http://localhost:4000/>에서 블로그 페이지가 어떻게 보이는지 확인할 수 있다. 매번 github에 commit하고 기다릴 필요가 없다!! 
* 만약 새빨간 에러글이 보이고 바로 아래에 `cannot load such file -- webrick (LoadError)`가 뜬다면 `bundle add webrick`을 통해 해결 가능하다.
* 로컬 서버에서 확인하는 것이기 때문에 내 블로그에 적용하고 싶으면 당연히 Github에 commit해야한다.


## 원하는 블로그 테마 찾기 
제일 중요한 단계나 다름 없다. 다양한 테마들이 있으니 꼼꼼히 찾아보자. 한번 정하고 커스텀 하면 바꾸기 어려울 테니...  
    * [깔끔한 사이트](https://jekyllthemes.io/free)  
    * [설명을 포함해서 정리한 사이트](https://jekyllthemes.dev/)  
    * [무료 테마 사이트](https://jekyll-themes.com/free/)  
    * [위랑 비슷한 사이트](http://themes.jekyllrc.org/)  


## 블로그 테마 입히기
1. 원하는 테마의 Github 페이지로 들어가면 Installation 가이드가 있을 것이다. 
![theme_1](../../../assets/images/theme_1.PNG)

2. 이 중 gem을 사용하는 방법을 이용할 것인데, `gem "just-the-docs"`와 `theme: just-the-docs`를 바꾸는 방법은 너무.. 너무 날것부터 시작하는 방법이다.(방금 1시간 삽질 해보고 깨달음)  
그래서 그 다음 방법을 사용할 것이다. 터미널에서 `gem install just-the-docs`를 실행해서 내가 원하는 테마의 gem을 다운받는다.

3. 이전에 루비를 설치할 때 별다른 설정을 하지 않았다면 `C:`에 루비가 설치되어 있을 것이다. 나의 경우는 `Ruby31-x64`라는 폴더에 설치가 되어있다.  
이제 방금 설치한 gem을 찾으러 갈 것이다. `C:\Ruby31-x64\lib\ruby\gems\3.1.0\gems` 위치로 가면 지금까지 설치한 gem들이 있는데 그 중 자신이 설치한 gem 폴더로 들어간다. (루비가 업데이트 된다면 경로가 바뀔 수도 있다)

4. 해당 폴더의 모든 내용을 복사해서 내 블로그 폴더에 붙여넣기 한다. 중복되는 파일이 있을 경우 덮어쓰기 해버린다.  
이 방법을 사용할 경우 테마의 기본 틀을 그대로 가져오는 것이다. 새로고침 해보면 테마가 적용된 것을 확인할 수 있다.
![theme_2](../../../assets/images/theme_2.png)


## 완성 소감
지금 내 블로그는 이미 어느정도 커스텀을 해버려서... 새로운 깃허브 계정 파고 이것저것 해봤다.
너무 뿌듯해서 완성 소감까지 쓸 정도..



## 참고한 사이트
* [just-the-docs](https://github.com/just-the-docs/just-the-docs)
* 블로그 테마 모음: <https://wiznxt.tistory.com/677>