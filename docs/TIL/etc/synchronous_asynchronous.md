---
layout: default
title: synchronous_asynchronous
parent: etc
grand_parent: TIL
---

# 동기와 비동기(JavaScript)
{: .no_toc }

## TABLE OF CONTENTS
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# 동기와 비동기
# Asynchronous JS

# 동기와 비동기
- 카페에서 주문할 때 앞사람의 메뉴가 준비될 때까지 사용자가 주문조차 못하면 화날 것이다(동기)
- 앞사람의 메뉴를 준비하는 동안 사용자의 주문을 받는 식으로 카페가 운영되어야 사용자의 만족도가 높을 것이다(비동기)

## 동기(synchronous)

- 모든 일을 순서대로 하나씩 처리하는 것
- 순서대로 처리한다 == 이전 작업이 끝나면 다음 작업을 시작한다
- 요청과 응답을 동기식으로 처리한다면?
    - 요청을 보내고 응답이 올때까지 기다렸다가 다음 로직을 처리

## 비동기(Asynchronous)

- 작업을 시작한 후 결과를 기다리지 않고 다음 작업을 처리하는 것(병렬적 수행)
- 시간이 필요한 작업들은 요청을 보낸 뒤 응답이 빨리 오는 작업부터 처리
- 거의 모든 웹이 비동기식임

## Single Thread 언어, JS

- JS는 한 번에 하나의 일만 수행할 수 있다
- 그래서 비동기 처리를 할 수 있도록 도와주는 환경이 필요하다
- 런타임(Runtime): 특정 언어가 동작할 수 있는 환경
- JS에서 비동기와 관련한 작업은 브라우저 또는 Node 환경에서 처리한다
- 브라우저 환경에서의 비동기 동작
    1. JavaScript Engine의 Call Stack
    2. Web API
    3. Task Queue
    4. Event Loop

## 비동기 처리 동작 방식
1. 모든 작업은 Call Stack(LIFO)으로 들어간 후 처리된다.
2. 오래 걸리는 작업이 Call Stack으로 들어오면 Web API로 보내서 처리하도록 한다.
3. Web API에서 처리가 끝난 작업들은 Task Queue(FIFO)에 순서대로 들어간다.
4. Event Loop가 Call Stack이 비어 있는 것을 체크하고, Task Queue에서 가장 오래된 장업을 Call Stack으로 보낸다.


# Axios 
공식 웹사이트 [링크](https://axios-http.com/kr/docs/intro)

- JS의 HTTP 웹 통신을 위한 라이브러리
- 확장 가능하나 인터페이스와 쉽게 사용할 수 있는 비동기 통신 기능을 제공
- get, post 등 여러 method 사용가능
    - then을 이용해서 성공하면 수행할 로직을 작성
    - catch를 이용해서 실패하면 수행할 로직을 작성


## Promise

- 콜백지옥을 해결하기 위해 등장한 비동기 처리를 위한 객체
- 앞의 작업이 끝나면 실행할 것이라는 객체. 순서를 보장한다
- 비동기 작업의 완료 또는 실패를 나타내는 객체
- Promise 기반의 클라이언트가 바로 이전에 사용한 Axios 라이브러리

### then(callback)

- 요청한 작업이 성공하면 callback 실행
- callback은 이전 작업의 성공 결과를 인자로 전달 받음

### catch(callback)

- then()이 하나라도 실패하면 callback 실행
- callback은 이전 작업의 실패 객체를 인자로 전달 받음

- then, catch 모두 항상 promise 객체를 반환
    
  - 계속해서 chaining 가능
    
  - then을 계속 이어 나가면서 작성 가능. 이전 then에서 꼭 return 해야 함
    
  - 코드가 피라미드 모양이 아니라 쭉 내려가는 모양이 됨
    

## Promise가 보장하는 것

1. Event Queue에 배치되는 엄격한 순서로 호출됨
2. 비동기 작업이 성공하거나 실패한 뒤에 `.then()` 메서드를 이용하여 추가한 경우에도 1번과 똑같이 동작
3. `.then()`을 여러 번 사용하여 여러 개의 callback 함수를 추가할 수 있음(Chaining)