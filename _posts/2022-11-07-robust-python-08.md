---
title: "Robust Python 08 - 견고한 파이썬"
excerpt: "Robust Python 08 - 견고한 파이썬"
date: 2022-11-07 00:00:00 +0900
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

## 2부. 사용자 정의 타입

[p.187] 사용자 저으이 타입은 코드베이스 내의 도메인 개념을 전달하는 역할을 한다.

- 열거형: 제약된 값들의 집합을 나타낸다.
- 데이터 클래스: 서로 다른 개념(클래스 인스턴스, 변수 등)의 관계를 나타낸다.
- 클래스: 서로 다른 개념 간의 관계를 나타내며 불변성을 보존해야 한다.(데이터 클래스와 차이는?)

학생의 시험 점수를 계산하는 함수가 다음과 같다고 생각해보자. 두 함수 중 어떤 함수를 사용하고 싶은가?

```python
def calculate_total_test_score(student: tuple[str, str],
                               subtotal: float) -> float:
  return subtotal * (1 + score_lookup[student[1]])

def calculate_total_test_score(student: Student,
                               subtotal: decimal.Decimal) -> decimal.Decimal:
  return subtotal * (1 + score_lookup[student.id])
```

훌륭한 도메인 연관 타입을 세움으로써 좀 더 건강한 시스템을 구축할 수 있다.

## 8장. 사용자 정의 타입: 열거형

변수가 특정한 값만 가질 수 있도록 하고 싶을 때가 있을 것이다.
예를 들어 게임 캐릭터가 몇 가지 상태(정지, 오른쪽 이동, 왼쪽 이동, 점프 중)만 가질 수 있다고 하자.
열거형(`Enum`)을 사용할 수 있다.

```python
from enum import Enum

class UnitState(Enum):
  STOP = 'Stop'
  GOING_RIGHT = 'GoingRight'
  GOING_LEFT = 'GoingLeft'
  JUMPING = 'Jumping'

# Enum 사용하기
UnitState.JUMPING

# Enum 이터레이션
for state_number, state in enumerate(UnitState, start=1):
  print(f'State {state_number}: {state.value}')
# State 1: Stop
# State 2: GoingRight
# ...

def set_unit_state(state: UnitState):
  ...
```

열거형은 정적 선택 항목을 전달하는 데 유용하다. 하지만 선택이 런타임에 결정되는 때에는 사용하지 않는 것이 좋다.
{: .notice--warning}

### 플래그

`Enum`에서 비트 연산이 필요하다면 `Flag`를 사용할 수 있다.

```python
from enum import Flag, auto

class UnitState(Flag):
  STOP = auto() # auto()는 자동으로 값을 할당한다.
  GOING_RIGHT = auto()
  GOING_LEFT = auto()
  JUMPING = auto()
  JUMPING_RIGHT = GOING_RIGHT | JUMPING
  JUMPING_LEFT = GOING_LEFT | JUMPING
```
