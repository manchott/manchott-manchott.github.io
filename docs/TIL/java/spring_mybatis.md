---
layout: default
title: Spring에 MyBatis 연동하는 프로젝트 만들기
parent: JAVA 
grand_parent: TIL
last_modified_date: 2022-12-06
---

# Spring에 MyBatis 연동하는 프로젝트 만들기
{: .no_toc }

## TABLE OF CONTENTS
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## board를 만드는 프로젝트 진행 순서

Spring Boot를 사용하면 이 작업이 훨씬 쉬워진다고 한다... 그래도 Spring 프로젝트를 만드는 순서를 기록할 겸, ~~2시간동안 오타 못 찾아서~~몇번이나 반복했던 과정을 기록해본다..

1. controller를 새로 만들고 `servlet-context.xml`에서 base-package를 변경한다
2. model 패키지 안에 dao, dto, service 패키지를 만든다
    1. dto에 데이터를 담을 Board 클래스를 만든다(생성자, getter, setter, toString)
    2. dao에 interface를 만든다(다양한 기능. selectAll, selectOne, insertBoard 등)
3. resources 패키지 안에 mapper 패키지를 만든다
    1. Mybatis 매핑XML에 기재된 SQL을 호출하기 위한 인터페이스인 board.xml을 만든다(`<select>`, `<insert>` 등)
4. views에 board, common 폴더를 만든다
    1. board 폴더에는 각 기능별 페이지를 맡은 jsp 파일을 만든다
    2. common은 각 페이지에서 공통으로 사용할 요소(bootstrap cdn 등)를 넣는다. common의 요소를 다른 파일에서 사용할 경우 상대 경로로 작성한다
    3. `pageEncoding="UTF-8”`을 확인한다
5. `pom.xml`에 라이브러리를 추가한다
    1. [mysql-connector-java](https://mvnrepository.com/artifact/mysql/mysql-connector-java): mysql 연결
    2. [Apache Commons DBCP](https://mvnrepository.com/artifact/org.apache.commons/commons-dbcp2): 커넥션을 관리하기 위해 사용
    3. [MyBatis](https://mvnrepository.com/artifact/org.mybatis/mybatis)
    4. [MyBatis Spring](https://mvnrepository.com/artifact/org.mybatis/mybatis-spring)
    5. dependency 중 `spring-webmvc`를 복사한 뒤 artifactId를 `spring-jdbc`로 바꿔준다
6. MyBatis 실행에 필요한 객체를 `root-context`에서 Spring Bean으로 등록한다
    1. `dataSource`로 driver, url, username, password를 등록한다
    2. 모든 MyBatis 애플리케이션이 사용하는 `sqlSessionFactory`인스턴스를 등록한다. (`mapperLocations`, `typeAliasesPackage` 등)
    3. mybatis가 scan할 수 있도록 dao를 base-package로 지정한다
7. service 패키지에 BoardService(interface)와 BoardServiceImpl(구현체)을 만든다.
    1. BoardService에는 사용자 친화적인 언어로 작성한다.
    2. BoardServiceImpl을 Bean으로 등록하기 위해 `@Service` annotation을 달아주고 `root-context`에서 `component-scan`의 base-package로 service 패키지를 설정한다.
    3. service에서는 dao를 호출해서 특정 명령을 내려야하기 때문에 dao를 가져온다. Spring container에서 만든 것을 쓸 것이기 때문에 직접 `@Autowired`를 달아주거나 setter를 만들고 달아준다(dao가 구현체가 있었고 기본생성자 없애고 다른 생성자를 만들었다면 setter는 제대로 작동하지 않을 수도 있다)
    4. 각각의 기능을 작성한다
8. BoardController를 만든다
    1. `@Controller`를 달아서 스캔 될 수 있게 한다
    2. boardService를 `@Autowired`한다
    3. Mapping을 통해 각 기능을 jsp로 연결한다(?)