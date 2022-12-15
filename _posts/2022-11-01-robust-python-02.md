---
title: "Robust Python 02 - 견고한 파이썬"
excerpt: "Robust Python 02 - 견고한 파이썬"
date: 2022-11-02 02:00:00 +0900
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

## 2장. 파이썬 타입의 소개

[p.61] 파이썬 코드의 유지 보수성을 확보하려면 타입<sup>type</sup>의 속성을 파악하고 신중하게 사용해야 한다. 타입이란 무엇인가? 나는 타입을 간단하게 정의할 것이다. 타입은 커뮤니케이션의 수단이다. 타입은 정보를 전달한다. 타입은 컴퓨터와 사용자가 추론할 수 있는 표현을 제공한다.

[p.66] 좀 더 구체적인 타입을 사용하는 것은 가능한 연산 및 해당 타입에서의 제약 사항을 미래의 기여자들에게 알리는 것이다. 완전 자동화된 부엌의 오픈 및 클로징을 다루는 코드베이스를 인계받는다고 해보자. 여러분은 클로징 시간을 변경할 수 있는 기능 추가를 의뢰받았다.  

```python
def close_kitchen_if_past_cutoff_time(point_in_time):
    if point_in_time >= closing_time():
        close_kitchen()
        log_time_closed(point_in_time)
```

이 코드에서 `point_in_time`을 변경을 해야 한다는 것을 알 것이다. 그런데 어디에서부터 시작해야 할까? 어떤 타입으로 이를 다뤄야 할까? `str`, `int`, `datetime` 중 하나일까? 책임감 있는 개발자는 테스트, 문서 또는 호출 코드를 검색할 것이다. 이 개발자는 `closing_time()`과 `log_time_closed()`가 어떤 타입을 받고 반환하는지에 주목할 것이다. 이 경우 이는 올바른 방법이지만 여전히 차선책인데, 오류는 막을 수 있지만 여전히 코드를 살펴보는 데 시간을 잡아먹어 배포를 지연시키기 때문이다. 이런 작은 예들이 한 번만 일어난다면 그러려니 넘어갈 수 있겠지만, 이런 이슈들이 쌓인다면 배포까지 힘든 여정을 거쳐야할 수 있다.

근본 원인은 파라미터(매개변수)의 의미적 표현을 불분명하게 한 것에 있다. 코드를 작성할 때 타입을 통해 여러분의 의도를 전달할 수 있으면 그렇게 해야 한다. 이를 주석으로 할 수도 있지만 나는 타입 어노테이션<sup>type annotations</sup>(파이썬 3.5부터 지원)을 통해 전달하기를 권장한다.

```python
def close_kitchen_if_past_cutoff_time(point_in_time: datetime.datetime):
    if point_in_time >= closing_time():
        close_kitchen()
        log_time_closed(point_in_time)
```

### 덕 타이핑

> 어떤 새가 오리처럼 걷고 오리처럼 소리를 낸다면 이 새는 오리일 것이다.

[p.73] 덕 타이핑이란 프로그래밍 언어에서 객체와 엔티티들이 어떤 인터페이스를 갖고 있다면 그 인터페이스의 타입처럼 쓸 수 있는 것을 의미한다.
예를 들어 `__iter__` 속성이 존재한다면 `for` 구문으로 Iteration이 가능하다.

[p.74] 덕 타이핑은 양날의 검과 같다. 덕 타이핑은 결합성을 높여주며 다중 타입을 처리할 수 있는 견고한 추상화 라이브러리의 구축은 복잡한 특수 사례의 필요성을 감소시킨다. 하지만 덕 타이핑을 과도하게 사용하면 개발자가 신뢰할 수 있는 가정은 허물어지기 시작한다. 특히 코드를 업데이트하는 경우에는 모든 호출 코드를 살펴보고 함수에 전달된 타입들이 새로운 업데이트를 충족하는지 확인해야 한다.
