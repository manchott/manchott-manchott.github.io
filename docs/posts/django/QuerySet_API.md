---
layout: default
title: QuerySet API Advanced
parent: Python
grand_parent: Posts
---

# defaultdict
{: .no_toc }

## QuerySet API Advanced
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## CRUD 기본
* 모든 User 레코드 조회: `User.objects.all()`
* User 레코드 생성
  1. 한번에 생성
  ```
  User.objects.create(
    first_name='길동',
    last_name='홍',
    age=100,
    country='제주도',
    phone='010-1234-5678',
    balance=10000,
  )
  ```
* 101번 user 레코드 조회: `user = User.objects.get(pk=101)`
* 101번 user 레코드의 last_name을 김으로 수정
  ```
  user.last_name = '김'
  user.save()
  ```
* 101번 user 레코드 삭제: `user.delete()`

* 전체 인원 수 조회: `User.objects.