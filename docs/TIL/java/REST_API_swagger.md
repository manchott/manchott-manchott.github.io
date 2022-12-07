---
layout: default
title: REST, API, Swagger
parent: JAVA
grand_parent: TIL
last_modified_date: 2022-12-07
---

# REST, API, Swagger

{: .no_toc }

## TABLE OF CONTENTS

{: .no_toc .text-delta }

1. TOC
   {:toc}

---

## REST(Representational State Transfer)

- URI는 하나의 고유한 리소스를 대표하도록 설계된다는 개념 + 전송 방식
- 웹의 장점을 최대한 활용할 수 있는 설계구조(architecture)

### REST 구성

- 자원 : URI
- 행위 : HTTP Method
- 표현 : JSON, XML 등의 언어로 표현 가능.
- 잘 표현된 HTTP URI로 리소스를 정의하고 HTTP method로 리소스에 대한 행위를 정의한다. 리소스는 JSON, XML과 같은 여러가지 언어로 표현할 수 있다.

### REST 아키텍처 스타일의 원칙([출처](https://aws.amazon.com/ko/what-is/restful-api/))
* 균일한 인터페이스: 서버가 표준 형식으로 정보를 전송한다.
  * 요청은 리소스를 식별해야 합니다. 이를 위해 균일한 리소스 식별자를 사용합니다.
  * 클라이언트는 원하는 경우 리소스를 수정하거나 삭제하기에 충분한 정보를 리소스 표현에서 가지고 있습니다. 서버는 리소스를 자세히 설명하는 메타데이터를 전송하여 이 조건을 충족합니다.
  * 클라이언트는 표현을 추가로 처리하는 방법에 대한 정보를 수신합니다. 이를 위해 서버는 클라이언트가 리소스를 적절하게 사용할 수 있는 방법에 대한 메타데이터가 포함된 명확한 메시지를 전송합니다.
  * 클라이언트는 작업을 완료하는 데 필요한 다른 모든 관련 리소스에 대한 정보를 수신합니다. 이를 위해 서버는 클라이언트가 더 많은 리소스를 동적으로 검색할 수 있도록 표현에 하이퍼링크를 넣어 전송합니다.
* 무상태: 서버가 이전의 모든 요청과 독립적으로 모든 클라이언트 요청을 완료하는 통신 방법
* 계층화 시스템: 클라이언트는 클라이언트와 서버 사이의 다른 승인된 중개자에게 연결할 수 있으며 여전히 서버로부터도 응답을 받는다. 서버는 요청을 다른 서버로 전달할 수도 있다. 이러한 계층은 클라이언트에 보이지 않는 상태로 유지된다.
* 캐시 가능성: 캐시 가능 또는 캐시 불가능으로 정의되는 API 응답을 사용하여 캐싱을 제어함
* 온디맨드 코드: 서버는 소프트웨어 프로그래밍 코드를 클라이언트에 전송하여 클라이언트 기능을 일시적으로 확장하거나 사용자 지정할 수 있다.

## API(Application Prograamming Interface)

- 두 소프트웨어 요소들이 서로 통신할 수 있게 하는 방식
- Application: 고유한 기능을 가진 모든 소프트웨어
- Interface: 두 애플리케이션 간의 요청과 응답에 의한 통신하는 방법

### API 유형

- 프라이빗 API
  - 기업 내부에 있으며 비즈니스 내에서 시스템과 데이터를 연결하는 데만 사용
- 퍼블릭 API
  - 일반에 공개되며 누구나 사용 가능(사용에 대한 권한 설정과 비용이 있을 수도 있다)
  - 공공데이터 포털, 기상청, Naver, Kakao, Youtube 등
  - 대부분이 REST 방식으로 작성

## REST API(REST + API)

- 기존의 전송방식과 달리 서버는 요청으로 받은 리소스에 대해 순수한 데이터를 전송
- 기존의 GET/POST 외에 PUT, DELETE 방식을 사용하여 리소스에 대한 CRUD 처리를 할 수 있다
- HTTP URI를 통해 제어할 자원(Resource)을 명시하고, HTTP METHOD(GET, POST, PUT, DELETE)를 통해 해당 자원(Resource)를 제어하는 명령을 내리는 방식의 Architecture
- 암묵적으로 정해진 표준을 따륾
  - `-`은 사용 가능하지만 `_`는 사용하지 않는다
  - 대소문자 구분을 하기 때문에 특별한 경우를 제외하고 대문자 사용은 하지 않는다
  - URI 마지막에 `/`를 사용하지 않는다
  - `/`로 계층 관계를 나타낸다
  - 확장자가 포함된 파일 이름을 직접 포함시키지 않는다
  - URI는 명사를 사용한다

### 기존 Service와 REST Service
- 기존
  - 요청 처리 후 가공된 data를 이용해 특정 플랫폼에 적합한 형태의 View로 만들어 반환
  - 핸드폰, PC, TV 등 플랫폼에 따른 View를 만들어줘야함
- REST
  - data 처리만 하거나, 처리 후 반환될 data가 있다면 JSON이나 XML 형식으로 전달
  - View에 대해 처리하지 않음

### REST 관련 Annotation

- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`: 요청방식
- `@RestController` : Controller가 REST 방식을 처리하기 위한 것임을 명시. Controller + `@ResponseBody`
- `@ResponseBody` : JSP 같은 뷰로 전달되는 것이 아니라 데이터 자체 전달.
- `@PathVariable` : URL경로에 있는 값을 파라미터로 추출.
- `@CrossOrigin` : Ajax의 크로스 도메인 문제 해결.
- `@RequestBody` : JSON 데이터를 원하는 타입으로 바인딩.
