---
title: "Robust Python 10 - 견고한 파이썬"
excerpt: "Robust Python 10 - 견고한 파이썬"
date: 2022-11-09 01:00:00 +0900
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

## 10장. 사용자 정의 타입: 클래스

[p.228] 클래스는 서로 연관된 데이터를 그룹화하는 또 하나의 방식이다. 사실은 `dataclass`에서처럼 클래스를 작성할 수 있다.

```python
class Person:
    name: str = 'Person'
    years_experience: int = 0
    address: str = ''

person = Person()

person1 = Person()
person1.name = 'Person1'

person2 = Person()
person2.name = 'Person2'

print(person.name)
# Person
print(person1.name)
# Person1
print(person2.name)
# Person2

person3 = Person('Person3', 10, '123')
# TypeError: Person() takes no arguments
```

### 생성자

앞의 코드 예에서 마지막 라인은 오류가 발생한다. 클래스는 생성자<sup>Constructor</sup>라는 특수한 메서드를 통해 클래스를 어떻게 초기화할지 정의해야 한다.

 ```python
class Person:
  def __init__(self,
                name: str,
                years_experience: int,
                address: str):
    self.name = name
    self.years_experience = years_experience
    self.address = address

person3 = Person('Person3', 10, '123')
```

[p.230] 그러면 왜 클래스를 사용할까? 딕셔너리나 데이터 클래스는 클래스보다 간단하다. 하지만 클래스는 딕셔너리나 클래스가 전달할 수 없는 키 값 하나를 전달하는데, 이 값이 바로 **불변 속성**<sup>invariant</sup>이다.

### 불변 속성

불변 속성이란 **해당 엔티티의 수명주기 동안 변하지 않는 엔티티 속성**을 의미한다. 불변 속성은 코드를 이해하는 데 기초를 제공한다. 다음은 불변 속성의 예시들이다.

- 모든 직원은 중복되지 않는 고유한 ID를 가진다.
- 게임에서 적들은 체력이 0보다 클 때만 행동을 취할 수 있다.
- 원의 반지름은 항상 양수다.

다음과 같은 불변 속성들을 갖는 피자 예시를 생각해보자(이 불변 속성들은 일반적 관점에서는 참이 아닐 수도 있으며 내가 만드는 시스템에서만 참인 것이다).

- 소스는 토핑 위에 올라갈 수는 없다.
- 토핑은 치즈 위 또는 아래에 올라갈 수 있다.
- 항상 단일 소스만 사용한다.
- 도우의 반지름은 항상 정수다.
- 도우의 반지름은 6인치에서 12인치 사이다.

```python
from typing import List

def is_sauce(t):
    return t in ['Tomato Sauce', "Olive Oil"]

class PizzaSpecification:
  def __init__(self,
                dough_radius_in_inches: int,
                toppings: list[str]):
    assert 6 <= dough_radius_in_inches <= 12, 'Dough must be between 6 and 12 inches'
    sauces = [t for t in toppings if is_sauce(t)]
    assert len(sauces) < 2, 'Can only have at most one sauce'

    self.__dough_radius_in_inches = dough_radius_in_inches
    sauce = sauces[:1]
    self.__toppings = sauce + [t for t in toppings if not is_sauce(t)]
```

 클래스내 불변 속성이 깨질 것 같다면 절대 이 클래스는 생성하지 말아야 한다. 예외를 발생시키거나(`AssertionError` 발생시킨 것처럼), 데이터를 적절히 변경(앞에서 `toppings` 순서를 재배열한 것처럼)해야 한다.

### 불변 속성이 유용한 이유

다루는 데이터가 다음의 경우 중 하나에 속한다면 불변 속성을 갖고 있는 것이며 클래스를 작성해야한다.

- 타입 시스템에서 미처 캐치하지 못한 형태(예를 들어 토핑 순서)에 제약이 되는 데이터들이 있는가?
- 상호의존적인 필드들이 있는가?(예를 들어 하나의 필드 변경 시 다른 필드도 같이 변경해야 하는 경우)
- 데이터에 필요한 보증이 있는가?

클래스를 작성하기로 했다면 다음을 얻을 수 있다.

1. DRY<sup>Don't Repeat Yourself</sup> 원칙을 지키고 있다. 객체 생성 전 여러 체크 코드를 반복하는 대신 한 곳에 모을 수 있다.
2. 코드는 한 번 작성되면  오래 지속될 가능성이 높으며, 불변 속성을 제공해 따라다니는 부담을 줄일 수 있다.
3. 더 효과적으로 코드를 추론할 수 있게 된다. 잘 작성된 클래스는 편안함과 신뢰성을 제공할 수 있다.

다음과 같은 딕셔너리로는 앞서 말한 사양들을 보장하기 어렵고, 사용자가 실행 시마다 정확히 알고 호출을 해주어야만 한다.

```python
{
  "dough_radius_in_inches": 7,
  "toppings": ["tomato sauce", "mozzarella", "pepperoni"]
}
```

불변 속성의 장점 중 하나는 단일 책임 원칙<sup>Single Responsibility Principle, 각 객체는 한 가지의 이유로만 변경되어야 한다. 그러려면 한 가지의 책임만 가져야 한다.</sup>과 관련이 있다. 연관된 불변 속성 집합을 정의하고 이를 하나의 클래스로 작성하는 방식을 추천한다.

### 불변 속성을 통한 커뮤니케이션

[p.239] 불변 속성으로 커뮤니케이션을 효과적으로 하지 않으면 불변 속성의 장점을 깨닫기 어렵다. 커뮤니케이션의 상대는 크게 다음과 같이 두 가지로 나눌 수 있으며, 이 두 대상을 염두에 두면서 클래스를 설계해야 한다.

1. 클래스의 소비자: 자신의 문제를 해결하기 위한 도구를 찾고 있는 사람들이다.
2. 미래의 클래스 유지 보수자: 클래스를 수정하는 사람들이다. 이들이 모든 호출자가 의존하는 불변 속성들을 깨지 않도록 하는 것이 중요하다.

### 클래스의 소비

[p.240] 생성자에 가정 설정 구문을 삽입하는 것은 사용자들에게 무엇이 가능하고 무엇이 불가능할지 알려주는 최상의 방법이다. 지식을 전달할 때에는 위키로 전달하는 방법도 있는데, 이는 자주 업데이트되는 대규모 아이디어에 더 적합하다. 코드 저장소의 README 파일은 이보다 더 나은 방법이긴 하지만 가장 좋은 방법은 클래스 자체에 주석을 달거나 문서화 문자열<sup>docstring</sup> 을 작성하는 것이다.

```python
class Specification:
  """
  This class represents ...
  """
  def __init__(self, ...):
    ...
```

코드가 하는 일을 절대적으로 자체 문서화를 해야 한다. → 산발적인 주석(comment)이 아니라 문서화 문자열(docstring)로 코드화 되지 않은 불변 속성까지도 포함해서 문서화하자.

### 유지 보수자들은 어떻게?

불변 속성을 변경하면 모든 클래스의 소비자들에게 영향을 주기 때문에 불변 속성의 변경은 민감한 일이다. 이런 변경을 캐치하려면 단위 테스트에 의존해야 한다. 단위 테스트를 작성할 때 클래스의 의도와 불변 속성에 대해 단위 테스트를 작성하는 것에 더해, 미래의 테스트 작성자가 언제 불변 속성이 깨졌는지 알 수 있도록 하자. `with` 블록이 끝났을 때 코드를 수행시키는 컨텍스트 관리자를 사용하여 테스트를 작성할 수 있다.

```python
import contextlib

class Person:
  def __init__(self, name: str, age: int):
    assert len(name) > 0, 'Invalid name'
    assert age >= 0, 'age >= 0'
    self.name = name
    self.age = age

@contextlib.contextmanager
def create_person(name: str, age: int):
  person = Person(name, age)
  yield person

  # 불변 속성 확인을 위한 assertion문들
  assert len(person.name) > 0
  assert person.age >= 0

def test_person_operations():
  with create_person('happy', 5) as person:
    # Do something with person
    person.age = person.age - 10

if __name__ == '__main__':
  # AssertionError 발생(assert person.age > 0)
  test_person_operations()
```

### 캡슐화

캡슐화란 호출자로부터 데이터 접근이나 변경 방식을 제어하는 것을 의미한다. 이는 API<sup>Application Programming Interface</sup>라고 할 수 있다. 모든 함수 호출, 모든 속성의 접근, 모든 초기화는 객체의 API에서 하는 주요 부분이다.

### 데이터 접근의 보호

- `public`: 누구나 접근 가능
- `protected`: 하위 클래스만 접근 가능(파이썬에서는 언더스코어 1개를 변수명 앞에 붙인다.)
- `private`: 해당 클래스 내에서만 접근 가능(파이썬에서는 언더스코어 2개를 변수명 앞에 붙인다.)

파이썬에서는 Protected, Private에 대해 실제로 접근 제어를 막지는 않는다. 일종의 약속이라고 생각하자. `protected`와 `private`으로 설정하면 클래스의 `help()` 명령에서 보이지 않는다. `private` 멤버의 경우 네임 맹글링을 통해 접근이 어렵게 만든다. (하지만 아래 예제에서 보듯 접근이 불가능하진 않다.)

```python
class Person:
  def __init__(self, name: str, age: int):
    self._name = name
    self.__age = age

if __name__ == '__main__':
  person = Person('happy', 30)
  print(person.__dict__)
  # {'_name': 'happy', '_Person__age': 30}
  person._Person__age = 100
  print(person._Person__age)
  # 100
```

### 운영

현재 나이에서 나이를 늘리거나 줄일 수 있는 함수를 추가해야할수도 있다. 단순히 나이를 조정하는 메서드를 추가하는 것은 쉬울 것이다. 하지만 불변 속성들이 깨지지 않도록 해야한다.

```python
class Person:
  def __init__(self, name: str, age: int):
    assert len(name) > 0, 'Invalid name'
    assert age >= 0, 'age >= 0'
    self._name = name
    self.__age = age

  def adjust_age(self, adjust: int):
    '''
    Adjust age of the person
    All rules for person construction
    '''
    if self.__age + adjust < 0:
        raise Exception('Violating age >= 0')
    self.__age += adjust
```

필자는 `staticmethod`와 `classmethod`를 자주 사용하지 않는다고 한다. 모듈 레벨 범위에서의 자유 함수를 사용하는 편이 더 강력하고, 이동하기 쉬우며 하위 타입의 메서드 오버라이드에 대해 걱정할 필요도 없다.
{: .notice--warning}

### 마치며

[p.253] 대부분의 개발자가 클래스의 의미에 대한 고려 없이 클래스를 너무 많이 사용하곤 한다. 생성할 사용자 정의 타입을 결정할 때 다음의 가이드를 참고하길 바란다.  

![적절한 추상화의 선택]({{site.baseurl}}/assets/images/2022-11-09-robust-python-10.svg){: .align-center}
