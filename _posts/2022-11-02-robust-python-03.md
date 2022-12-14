---
title: "Robust Python 03 - 견고한 파이썬"
excerpt: "Robust Python 03 - 견고한 파이썬"
date: 2022-11-03 00:30:00 +0900
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

## 3장. 타입 어노테이션

### 타입 어노테이션이란?

[p.79] 가용 직원들과 레스토랑의 오픈 시간을 파라미터로 받ㅇ차 그날 가능한 작업자를 스케줄링하는 코드를 생각해보자. 이 코드는 다음과 같이 사용하면 좋을 것 같다.

`def schedule_restaurant_open(optn_time, workers_needed):`

세부 코드를 읽기 전에 어떤 타입의 인자들이 전달돼야 하는지 생각해보라. `open_time`은 `datetime`일까? `workers_needed`는 이름의 리스트일까? 아니면 `Worker` 객체의 리스트일까? 아니면 다른 것일까? 이런 추측이 불확실하다면 코드의 호출이나 수행 부분의 점검이 필요하다.

```python
import datetime
import random

def schedule_restaurant_open(open_time: datetime.datetime, 
                             workers_needed: int):
    workers = find_workers_available_for_time(open_time)
    # use random.sample to pick X available workers
    # where X is the number of workers needed.
    for worker in random.sample(workers, workers_needed):
        schedule(worker, open_time)
```

타입 어노테이션을 볼 수 있다면 어떻게 동작될지 여러분은 바로 예상할 것이다. 이렇게 타입 어노테이션은 인지적인 오버헤드를 줄여주며 유지 보수 담당자와의 마찰도 줄여준다.

타입 어노테이션이 없는 코드를 만난다면 파라미터명을 `number_workers_needed`처럼 바꾸는 것을 생각해보라.
{: .notice--primary}

다시 코드를 보면 코드 중반부에서 `find_workers_available_for_time`을 호출한다. 이 함수에서 반환되는 타입을 한 눈에 알 수 없다.

```python
def find_workers_available_for_time(open_time: datetime.datetime):
    workers = worker_database.get_all_workers()
    available_workers = [worker for worker in workers
                         if is_available(worker)]
    if available_workers:
        return available_workers

    # fall back to workers who listed they are available in
    # in an emergency
    emergency_workers = [worker for worker in get_emergency_workers()
                         if is_available(worker)]git
    if emergency_workers:
        return emergency_workers

    # Schedule the owner to open, they will find someone else
    return [OWNER]
```

[p.82] 이 코드에는 3개의 반환 구문이 있으며, `work_database`, `is_available`, `get_emergency_workers`, `OWNER`의 타입을 확인해야 한다. 
반환 타입의 어노테이션을 선언하여 이를 해결할 수 있다.

```python
def find_workers_available_for_time(open_time: datetime.datetime) -> list[str]:
```

<div class="notice--warning" markdown="1">
파이썬 3.8 및 그 이전 버전에서 `list`, `dict`, `set` 같은 내장 컬렉션 타입들은 문법에 `list[CookBook]`, `dict[str, int]`와 같은 브래킷을 허용하지 않았다. 대신 타입 모듈로부터의 타입 어노테이션을 써야 했다
```python
from typing import Dict, List
def count_authors(cookbooks: List[Cookbook]):
  ...
```
</div>

### 타입 어노테이션의 장점: 자동 완성

[p.84] 파이썬은 동적 타입 언어이기 때문에 어떤 연산이 가능한지 예측하기는 쉽지 않다. 하지만 타입 어노테이션이 돼 있다면 많은 파이썬을 인식하는 IDE들이 변수들의 연산을 쉽게 예측하고 자동 완성 기능을 제공할 수 있다.

### 타입 어노테이션의 장점: 타입 체커

[p.85] 타입 체커는 타입 어노테이션을 체크하는 툴이다. mypy는 제일 많이 쓰이는 타입 체커다.

```bash
# mypy 패키지 설치
pip install mypy

# 타입 오류 탐지
mypy 파일명.py
```

### 타입 어노테이션 적용 조건

[p.90] 타입 어노테이션은 잘못하면 너무 과하게 적용되기 쉽다. 특히 타입이 분명한 변수들은 일일이 어노테이션을 할 필요가 없고, 프라이빗 메서드에 대한 파라미터까지 어노테이션할 필요도 없다. 언제 타입 체커를 사용해야 할까?

- 다른 모듈이나 사용자가 호출할 가능성이 높은 함수
- 타입이 복잡한 곳 또는 비직관적인 곳
- mypy가 타입이 필요하다고 하는 경우
