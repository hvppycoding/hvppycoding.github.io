---
title: "Robust Python 09 - 견고한 파이썬"
excerpt: "Robust Python 09 - 견고한 파이썬"
date: 2022-11-07 01:00:00 +0900
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