---
title: "Fluent Python 03 - 딕셔너리와 집합"
excerpt: "Fluent Python 03 - 딕셔너리와 집합"
date: 2022-03-20 17:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Fluent Python
mathjax: "true"
---

**Notice:** 본 글은 책 [『전문가를 위한 파이썬<sup>Fluent Python</sup>』](https://books.google.co.kr/books?id=NJpIDwAAQBAJ&hl=ko&source=gbs_navlinks_s)을 학습하며 정리한 글입니다. 전체 소스 코드는 [Fluent Python github 레포지토리](https://github.com/fluentpython/example-code)에서 확인할 수 있습니다.
{: .notice--info}

# Chapter 3. 딕셔너리와 집합

`dict`형은 파이썬 구현의 핵심 부분이다. 모듈 네임스페이스, 클래스 및 인스턴스 속성, 함수의 키워드 인수 등 핵심 부분에 딕셔너리가 사용되고 있고, 내장 함수들은 `__builtins__.__dict__`에 들어 있다.  
중요한 역할을 맡고 있으므로 파이썬 `dict` 클래스는 상당히 최적화되어있다. 파이썬의 고성능 딕셔너리 뒤에는 <span class="custom-highlight" markdown="1">**해시 테이블**</span>이라는 엔진이 있다.

## 3.1 일반적인 매핑형

`collections.abc` 모듈은 `dict` 및 이와 유사한 자료형의 인터페이스를 정의하기 위해 `Mapping` 및 `MutableMapping` 추상 베이스 클래스(ABC)를 제공한다.

![Mapping UML]({{base.siteurl}}/assets/images/2022-03-20-mapping.drawio.svg)

함수 인수가 `dict`형인지 검사하는 것 보다 `isinstance(my_dict, collections.abc.Mapping)` 함수를 사용하는 것이 좋다. 다른 매핑형이 사용될 수도 있기 때문이다.  
표준 라이브러리에서 제공하는 매핑형은 모두 `dict`를 사용해서 구현하므로, 키가 **해시 가능**해야 한다.  

## 3.2 지능형 딕셔너리

**지능형 딕셔너리**<sup>dictcomp</sup>는 모든 반복형 객체에서 키-값 쌍을 생성함으로써 딕셔너리 객체를 만들 수 있다.

```python
DIAL_CODES = [
        (86, 'China'),
        (91, 'India'),
        (1, 'United States'),
        (62, 'Indonesia'),
        (55, 'Brazil'),
        (92, 'Pakistan'),
        (880, 'Bangladesh'),
        (234, 'Nigeria'),
        (7, 'Russia'),
        (81, 'Japan'),
    ]

country_code = {country: code for code, country in DIAL_CODES}
print(country_code)
# {'China': 86, 'India': 91, 'United States': 1, 'Indonesia': 62, 'Brazil': 55, 'Pakistan': 92, 'Bangladesh': 880, 'Nigeria': 234, 'Russia': 7, 'Japan': 81}
```

## 3.3 공통적인 매핑 메서드

기본 `dict` 클래스의 API는 다음과 같다.

| 메서드 | 설명 |
|-------|------|
|`d.clear()`| 모든 항목 제거 |
|`d.__contains__(k)` | `k in d` |
|`d.copy()` | 얕게 복사 |
|`d.__delitem__(k)` | `del d[k]` - 키가 `k`인 항목을 제거 |
|`d.fromkeys(it, [initial]` | 선택적인 초기값(default는 None)을 받아, 반복 가능한 키들을 이용해서 새로 딕셔너리 생성 |
|`d.get(k, [default])` | `k` 키를 가진 항목을 반환한다. 해당 항목이 없으면 `default` 반환한다. `default`의 기본값은 `None`이다. |
|`d.__getitem__(k)` | `d[k]` - `k` 키를 가진 항목을 반환한다. |
|`d.items()` | (키, 값) 쌍으로 구성된 항목들의 뷰를 가져온다. |
|`d.__iter__()` | 키에 대한 반복자를 가져온다. |
|`d.keys()` | 키에 대한 뷰를 가져온다. |
|`d.__len__()` | `len(d)` - 항목 수를 반환한다. |
|`d.__missing__()` | `__getitem__()`이 `k` 키를 찾을 수 없을 때 호출된다. |
|`d.pop(k, [default])` | `k` 키를 제거하고 반환한다. 항목이 없으면 `default`를 반환한다. 기본값은 `None`이다. |
|`d.popitem()` | 처음이나 마지막 (키, 값) 항목을 제거하고 반환한다. |
|`d.setdefault(k, [default])` | `k in d`가 참이면 `d[k]`를 반환하고, 아니면 `d[k] = default`로 설정하고 이 값을 반환한다. |
|`d.__setitem__(k, v)` | `d[k] = v` - `k` 키를 가진 항목의 값을 `v`로 설정한다. |
|`d.update(m, [**kargs])` | (키, 값) 쌍의 매핑이나 반복형 객체에서 가져온 항목들로 `d`를 갱신한다. |
|`d.values()` | 값들에 대한 뷰를 가져온다. |

### 3.3.1 존재하지 않는 키를 `setdefault()`로 처리하기

조기 실패<sup>fail-fast</sup> 철학에 따라, 존재하지 않는 키 `k`로 `d[k]`를 접근하면 `dict`는 오류를 발생시킨다. `KeyError`를 처리하는 것보다 기본값을 사용하는 것이 더 편리한 경우에는 `d[k]` 대신 `d.get(k, default)`를 사용할 수 있다. 그런데 값을 갱신해야할 때는 `get()` 후에 `d[k] = ?`하여 값을 할당하는 것보다 `setdefault()` 함수를 사용하면 더 좋다.

```python
text = """Kurt,Cobain
Donald,Duck
Helen,Keller
Warren,Cobain
Duck,Duck
Johnny,Duck"""

d = {}

for line in text.split('\n'):
    first, last = line.split(',')
    d.setdefault(last, []).append(first)

for k in d:
    print(k + ':', d[k])
```

<div class="no-highlight" markdown="1">

```text
Cobain: ['Kurt', 'Warren']
Duck: ['Donald', 'Duck', 'Johnny']
Keller: ['Helen']
```

</div>

`d.setdefault(last, []).append(first)` 코드는 아래 코드와 동일하다.

```python
if last not in d:
  d[last] = []
d[last].append(first)
```

## 3.4 융통성 있게 키를 조회하는 매핑

### 3.4.1 `defaultdict`: 존재하지 않는 키에 대한 또 다른 처리

`collections.defaultdict`를 사용해서 앞의 문제를 해결할 수 있다.

```python
import collections

text = """Kurt,Cobain
Donald,Duck
Helen,Keller
Warren,Cobain
Duck,Duck
Johnny,Duck"""

d = collections.defaultdict(list)

for line in text.split('\n'):
    first, last = line.split(',')
    d[last].append(first)

for k in d:
    print(k + ':', d[k])
```

### 3.4.2 `__missing__()` 메서드

`dict` 클래스를 상속하고 `__missing__` 메서드를 정의하면, `dict.__getitem__()` 메서드가 키를 발견할 수 없을 때 `KeyError`를 발생시키지 않고 `__missing__()` 메서드를 호출한다.  

비문자열로 키를 검색할 때 키를 찾지 못하면 문자열화하여 탐색하는 딕셔너리를 구현해보자.

```python
class StrKeyDict0(dict):  # <1>

    def __missing__(self, key):
        if isinstance(key, str):  # <2>
            raise KeyError(key)
        return self[str(key)]  # <3>

    def get(self, key, default=None):
        try:
            return self[key]  # <4>
        except KeyError:
            return default  # <5>

    def __contains__(self, key):
        return key in self.keys() or str(key) in self.keys()  # <6>

d = StrKeyDict0()
d['4'] = 'four'
d['2'] = 'two'

print(d[4])
# four
print(d.get(4)) # get 함수를 구현하지 않으면 여기서 None이 리턴되므로 주의.
# four
print(d[1])
# Traceback (most recent call last):
#   File "strkeydict0.py", line 63, in <module>
#     print(d[1])
#   File "strkeydict0.py", line 44, in __missing__
#     return self[str(key)]  # <3>
#   File "strkeydict0.py", line 43, in __missing__
#     raise KeyError(key)
# KeyError: '1'
```

## 3.5 그 외 매핑형

- `collections.OrderedDict`: 키를 삽입한 순서대로 유지함으로써 항목을 반복하는 순서를 예측할 수 있다.
- `collections.ChainMap`: 매핑들의 목록을 담고 있으며 한꺼번에 모두 검색할 수 있다. 각 매핑을 차례대로 검색하고, 그 중 하나에서라도 키가 검색되면 성공한다. 다음 코드 예제는 파이썬에서 변수를 조회하는 기본 규칙을 표현하고 있다.

  ```python
  import builtins
  pylookup = ChainMap(locals(), globals(), vars(builtins))
  ```

- `collections.Counter`: 모든 키에 대한 횟수 카운터를 갖고 있는 매핑이다. 다음 코드는 `Counter`를 사용해서 각 글자의 횟수를 계산하는 예이다.

  ```python
  collections.Counter('abracadabra')
  # Counter({'a': 5, 'b': 2, 'r': 2, 'c':1, 'd': 1})
  ```

## 3.6 `UserDict` 상속하기

`dict`보다는 `UserDict`를 상속해서 매핑형을 만드는 것이 쉽다. 특수 메서드들의 원치 않는 재귀적 호출이 일어날 수 있기 때문에 까다롭기 때문이다.

```python
class StrKeyDict(collections.UserDict):  # <1>
  def __missing__(self, key):  # <2>
      if isinstance(key, str):
          raise KeyError(key)
      return self[str(key)]

  def __contains__(self, key):
      return str(key) in self.data  # <3>

  def __setitem__(self, key, item):
      self.data[str(key)] = item   # <4>

# END STRKEYDICT

d = StrKeyDict()
d['4'] = 'four'
d['2'] = 'two'

print(d[4])
# four
print(d.get(4))
# four
print(d[1])
# Traceback (most recent call last):
#   File "strkeydict0.py", line 63, in <module>
#     print(d[1])
#   File "strkeydict0.py", line 44, in __missing__
#     return self[str(key)]  # <3>
#   File "strkeydict0.py", line 43, in __missing__
#     raise KeyError(key)
# KeyError: '1'
```

## 3.7 불변 매핑

표준 라이브러리에서 제공하는 매핑형은 모두 가변형이지만, 사용자가 실수로 매핑을 변경하지 못하도록 보장하고 싶은 경우가 있을 수 있다.  
`MappingProxyType`이라는 래퍼 클래스를 통해 불변 매핑 객체를 만들 수 있다. 원래의 매핑을 변경하면 `MappingProxy`에 반영되지만, 이를 직접 변경할 수는 없다.

```python
from types import MappingProxyType

d = {1: 'A'}
d_proxy = MappingProxyType(d)
print(d_proxy[1])
# A

# d_proxy[2] = 'x'
# TypeError: 'mappingproxy' object does not support item assignment
d[2] = 'B'

print(d_proxy)
# {1: 'A', 2: 'B'}
```

## 3.8 집합 이론

집합 `set` 객체는 고유한 객체의 모음으로서, 기본적으로 중복 항목을 제거한다.

```python
l = ['spam', 'spam', 'eggs', 'spam']
print(set(l))
# {'eggs', 'spam'}
```

### 3.8.1 집합 리터럴

`{1}`, `{1, 2}`와 같은 수학적 표기법과 동일하게 집합 리터럴을 생성할 수 있다. 하지만 공집합의 경우 `{}`는 딕셔너리를 생성하므로 `set()`로 표기해야 한다.

### 3.8.2 지능형 집합

기존의 지능형 리스트나 지능형 딕셔너리와 유사하다.

```python
text = """Kurt,Cobain
Donald,Duck
Helen,Keller
Warren,Cobain
Duck,Duck
Johnny,Duck"""

s = {line.split(',')[1] for line in text.split('\n') if line != 'Helen,Keller'}
print(s)
# {'Duck', 'Cobain'}
```

### 3.8.3 집합 연산

| 연산 | 설명 |
|-----|------|
| `a & b` | 교집합 |
| `a | b` | 합집합 |
| `a - b` | 차집합 |
| `a ^ b` | 대칭차(`a & b`의 여집합)|

이 외에도 집합 연산을 제공한다. 예를 들어 $S \subset Z$ 연산의 경우 파이썬 연산자 `S < Z`로 구할 수 있다. 이는 특별 메서드 `__lt__()`에 구현되어 있다.
