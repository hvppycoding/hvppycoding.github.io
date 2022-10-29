---
title: "Fluent Python 02 - 시퀀스"
excerpt: "Fluent Python 02 - 시퀀스"
date: 2022-03-20 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Fluent Python
---

**Notice:** 본 글은 책 [『전문가를 위한 파이썬<sup>Fluent Python</sup>』](https://books.google.co.kr/books?id=NJpIDwAAQBAJ&hl=ko&source=gbs_navlinks_s)을 학습하며 정리한 글입니다. 전체 소스 코드는 [Fluent Python github 레포지토리](https://github.com/fluentpython/example-code)에서 확인할 수 있습니다.
{: .notice--info}

# Chapter 2. 시퀀스

파이썬은 시퀀스를 통일된 방식으로 다룰 수 있는 ABC 언어의 특징을 물려받았다. 문자열, 리스트, 바이트 시퀀스, 배열, XML 요소, 데이터베이스 결과에 모두 반복, 슬라이싱, 정렬, 연결 등 공통된 연산을 적용할 수 있다.  
파이썬에서 제공하는 다양한 시퀀스를 이해하면 코드를 새로 구현할 필요가 없다.

## 2.1 내장 시퀀스 개요

파이썬 표준 라이브러리는 C로 구현된 다음과 같은 시퀀스형을 제공한다.

- 컨테이너 시퀀스
  - 서로 다른 자료형의 항목을 담을 수 있는 `list`, `tuple`, `collections.deque`형
  - 객체에 대한 참조를 담고 있으며 객체는 어떠한 자료형도 될 수 있다.
- 균일 시퀀스
  - 단 하나의 자료형만 담을 수 있는 `str`, `bytes`, `bytearray`, `array.array`형
  - 객체에 대한 참조 대신 메모리 공간에 각 항목의 값을 직접 담는다. 균일 시퀀스가 메모리를 적게 사용하지만, 문자, 바이트, 숫자 등 기본적인 자료형만 저장할 수 있다.

또한, 가변성에 따라 분류할 수도 있는데, `tuple`, `str`, `bytes`형은 불변 시퀀스이고, 그 외 나머지 시퀀스는 가변 시퀀스이다.

아래 그림을 보면 가변 시퀀스가 불변 시퀀스와 어떻게 다른지 어느 메서드를 상속하는지 알 수 있다.  
![collections.abc UML]({{base.siteurl}}/assets/images/2022-03-20-sequence.drawio.svg)

## 2.2 지능형 리스트와 제너레이터 표현식

지능형 리스트<sup>list comprehension</sup>(리스트형)나 제너레이터 표현식<sup>generator expression</sup>(그 외 시퀀스형)을 사용하면 시퀀스를 간단히 생성할 수 있다.

### 2.2.1 지능형 리스트와 가독성

아래 [예제 1] 과 [예제 2] 중 어느 코드가 읽기 쉬운가?

```python
# 예제 1
>>> symbols = '$¢£¥€¤'
>>> codes = []
>>> for symbols in symbols:
...     codes.append(ord(symbol))
>>> codes
[36, 162, 163, 165, 8364, 164]
```

```python
# 예제 2
>>> symbols = '$¢£¥€¤'
>>> codes = [ord(symbol) for symbol in symbols]
>>> codes
[36, 162, 163, 165, 8364, 164]
```

지능형 리스트를 배운 후에는 [예제 2]가 더 읽기 좋다는 생각이 들 것이다. 의도를 명확히 보여주기 때문이다.  
`for` 루프는 아주 다양한 일에 사용할 수 있는 반면, 지능형 리스트는 오로지 새로운 리스트를 만드는 일만 한다.  

물론 지능형 리스트의 기능을 이용해서 일련의 코드를 반복 수행하는 코드도 본 적이 있다. <span class="custom-highlight" markdown="1">**생성된 리스트를 사용하지 않을 거라면 지능형 리스트 구문을 사용하지 말아야 한다. 그리고 코드를 짧게 만들어야 한다**</span>

### 2.2.2 지능형 리스트와 map()/filter() 비교

`map()`과 `filter()` 함수를 이용해서 수행할 수 있는 작업은 지능형 리스트를 이용해서 모두 구현할 수 있다.

```python
>>> symbols = '$¢£¥€¤'
>>> beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]
>>> beyond_ascii
[162, 163, 165, 8364, 164]
>>> beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols)))
>>> beyond_ascii
[162, 163, 165, 8364, 164]
```

### 2.2.3 데카르트 곱

지능형 리스트는 두 개 이상의 반복 가능한 자료형의 데카르트 곱을 나타내는 리스트를 만들 수 있다.  

![Cartesian product]({{base.siteurl}}/assets/images/2022-03-20-cartesian-product.drawio.svg)

```python
S = ['♠', '♥', '♦', '♣']
R = ['A', 'K', 'Q']

RxS = [(r, s) for r in R for s in S]

print(RxS)
# [('A', '♠'), ('A', '♥'), ('A', '♦'), ('A', '♣'),
#  ('K', '♠'), ('K', '♥'), ('K', '♦'), ('K', '♣'), 
#  ('Q', '♠'), ('Q', '♥'), ('Q', '♦'), ('Q', '♣')]
```
지능형 리스트는 오로지 리스트만 만들 수 있다. 다른 종류의 시퀀스를 채우려면 제너레이터 표현식을 사용해야 한다.

### 2.2.4 제너레이터 표현식

튜플, 배열 등의 시퀀스형을 초기화하려면 지능형 리스트를 사용할 수도 있지만, 다른 생성자에 전달할 리스트를 통째로 만들지 않고 반복자 프로토콜<sup>iterator protocol</sup>을 이용해서 항목을 하나씩 생성하는 제너레이터 표현식은 메모리를 더 적게 사용한다.  
제너레이터 표현식은 지능형 리스트와 구문은 유자하지만, 대괄호 대신 괄호를 사용한다.

```python
>>> symbols = '$¢£¥€¤'
>>> tuple(ord(symbol) for symbol in symbols)
(36, 162, 163, 165, 8364, 164)
```

다음 코드는 데카르트 곱 예제에 제너레이터 적용한 것이다. 제너레이터 표현식 사용 시 한 번에 모든 항목(여기서는 12개 항목)을 메모리에 생성하지 않고, 한 번에 한 항목을 생성하며 for 문에 전달하여 메모리 사용을 최소화할 수 있다.

```python
S = ['♠', '♥', '♦', '♣']
R = ['A', 'K', 'Q']

for card in ((r, s) for r in R for s in S):
    print(card)

# ('A', '♠')
# ('A', '♥')
# ...
# ('Q', '♦')
# ('Q', '♣')
```

## 2.3 튜플은 단순한 불변리스트가 아니다

### 2.3.1 레코드로서의 튜플
튜플을 레코드로 사용하는 경우가 종종 있다. 튜플을 단지 불변 리스트로 생각한다면 항목의 크기나 순서가 중요할 수도 있고 그렇지 않을 수도 있다. 그러나 튜플을 필드의 집합으로 사용하는 경우에는 항목 수가 고정되어 있고 항목의 순서가 중요하다.  

### 2.3.2 튜플 언패킹

튜플 언패킹

```python
>>> lax coordinates = (33.9425, -118.408056)
>>> latitude, longitude = lax_coordinates # 튜플 언패킹
>>> latitude
33.9425
>>> longitude
-118.408056
```

그리고 함수를 호출할 때 인수 앞에 *를 붙여 튜플을 언패킹할 수 있다.

```python
>>> divmode(20, 8)
(2, 4)
>>> t = (20, 8)
>>> divmod(*t)
(2, 4)
>>> quotient, remainder = divmod(*t)
```

튜플을 언패킹할 때 일부 항목에만 관심이 있는 경우에 *를 사용할 수도 있다.  

```python
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, *body, c, d = range(5)
>>> a, body, c, d
(0, [1, 2], 3, 4)
```

### 2.3.3 중첩 튜플 언패킹

```python
city, country, pop, (latitude, longitude) = ('Tokyo', 'JP', 36.933, (35.689722, 139.691667))

print(city, country, pop, latitude, longitude)
# Tokyo JP 36.933 35.689722 139.691667
```

### 2.3.4 명명된 튜플
튜플은 편리하지만 레코드로 사용하기에는 부족하다. 때로는 필드 이름을 붙일 필요가 있다. `collections.namedtuple()` 함수는 필드명과 클래스명을 추가한 튜플의 서브클래스를 생성하는 팩토리 함수로서, 디버깅을 용이하게 해준다. 

**Note** 필드명은 클래스에 저장되므로 `namedtuple()`로 생성한 객체는 튜플과 동일한 크기의 메모리만 사용한다. 속성을 객체마다 존재하는 `__dict__`에 저장하지 않으므로 일반적인 객체보다 메모리를 적게 사용한다.
{: .notice--info}

```python
from collections import namedtuple

City = namedtuple('City', 'name country population coordinates')
tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
print(tokyo)
# City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722, 139.691667))
print(tokyo.population)
# 36.933
print(tokyo[1])
# JP
print(City._fields)
# ('name', 'country', 'population', 'coordinates')

delhi_data = ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
delhi = City._make(delhi_data)
print(delhi._asdict())
# {'name': 'Delhi NCR', 'country': 'IN', 'population': 21.935, 'coordinates': (28.613889, 77.208889)}

for key, value in delhi._asdict().items():
    print(key + ':', value)
# name: Delhi NCR
# country: IN
# population: 21.935
# coordinates: (28.613889, 77.208889)
```

## 2.4 슬라이싱

### 2.4.1 슬라이스와 범위 지정시에 마지막 항목이 포함되지 않는 이유

슬라이스와 범위 지정시에 마지막 항목을 포함하지 않는 관례는 인덱스 번호가 0번부터 시작하는 파이썬, C 등의 언어에서 잘 작동한다. 이는 다음과 같은 장점이 있다.
- `my_list[:3]`처럼 중단점만 이용해서 슬라이스 범위를 지정할 때 길이를 계산하기 쉽다.
- 시작점과 중단점을 모두 지정할 때도 길이를 계산하기 쉽다. 중단점에서 시작점을 빼면 된다.
- `x` 인덱스를 기준으로 겹침 없이 시퀀스를 분할하기 쉽다. 단지 `my_list[:x]`와 `my_list[x:]`로 지정하면 된다.

### 2.4.2 슬라이스 객체
`s[a:b:c]`는 c 보폭<sup>stride</sup>만큼씩 항목을 건너뛰게 만든다. 보폭이 음수인 경우에는 거꾸로 거슬러 올라가 항목을 반환한다.  
`a:b:c` 표기법은 인덱스 연산을 수행하는 [] 안에서만 사용할 수 있으며, `slice(a, b, c)` 객체를 생성한다.  
아래 코드에서는 슬라이스를 하드코딩하는 대신 각 슬라이스에 이름을 붙였다. 

```python
invoice = """
0.....6.................................40..........52.55........
1909  Pimoroni PiBrella                       $17.50  3    $52.50
1489  6mm Tactile Switch x20                   $4.95  2     $9.90
1510  Panavise Jr. - PV-201                   $28.00  1    $28.00
"""

SKU = slice(0, 6)
DESCRIPTION = slice(6, 40)
UNIT_PRICE = slice(40, 52)
QUANTITY = slice(52, 55)
ITEM_TOTAL = slice(55, None)

line_items = invoice.split('\n')[2:]

for item in line_items:
    print(item[UNIT_PRICE], item[DESCRIPTION])

#      $17.50 Pimoroni PiBrella
#       $4.95 6mm Tactile Switch x20
#      $28.00 Panavise Jr. - PV-201
```

### 2.4.3 다차원 슬라이싱과 생략 기호

[] 연산자는 콤마로 구분해서 여러 개의 인덱스나 슬라이스를 가질 수 있다. 
예를 들어 NumPy 패키지에서 `a[i, j]` 구문으로 2차원 `numpy.ndarray` 배열이의 항목이나 
`a[m:n, k:l]` 구문으로 2차원 슬라이스를 가져올 때 사용한다.  
[] 연산자를 처리하는 `__getitem__()`과 `__setitem__()` 특수 메서드는 `a[i, j]`에 들어 있는 인덱스들을 튜플로 받는다. 즉, `a[i, j]`를 평가하기 위해 파이썬은 `a.__getitem__((i, j))`를 호출한다.  

세 개의 마침표(...)로 표현된 생략 기호는 파이썬에서 하나의 토큰으로 인식된다. 생략 기호 객체는 `f(a, ..., z)`처럼 함수의 인수나, `a[i:...]`처럼 슬라이스의 한 부분으로 전달할 수 있다. NumPy는 다차원 배열을 슬라이싱할 때 생략 기호를 사용한다. 예를 들어 x가 4차원 배열이라면 `x[i, ...]`는 `x[i, :, :, :,]`와 동일하다.

### 2.4.4 슬라이스에 할당하기

할당문의 왼쪽에 슬라이스 표기법을 사용하거나 `del`문의 대상 객체로 지정함으로써 가변 시퀀스를 연결하거나, 잘라 내거나, 값을 변경할 수 있다.

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 11, 5, 22, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
>>> l[2:5] = [100]
>>> l
[0, 1, 100, 22, 9]
```

## 2.5 시퀀스에 덧셈과 곱셈 연산자 사용하기

### 2.5.1 리스트의 리스트 만들기

중첩된 리스트를 가진 리스트를 초기화해야 하는 경우가 종종 있다. 이런 리스트를 초기화할 때는 지능형 리스트를 사용하는 것이 가장 좋다.
**Caution** `a`가 가변 항목을 담고 있을 때 `a * n`과 같은 표현식을 사용하려면 주의해야 한다.
{: .notice--warning}

```python
board = [['_'] * 3 for i in range(3)]
print(board)
# [['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
board[1][2] = 'X'
print(board)
# [['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]

weired_board = [['_'] * 3] * 3
print(weired_board)
# [['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
weired_board[1][2] = 'O'
print(weired_board)
# [['_', '_', 'O'], ['_', '_', 'O'], ['_', '_', 'O']]
```

## 2.6 시퀀스의 복합 할당

`+=` 연산자가 작동하도록 만드는 특수 메서드는 `__iadd__()`다. 그런데 해당 메서드가 구현되어 있지 않으면, 파이썬은 대신 `__add__()` 메서드를 호출한다. `a`가 `__iadd__()`를 구현하지 않은 경우 `a += b` 표현식은 `a = a + b`가 되어 먼저 `a + b`를 평가하고, 객체를 새로 생성한 후 `a`에 할당한다.  
일반적으로 가변 시퀀스에 대해서는 `__iadd__()` 메서드를 구현해서 `+=` 연산자가 기존 객체의 내용을 변경하게 만드는 것이 좋다.

## 2.7 `list.sort()`와 `sorted()` 내장 함수

`list.sort()` 메서드는 사본을 만들지 않고 리스트 내부를 변경해서 정렬한다. `sort()` 메서드는 타깃 객체를 변경하고 새로운 리스트를 생성하지 않았음을 알려주기 위해 `None`을 반환한다.  
이와 반대로 `sorted()` 내장 함수는 새로운 리스트를 생성해서 반환한다.  
`list.sort()` 메서드와 `sorted()` 함수 모두 두 개의 옵션 파라미터가 존재한다.
- `reverse`: 이 키워드가 참이면 비교 연산을 반대로 해서 내림차순으로 반환한다. 기본값은 `False`이다.
- `key`: 정렬에 사용할 키를 생성하기 위해 각 항목에 적용할 함수를 파라미터로 받는다. 예를 들어 문자열의 리스트를 정렬할 때 `key=str.lower`로 지정하면 대소문자를 구분하지 않고 정렬하며, `key=len`으로 지정하면 문자열의 길이에 따라 문자열을 정렬한다.  

일단 시퀀스를 정렬한 후에는 아주 효율적으로 검색할 수 있다. 파이썬 표준 라이브러리의 `bisect` 모듈에서 표준 이진 검색 알고리즘을 제공하고 있다.

## 2.8 정렬된 시퀀스를 `bisect`로 관리하기

`bisect` 모듈은 시퀀스 이진 검색을 위한 `bisect()`함수와 시퀀스에 새로운 항목 삽입을 위한 `insort()` 함수를 제공한다.

### 2.8.1 `bisect()`로 검색하기

`bisect(haystack, needle)`은 정렬된 시퀀스인 `haystack` 안에서 오름차순 정렬 상태를 유지한 채로 `needle`을 추가할 수 있는 위치를 찾아낸다. 결과 index를 이용하여 `haystack.insert(index, needle)` 함수를 호출하면 `needle`을 추가할 수 있지만, `insort()` 함수는 이 두 과정을 더 빨리 처리한다.

```python
import bisect
import sys

HAYSTACK = [1, 4, 5, 6, 8, 12, 15, 20, 21, 23, 23, 26, 29, 30]
NEEDLES = [0, 1, 2, 5, 8, 10, 22, 23, 29, 30, 31]

ROW_FMT = '{0:2d} @ {1:2d}    {2}{0:<2d}'

def demo(bisect_fn):
    for needle in reversed(NEEDLES):
        position = bisect_fn(HAYSTACK, needle)  # <1>
        offset = position * '  |'  # <2>
        print(ROW_FMT.format(needle, position, offset))  # <3>

if __name__ == '__main__':

    if sys.argv[-1] == 'left':    # <4>
        bisect_fn = bisect.bisect_left
    else:
        bisect_fn = bisect.bisect

    print('DEMO:', bisect_fn.__name__)  # <5>
    print('haystack ->', ' '.join('%2d' % n for n in HAYSTACK))
    demo(bisect_fn)
```

<div class="no-highlight" markdown="1">

```
>>> python bisect_demo.py
DEMO: bisect_right
haystack ->  1  4  5  6  8 12 15 20 21 23 23 26 29 30
31 @ 14      |  |  |  |  |  |  |  |  |  |  |  |  |  |31
30 @ 14      |  |  |  |  |  |  |  |  |  |  |  |  |  |30
29 @ 13      |  |  |  |  |  |  |  |  |  |  |  |  |29
23 @ 11      |  |  |  |  |  |  |  |  |  |  |23
22 @  9      |  |  |  |  |  |  |  |  |22
10 @  5      |  |  |  |  |10
 8 @  5      |  |  |  |  |8
 5 @  3      |  |  |5
 2 @  1      |2
 1 @  1      |1
 0 @  0    0

>>> python bisect_demo.py left
DEMO: bisect_left
haystack ->  1  4  5  6  8 12 15 20 21 23 23 26 29 30
31 @ 14      |  |  |  |  |  |  |  |  |  |  |  |  |  |31
30 @ 13      |  |  |  |  |  |  |  |  |  |  |  |  |30
29 @ 12      |  |  |  |  |  |  |  |  |  |  |  |29
23 @  9      |  |  |  |  |  |  |  |  |23
22 @  9      |  |  |  |  |  |  |  |  |22
10 @  5      |  |  |  |  |10
 8 @  4      |  |  |  |8
 5 @  2      |  |5
 2 @  1      |2
 1 @  0    1
 0 @  0    0
```

</div>

`bisect`는 두 가지 방식으로 조절할 수 있다.
1. 옵션 파라미터인 `lo`와 `hi`를 사용하면 검색할 시퀀스 영역을 좁힐 수 있다. `lo`의 기본값은 0, `hi`의 기본값은 시퀀스의 `len()`이다.
2. `bisect` 함수는 실제로는 `bisect_right()` 함수의 별칭이며, `bisect_left()` 함수도 존재한다. 이 두 함수는 리스트 안에 `needle`과 동일한 값이 존재할 때만 동작에 차이가 있으며, `bisect_right()`는 기존 항목들 바로 뒤의 index를 반환하고, `bisect_left()`는 기존 항목 첫 위치 index를 반환한다.

### 2.8.2 `bisect.insort()`로 삽입하기

정렬은 값비싼 연산이므로 시퀀스를 한 번 정렬한 후에는 정렬 상태를 유지하는 것이 좋다. `insort(seq, item)` 함수는 `seq`를 오름차순으로 유지한 채로 `item`을 `seq`에 삽입한다.

```python
import bisect
import random

SIZE = 7

random.seed(1729)

my_list = []
for i in range(SIZE):
    new_item = random.randrange(SIZE*2)
    bisect.insort(my_list, new_item)
    print('%2d ->' % new_item, my_list)
```

<div class="no-highlight" markdown="1">

```
10 -> [10]
 0 -> [0, 10]
 6 -> [0, 6, 10]
 8 -> [0, 6, 8, 10]
 7 -> [0, 6, 7, 8, 10]
 2 -> [0, 2, 6, 7, 8, 10]
10 -> [0, 2, 6, 7, 8, 10, 10]
```

</div>

## 2.9 리스트가 답이 아닐 때

리스트형이 사용하기 편하기 때문에 파이썬 프로그래머들은 리스트형을 남용하곤 한다. 물론 나도 그랬다. 만약 숫자들로 구성된 리스트를 다루고 있다면 배열을 사용하는 것이 좋다.
예를 들어 숫자 천만 개를 저장해야 할 때는 배열이 훨씬 더 효율적이다.
한편 리스트 양쪽 끝에 항목을 계속 추가하거나 삭제할 때는 `deque`가 더 빠르다.

### 2.9.1 배열

리스트 안에 숫자만 들어 있다면 배열(`array.array`)이 효율적이다. 

```python
from array import array
from random import random

floats = array('d', (random() for i in range(10**7)))
```

### 2.9.2 메모리 뷰
메모리 뷰(`memoryview`) 내장 클래스는 공유 메모리 시퀀스형으로 bytes를 복사하지 않고 배열을 다룰 수 있게 해준다.

### 2.9.3 NumPy와 SciPy
NumPy와 SciPy가 제공하는 고급 배열 및 행렬 연산 덕분에 파이썬이 과학 계산 애플리케이션에서 널리 사용되게 되었다. 아래 예제는 NumPy에서 2차원 배열에 간단한 연산을 수행하는 방법을 보여준다.

```python
import numpy

a = numpy.arange(12)
print(a)
# [ 0  1  2  3  4  5  6  7  8  9 10 11]
print(type(a))
# <class 'numpy.ndarray'>
print(a.shape)
# (12,)
a.shape = 3, 4
print(a)
# [[ 0  1  2  3]
# [ 4  5  6  7]
# [ 8  9 10 11]]
print(a[2])
# [ 8  9 10 11]
print(a[2, 1])
# 9
print(a[:, 1])
# [1 5 9]
print(a.transpose())
# [[ 0  4  8]
#  [ 1  5  9]
#  [ 2  6 10]
#  [ 3  7 11]]
```

### 2.9.4 덱 및 기타 큐

`append()`와 `pop()` 메서드를 사용해서 리스트를 스택이나 큐로 사용할 수 있다. 그러나 리스트 왼쪽(인덱스 0)에 삽입하거나 삭제하는 연산은 전체 리스트를 이동시켜야 해서 처리 부담이 크다.  

덱(`collections.deque`) 클래스는 큐의 양쪽 어디에서든 빠르게 삽입 및 삭제할 수 있도록 설계된 thread-safe 양방향 큐다. 

```python
from collections import deque

dq = deque(range(10), maxlen=10)
print(dq)
# deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)

dq.rotate(3)
print(dq)
# deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)

dq.rotate(-4)
print(dq)
# deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)

dq.appendleft(-1)
print(dq)
# deque([-1, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)

dq.extend([11, 22, 33])
print(dq)
# deque([3, 4, 5, 6, 7, 8, 9, 11, 22, 33], maxlen=10)

dq.extendleft([10, 20, 30, 40])
print(dq)
# deque([40, 30, 20, 10, 3, 4, 5, 6, 7, 8], maxlen=10)
```

## 2.10 읽을거리

`sorted()`와 `list.sort()`에 사용된 정렬 알고리즘은 팀정렬<sup>Timsort</sup>이다. 팀정렬은 데이터의 정렬된 정도에 따라 삽입 정렬과 병합 정렬 사이를 전환하는 적응형 알고리즘이다. 실세계 데이터에는 정렬된 데이터 덩어리들이 들어 있는 경우가 많기 때문에 상당히 효율적이다.