---
title: "Robust Python 13 - 견고한 파이썬"
excerpt: "Robust Python 13 - 견고한 파이썬"
date: 2022-11-24 01:00:00 +0900
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

## 13장. 프로토콜

[p.301] 덕 타이핑을 사용하면 어떤 상위 클래스나 사전 정의된 상속 구조가 필요없다. 타입 체커는 덕 타이핑을 어떻게 다뤄야 할지 정보가 전혀 없다. 이를 위해 파이썬 3.8부터 도입된 기능인 프로토콜을 소개한다. 프로토콜을 통해 덕 타입 변수에 어노테이션을 추가할 수 있다.  

메뉴를 반으로 나누는 아래와 같은 메서드 `split_dish`가 있다고 하자.  

```python
import math
def split_dish(dish: ???) -> ????:
  dishes = dish.split_in_half()
  assert len(dishes) == 2
  for half_dish in dishes:
    half_dish.cost = math.ceil(half_dish.cost) / 2
    half_dish.name = "½ " + half_dish.name
  return dishes
```

타입 체킹을 어떻게 해야할까?  

좋지 않지만 가능한 방법들: 타입을 비우거나 Any로 사용, Union 사용(지속 업데이트 필요), 상속 사용(클래스 간 복잡도 증가), 믹스인 사용(이상적인 해법은 아니다)

## 프로토콜

[p.308] 프로토콜은 타입 힌트와 런타임 타입 시스템 사이의 간격을 줄이는 방법을 제공하며 타입 체크 중에 구조적 하위 타입을 제공할 수 있다. 사실 이미 여러분은 프로토콜을 잘 활용하고 있을 수도 있다. 바로 이터레이터 프로토콜이다.

```python
from random import shuffle
from typing import MutableSequence, Iterator

class ShuffleIterator:
    def __init__(self, sequence: MutableSequence):
        self.sequence = list(sequence)
        shuffle(self.sequence)
    
    def __iter__(self):
        return self

    def __next__(self):
        if not self.sequence:
            raise StopIteration
        return self.sequence.pop(0)

my_list = [1, 2, 3, 4]
iterator: Iterator = ShuffleIterator(my_list)

assert {1,2,3,4} == {n for n in iterator}
```

위 코드에서 타입의 작동을 위해 `Iterator`의 하위 클래스화할 필요가 없음에 주목하라. 이는 `ShuffleIterator`가 이터레이터 수행을 위한 두 개의 메서드 `__iter__`과 `__next__`를 갖고 있기 때문이다.  

## 프로토콜의 정의

```python
from typing import Protocol, Tuple

class Splittable(Protocol):
    cost: int
    name: str

    def split_in_half(self) -> tuple['Splittable', 'Splittable']:
        # this is a literal ellipsis to indicate a stub function
        ... 
```

위와 같이 프로토콜을 정의할 수 있다. 만약 `BLTSandwich`를 잘라 나눌 수 있게 하고 싶다면 아래와 같이 구현하면 된다.

```python
class BLTSandwich:
    def __init__(self):
        self.cost = 6.95
        self.name = 'BLT'
        # This class handles a fully constructed BLT sandwich
        # ... 

    def split_in_half(self) -> tuple['BLTSandwich', 'BLTSandwich']:
        # Instructions for how to split a sandwich in half
        # Cut along diagonal, wrap separately, etc.
        # Return two sandwiches in return
        return (BLTSandwich(), BLTSandwich())
```

명시적인 클래스 상속이 없음에 주목하라.

```python
def split_dish(order: Splittable) -> tuple[Splittable, Splittable]:
```

`split_dish` 함수는 위와 같이 타입 체크를 사용할 수 있다.

## 런타임 시 체크할 수 있는 프로토콜

[p.314] 지금까지의 프로토콜 설명은 정적 검사만 다루었다. 하지만 런타임 시에 `isinstance()`나 `issubclass()`와 같은 함수를 통해 체크하고 싶을 수 있다. 하지만 기본 프로토콜에서는 이를 지원하지 않는다. 런타임 체크를 위해서는 `runtime_checkable` 데코라이터를 적용해야만 한다.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Splittable(Protocol):
    cost: int
    name: str

    def split_in_half(self) -> tuple['Splittable', 'Splittable']:
        # this is a literal ellipsis to indicate a stub function
        ... 

class BLTSandwich:
  ...

assert isinstance(BLTSandwich(), Splittable)
```

<div class="notice--warning" markdown="1">
Question. 어떻게 직접 상속받지도 않았는데 `isinstance`가 가능할까? `runtime_checkable` 데코레이터는 소스 코드는 다음과 같다.  

```python
def runtime_checkable(cls):
    if not issubclass(cls, Generic) or not cls._is_protocol:
        raise TypeError('@runtime_checkable can be only applied to protocol classes,'
                        ' got %r' % cls)
    cls._is_runtime_protocol = True
    return 
```

`Protocol`의 서브 클래스가 생성될 때 `Protocol`의 `__init_subclass__`가 호출되며, 여기에서 subclass 여부를 체크할 때 호출되는 `__subclasshook__`을 정의한다. `__subclasshook__(subclass)`에서는 `subclass`를 해당 베이스 클래스의 서브 클래스로 간주할 지를 검사할 수 있는데, 프로토콜에 정의된 attribute들이 `subclass` 타입에 정의되어 있는지를 검사한다. 아래는 소스 코드이다.

```python
class Protocol(Generic, metaclass=_ProtocolMeta):
    __slots__ = ()
    _is_protocol = True
    _is_runtime_protocol = False

    def __init_subclass__(cls, *args, **kwargs):
        super().__init_subclass__(*args, **kwargs)

        # Determine if this is a protocol or a concrete subclass.
        if not cls.__dict__.get('_is_protocol', False):
            cls._is_protocol = any(b is Protocol for b in cls.__bases__)

        # Set (or override) the protocol subclass hook.
        def _proto_hook(other):
            if not cls.__dict__.get('_is_protocol', False):
                return NotImplemented

            # First, perform various sanity checks.
            if not getattr(cls, '_is_runtime_protocol', False):
                if _allow_reckless_class_checks():
                    return NotImplemented
                raise TypeError("Instance and class checks can only be used with"
                                " @runtime_checkable protocols")
            if not _is_callable_members_only(cls):
                if _allow_reckless_class_checks():
                    return NotImplemented
                raise TypeError("Protocols with non-method members"
                                " don't support issubclass()")
            if not isinstance(other, type):
                # Same error message as for issubclass(1, int).
                raise TypeError('issubclass() arg 1 must be a class')

            # Second, perform the actual structural compatibility check.
            for attr in _get_protocol_attrs(cls):
                for base in other.__mro__:
                    # Check if the members appears in the class dictionary...
                    if attr in base.__dict__:
                        if base.__dict__[attr] is None:
                            return NotImplemented
                        break

                    # ...or in annotations, if it is a sub-protocol.
                    annotations = getattr(base, '__annotations__', {})
                    if (isinstance(annotations, collections.abc.Mapping) and
                            attr in annotations and
                            issubclass(other, Generic) and other._is_protocol):
                        break
                else:
                    return NotImplemented
            return True

        if '__subclasshook__' not in cls.__dict__:
            cls.__subclasshook__ = _proto_hook

        # We have nothing more to do for non-protocols...
        if not cls._is_protocol:
            return

        # ... otherwise check consistency of bases, and prohibit instantiation.
        for base in cls.__bases__:
            if not (base in (object, Generic) or
                    base.__module__ in _PROTO_ALLOWLIST and
                    base.__name__ in _PROTO_ALLOWLIST[base.__module__] or
                    issubclass(base, Generic) and base._is_protocol):
                raise TypeError('Protocols can only inherit from other'
                                ' protocols, got %r' % base)
        if cls.__init__ is Protocol.__init__:
            cls.__init__ = _no_init_or_replace_init

```

</div>
