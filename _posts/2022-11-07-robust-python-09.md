---
title: "Robust Python 09 - 견고한 파이썬"
excerpt: "Robust Python 09 - 견고한 파이썬"
date: 2022-11-07 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Robust Python
---

본 포스트는 『단단한 파이썬』 책을 보며 정리한 글입니다. 자세한 내용은 책을 참고하세요.  
예제 코드: [https://github.com/pviafore/RobustPython](https://github.com/pviafore/RobustPython)
{: .notice--info}


# 9장. 사용자 정의 타입: 데이터 클래스

[p.209] 데이터 클래스는 연관 데이터의 그룹화를 위한 사용자 정의 타입이다. 딕셔너리, 튜플을 사용할수도 있지만 안에 무엇이 담겨있는지 런타임에 확인하기 어렵다. 

### 데이터 클래스의 동작

`Fraction`은 복합 타입의 좋은 예다. 여기에는 `numerator`와 `denominator`라는 두 개의 스칼라 값이 존재한다.
이 둘은 서로 **독립적**이며 한 쪽이 변경되어도 다른 쪽에 영향을 주지 않는다. 하지만 이들을 하나의 타입으로 묶는다면 하나의 논리적인 개념을 이룬다. 데이터 클래스는 이런 개념을 쉽게 만들어준다.

```python
from dataclasses import dataclass

@dataclass
class MyFraction:
  numerator: int = 0
  denominator: int = 1
```

아래 예시는 `dataclass`의 특성들은 보여주는 코드이다. `dataclass`로 평점, 도서, 도서 카테고리를 만들었다.

```python
import datetime
from dataclasses import dataclass
from enum import auto, Enum

class Currency(Enum):
  DOLLAR = auto()
  WON = auto()

@dataclass(eq=True, order=True, frozen=True) # 비교/정렬 가능(필드 순차 비교)
class Ratings:
  average_ratings: float = 0.0
  number_of_ratings: int = 0

print(sorted([Ratings(5.0, 1), Ratings(4.5, 2), Ratings(5.0, 10), Ratings(0.5, 1)]))
# [Ratings(average_ratings=0.5, number_of_ratings=1), Ratings(average_ratings=4.5, number_of_ratings=2), 
#  Ratings(average_ratings=5.0, number_of_ratings=1), Ratings(average_ratings=5.0, number_of_ratings=10)]

print(Ratings(5.0, 1) == Ratings(5.0, 1))
# True

@dataclass(frozen=True) # frozen 설정 시 변경 불가
class Book:
  title: str
  author: str
  price: float = 10.0
  currency: Currency = Currency.DOLLAR
  ratings: Ratings = Ratings()
  
@dataclass
class BookCategory:
  title: str
  books: set[Book]

  # 메서드 추가도 가능
  def get_book_titles(self):
    return [b.title for b in self.books]

algo_for_opt = Book('Algorithms for Optimization', 'Mykel J. Kochenderfer', 52.99, Currency.DOLLAR)
# > algo_for_opt.price = 50.0
# dataclasses.FrozenInstanceError: cannot assign to field 'price'
art_of_cp = Book('The Art of Computer Programming', 'Donald E. Knuth', 162900, Currency.WON)
ddd = Book('Domain-Driven Design', 'Eric Evans', 44.17, Currency.DOLLAR)

computer_category = BookCategory(
  'Computers',
  {algo_for_opt, art_of_cp, ddd}
)

print(algo_for_opt.price)
# 52.99

print(computer_category.get_book_titles())
# ['Algorithms for Optimization', 'The Art of Computer Programming', 'Domain-Driven Design']

print(str(computer_category))
# BookCategory(
#   title='Computers', 
#   books={
#     Book(title='The Art of Computer Programming', author='Donald E. Knuth', price=162900, currency=<Currency.WON: 2>, ratings=Ratings(average_ratings=0.0, number_of_ratings=0)),
#     Book(title='Domain-Driven Design', author='Eric Evans', price=44.17, currency=<Currency.DOLLAR: 1>, ratings=Ratings(average_ratings=0.0, number_of_ratings=0)), 
#     Book(title='Algorithms for Optimization', author='Mykel J. Kochenderfer', price=52.99, currency=<Currency.DOLLAR: 1>, ratings=Ratings(average_ratings=0.0, number_of_ratings=0))
#   }
# )
```

추가 정보는 `dataclass` [공식 문서](https://docs.python.org/ko/3/library/dataclasses.html)를 참고하자.