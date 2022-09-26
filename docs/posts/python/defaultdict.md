---
layout: default
title: defaultdict
parent: Python
grand_parent: Posts
---

# defaultdict
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
[백준 10816번 숫자카드2](https://www.acmicpc.net/problem/10816)를 풀다가 다른 친구의 풀이를 봤는데 defaultdict를 사용하길래 더 자세히 알아보고 싶어졌다.

### 정의
기본값이 있는 딕셔너리를 만드는 dict클래스의 서브클래스이다.  
인자로 주어진 객체(default_factory)의 기본값을 딕셔너리값의 초깃값으로 지정할 수 있다.  
int, list, set 등으로 초기화 할 수 있다.


### [파이썬 공식 문서](https://docs.python.org/ko/3/library/collections.html?highlight=defaultdict#collections.defaultdict)
공식 문서를 읽어보니 꽤 편리한 딕셔너리이다.
* list를 default_factory로 사용하면, 키-값 쌍의 시퀀스를 리스트의 딕셔너리로 쉽게 그룹화 할 수 있다. 기존의 딕셔너리는 key가 없는 값이면 에러를 띄우는데 defaultdict는 새로운 key값을 생성하고 value를 list로 만든다
  ```python
  from collections import defaultdict

  s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
  d = defaultdict(list)
  for k, v in s:
      d[k].append(v)
  print(sorted(d.items()))
  # [('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]

  # 일반 딕셔너리 사용 시 에러 발생
  D = dict()
  for k, v in s:
      D[k].append(v)
  print(sorted(D.items()))
  # KeyError
  ```
* default_factory를 int로 설정하면 defaultdict를 세는(counting) 데 유용하게 사용할 수 있다.
  ```python
  from collections import defaultdict

  s = 'mississippi'
  d = defaultdict(int)
  for k in s:
      d[k] += 1

  print(sorted(d.items()))
  # [('i', 4), ('m', 1), ('p', 2), ('s', 4)]
  ```

### 비슷한 기능: dict.setdefault()
* 첫 번째 인자로 key, 두 번째 인자로 기본값을 넘기면 된다
```python
s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
d = {}
for i in s:
    d.setdefault(i, 0)
    d[i] += 1
```

### 참고 사이트
<https://docs.python.org/ko/3/library/collections.html?highlight=defaultdict#collections.defaultdict>
<https://dongdongfather.tistory.com/69>
<https://www.daleseo.com/python-collections-defaultdict/>
<https://codingdog.tistory.com/entry/%ED%8C%8C%EC%9D%B4%EC%8D%AC-collections-defaultdict%EC%97%90-%EB%8C%80%ED%95%B4-%EA%B0%84%EB%8B%A8%ED%9E%88-%EC%95%8C%EC%95%84%EB%B4%85%EC%8B%9C%EB%8B%A4>