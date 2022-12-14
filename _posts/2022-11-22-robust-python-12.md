---
title: "Robust Python 12 - 견고한 파이썬"
excerpt: "Robust Python 12 - 견고한 파이썬"
date: 2022-11-22 01:00:00 +0900
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

## 12장. 하위 타입

[p.279] 하위 타입을 올바르게 사용하면 코드베이스를 쉽게 확장할 수 있으며 코드베이스의 다른 부분들을 손상시킬 우려 없이 새로운 동작을 도입할 수 있다. 그러나 하위 타입을 만들 때는 신중해야 한다. 하위 타입의 관계를 잘 작성하지 못하면 예기지 못한 상황으로 인해 코드베이스의 견고성이 떨어질 수 있다.  

OOP에 관심이 있다면 『Head First Object-Oriented Analysis and Design』(O'Reilly, 2006)을 추천한다.
{: .notice--primary}

### 상속

[p.281] 하위 타입을 얘기할 때 대부분의 개발자는 바로 상속을 생각할 것이다. 상속은 다른 타입으로부터 새로운 타입을 만들어내는 방법이며 이때 오리지널 타입의 모든 동작은 새로운 타입으로 복제된다.
[p.285] 여러분은 새로운 동작을 기존 코드에 추가할 수 있으며 이때 기존 코드의 수정은 필요가 없다. 기존 코드 수정 작업을 하지 않아도 된다면 버그의 발생 가능성은 어마어마하게 줄어든다. 

### 다중 상속

[p.285] 파이썬에서는 여러 개의 클래스에서 상속받을 수 있지만, 이를 자주 쓰지는 말라. 하나의 클래스가 두 개의 분리된 불변 속성을 각각 베이스 클래스에서 상속받는다면 코드를 보는 사람에게 또다른 인지적 부담을 주게된다. 다만 다중 상속 사용을 추천하는 단 하나의 케이스가 있으니 바로 믹스인<sup>mixin</sup>이다. 믹스인은 제네릭 메서드들을 상속받을 수 있는 클래스다. 믹스인 베이스 클래스들은 기본적으로 불변 속성이나 데이터를 갖고 있지 않으며 메서드들로만 이뤄져 있다. 이 메서드들은 오버라이드용이 아니다. 단지 필요한 것을 제공하기 위한 클래스이다.

### 치환 가능성

[p.292] 리스코프 치환 원칙(LSP): 하위 타입은 상위 타입과 동일한 특성(동작)을 모두 준수해야 한다.

#### 불변 속성

하위 타입은 상위 타입의 모든 불변 속성을 갖고 있어야 한다. `Retangle`에서 `Square`를 하위 타입으로 사용했을 때 높이와 너비는 서로 독립적이라는 불변 속성을 무시했었다.

### 설계 고려 사항

#### 모든 오버라이딩된 메서드는 `super()`를 포함해야 한다.

[p.296] 오버라이딩된 메서드에서 `super()`를 호출하지 않으면 하위 클래스가 베이스 클래스와 정확히 동일하게 동작한다는 보장이 없다. `super()` 호출이 필요 없는 경우는 베이스 메서드가 비어있으며(추상 클래스의 경우) 이것이 코드의 수명주기 도안 계속 유지가 될 때다.

### 합성

[p.297] 상속을 사용하지 않아야 할 때를 아는 것도 중요하다. 내가 본 가장 큰 실수 중 하나는 상속을 코드 재사용 목적으로만 사용했던 것이다. 상속은 코드를 재사용하는 좋은 방법이지만 상속의 주된 이유는 상위 타입 대신 하위 타입이 사용되는 관계를 모델링하는 것이다.
[p.298] 합성은 엔티티 간 커플링이 약한 형태이기 때문에 재사용 메커니즘으로서 상속보다 선호되는 편이다.
