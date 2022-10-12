---
layout: default
title: QuerySet API Advanced
parent: Django
grand_parent: Posts
---

# defaultdict
{: .no_toc }

## QuerySet API Advanced
{: .no_toc .text-delta }

1. TOC
{:toc}

---

SQL로 넘어가는Query가 궁금하다면 `print(User.objects.all().query)`를 통해 확인할 수 있다

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

* 전체 인원 수 조회:
  1. `len(User.objects.all())`
  2. `User.objects.count()`
  
## Sorting data
* 나이가 어린 순으로 이름과 나이 조회하기
  `User.objects.order_by('age').values('first_name', 'age')`
* 나이가 많은 순으로 이름과 나이 조회하기
  `User.objects.order_by('-age').values('first_name', 'age')`
* 랜덤으로 이름과 나이 조회하기
  `User.objects.order_by('?').values('first_name', 'age')`
* 나이가 어린 순으로, 만약 같은 나이라면 계좌 잔고가 많은 순으로 정렬해서 이름과 나이, 계좌 잔고 조회하기
  `User.objects.order_by('age', '-balance').values('first_name', 'age','balance')`

## Filtering data
* 중복 없이 모든 지역 오름차순으로 조회하기
  `User.objects.distinct().values('country').order_by('country')`
* 나이가 30인 사람들의 이름 조회
  `User.objects.filter(age=30).values('first_name')`
* 나이가 30살 이상인 사람들의 이름과 나이 조회
  `User.objects.filter(age__gte=30).values('first_name')`
* 나이가 30살 이상이고 계좌 잔고가 50만원 초과인 사람들의 이름과 나이, 계좌 잔고 조회
  `User.objects.filter(age__gte=30, balance__gt=500000).values('first_name', 'age','balance')`
* 이름에 '호'가 포함되는 사람들의 이름과 성 조회하기
  `User.objects.filter(first_name__contains='호').values('first_name', 'last_name')`
* 핸드폰 번호가 011로 시작하는 사람들의 이름과 핸드폰 번호 조회
  `User.objects.filter(phone__startswith='011-').values('first_name', 'phone')`
* 이름이 '준'으로 끝나는 사람들의 이름과 성 조회하기
  `User.objects.filter(first_name__endswith='준').values('first_name', 'last_name')`
* 경기도 혹은 강원도에 사는 사람들의 이름과 지역 조회하기
  `User.objects.filter(country__in=['강원도', '경기도']).values('first_name', 'country')`
* 경기도 혹은 강원도에 살지 않는 사람들의 이름과 지역 조회하기
  `User.objects.exclude(country__in=['강원도', '경기도']).values('first_name', 'country')`
* 나이가 가장 어린 10명의 이름과 나이 조회하기
  `User.objects.order_by('age').values('first_name', 'age')[:10]`
* 나이가 30이거나 성이 김씨인 사람들 조회
  ```python
  from django.db.models import Q # OR을 위한 import
  User.objects.filter(Q(age=30) | Q(last_name='김')).values('last_name', 'age')
  ```
## Aggregation(Grouping data)
### aggregate(): 전체 queryset에 대한 값을 계산
* 나이가 30살 이상인 사람들의 평균 나이 조회하기
  ```python
  from django.db.models import Avg
  User.objects.filter(age__gte=30).aggregate(Avg('age'))
  # 딕셔너리 key 이름 수정도 가능
  User.objects.filter(age__gte=30).aggregate(avg30 = Avg('age'))
  ```
* 가장 높은 계좌 잔액 조회하기
  ```python
  from django.db.models import Max
  User.objects.aggregate(Max('balance'))
* 모든 계좌 잔액 총액 조회하기
  ```python
  from django.db.models import Sum
  User.objects.aggregate(Sum('balance'))
  ```
### annotate(): 쿼리의 각 항목에 대한 요약 값을 계산(SQL의 GROUP BY에 해당)
* 각 지역별로 몇 명씩 살고 있는지 조회하기
  ```python
  from django.db.models import Count
  User.objects.values('country').annotate(num_of_country = Count('country'))
  ```
* 각 지역별로 몇 명씩 살고 있는지 + 지역별 계좌 잔액 평균 조회하기
  `User.objects.values('country').annotate(num_of_country = Count('country'), avg_balance=Avg('balance'))`