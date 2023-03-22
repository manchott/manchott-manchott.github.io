---
layout: default
title: SpringBoot 테스트코드 작성하기
parent: AWS
grand_parent: TIL
last_modified_date: 2022-12-27
---

# SpringBoot 테스트코드 작성하기
{: .no_toc }

## TABLE OF CONTENTS
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# 테스트코드 작성하기
스프링, 스프링부트를 학습하며 TDD, 단위 테스트 등등을 많이 들었다. 실제로 스프링 프로젝트를 시작하면 아예 test 폴더가 따로 있을 정도니까.  
그런데 *스프링 부트와 AWS로 혼자 구현하는 웹 서비스* 책에서는 처음부터 테스트코드를 작성하는 법을 알려준다!  
이번 기회에 제대로 학습해볼 예정이다. 책에서는 JUnit4를 쓰던데 최근에는 JUnit5를 많이 쓰는듯해서 5를 기준으로 학습할 것이다.

