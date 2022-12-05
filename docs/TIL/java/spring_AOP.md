---
layout: default
title: Spring AOP
parent: JAVA
grand_parent: TIL
---

# Spring AOP

{: .no_toc }

## TABLE OF CONTENTS

{: .no_toc .text-delta }

1. TOC
   {:toc}

---
## AOP(Aspect Oriented Programming)

- OOP에서 모듈화의 핵심 단위는 클래스, AOP에서 모듈화의 단위는 Aspect
- Aspect는 여러 타입과 객체에 거쳐서 사용되는 기능의 모듈화를 한 것
- AOP 프레임워크는 Spring IoC를 보완한다.

## AOP 용어

- Aspect: 하나의 기능. 공통 관심사 코드
- Join Point: 수많은 후보. 메서드 호출
- Pointcut: Join Point 중 내가 선택한 하나
- Advice: 어떤 Aspect가 어떤 Pointcut의 before냐 after냐까지 정의한 것
- Target: Advice가 적용되고자 하는 대상
- AOP Proxy: Advice가 Target에 적용되어 만들어진 완성체
- Weaving: Advice가 Target에 끼어드는 동작 자체

## AOP 구현 방식
* XML 방식
  * xml 파일에 AOP 설정을 하는 것
  * `<aop:config>` 안에 `<aop:pointcut>`, `<aop:aspect>`를 선언한다
* Annotation 방식
  * Annotation(`@`)을 이용해 하나의 클래스에서 AOP 설정을 하는 것
  * `MyAspect` 클래스에 `@Component`, `@Aspect` 설정 후 클래스 요소로 `@Pointcut`과 다양한 Advice Type들을 선언한다
* Java Config 방식
  * `ApplicatoinConfig` 클래스 안에 Bean을 선언하는 방식
  * `@Configuration` annotation을 설정한 클래스 안에 `@Bean`을 설정한 메소드를 생성한다
  
## Advice Type
* before: target 메소드 호출 이전
* after: target 메소드 호출 이후. java exception 문장의 finally와 동일하게 동작
* after-returning: target 메소드 정상 동작 후
* after-throwing: target 메소드 에러 발생 후
* around: target 메소드의 실행 시기, 방법, 실행 여부를 결정