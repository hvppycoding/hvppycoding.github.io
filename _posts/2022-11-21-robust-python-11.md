---
title: "Robust Python 11 - 견고한 파이썬"
excerpt: "Robust Python 11 - 견고한 파이썬"
date: 2022-11-21 01:00:00 +0900
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

## 11장. 사용자 정의 인터페이스

[p.255] 코드 인터페이스의 패러독스: 여러분은 인터페이스를 설계할 작성할 한 번의 기회가 있다. 하지만 올바르게 설계되었는지는 써보기 전에는 모를 것이다.
개발자들은 여러분이 생성한 타입을 사용하자마자 의존성이 생기게 된다. 만약 여러분이 이전 버전과 호환되지 않는 변경을 시도 한다면 모든 호출 코드가 깨져버릴 수 있다.

## 자연스러운 인터페이스 설계

[p.257] 인터페이스 설계 목표는 사용자가 여러분의 인터페이스 사용이 자연스럽다고 느껴지게 하는 것이다. 코드가 쓰기 어렵다면 사용자가 비슷한 자신만의 클래스를 만들거나, 테스트 감소, 버그 발생 등의 문제로 이어질 수 있다.  
사용자 입장에서 생각하기 위해서 유용한 전략 몇 가지를 소개한다.

### 테스트 주도 개발

[p.259]테스트 주도 개발(TDD)은 실패하는 테스트 추가, 테스트 통과하는 코드 작성, 리팩토링의 루프를 기본으로 한다. TDD는 **설계 방법론**이다. 테스트는 중요하지만 이는 방법론의 부산물에 불과하다. 진짜 가치는 테스트가 인터페이스 설계를 얼마나 도와주는지에 있다.  
TDD를 사용하면 구현체를 작성하기 전에 어떻게 코드가 호출되는지를 미리 볼 수 있다.  
추가적인 장점으로 여러분의 테스트는 문서화의 한 형태로서 역할을 한다.  

### README 주도 개발

### 사용성 테스트

## 자연스런 상호작용

코드에 익숙하지 않더라도 도메인을 잘 아는 사람이라면 쉽게 이해할 수 있도록 인터페이스를 모델링하라.

### 자연스런 상호작용 예제

[p.263] 이번 장에서는 자동화된 식료품 픽업 서비스의 일부를 위한 일터페이스를 설계할 것이다. 사용자가 스마트폰으로 레시피를 스캔하면 앱이 자동으로 어떤 재료가 필요한지 찾아낸다. 사용자가 주문을 승인하면 앱은 지역 식료품점들에 재료의 가용을 묻는 쿼리를 보내고 그 결과에 따라 배송 일정을 잡는다.

![식료품 배달앱 워크플로우]({{site.baseurl}}/assets/images/2022-11-21-robust-python-11.drawio.svg){: .align-center}

```python
recipes: List[Recipe] = get_recipes_from_scans()

# 주문을 받기 위한 부분
order = ????

display_order(order)
wait_for_user_order_confirmation()
if order.is_confirmed():
  grocery_inventory = get_grocery_inventory()
  grocery_list = ????
  # TODO: 식재료를 미리 예약해 다른 사람들이 가져가지 못하게 한다.
  wait_for_user_grocery_confirmation(grocery_list)
  # TODO: 실제로 식재료가 주문됐는가?
  deliver_ingredient(grocery_list)
```

코드를 시작하기 전에 인터페이스를 의도적으로 디자인하는 습관을 들이자.  

1. 전달받은 각 레시피의 모든 재료를 한곳으로 취합한다. 이는 `Order`가 된다.
2. `Order`는 사용자 필요에 따라 추가/삭제할 수 있다. 하지만 사용자가 한 번 승인을 하면 수정은 불가능하다.
3. 주문이 승인되면 모든 재료를 확인해 어느 식료품점에 재고가 있는지 확인한다. 이것이 `Grocery List`가 된다.
4. `Grocery List`는 식료품점의 리스트 및 가게에서 주문해야 할 재료의 리스트로 구성되어 있다. 각 아이템은 주문을 넣기 전까지는 임시 예약돼 있다.
5. 사용자가 `Grocery List`를 승인하면 주문을 넣는다. 임시 보관된 항목들은 삭제되고 배달에 들어간다.

```python
from typing import Iterable, Optional
from copy import deepcopy

class Order:
  ''' 식재료 목록을 나타내는 Order 클래스 '''
  def __init__(self, recipes: Iterable[Recipe]):
    self.__confirmed = False
    self.__ingredients: set[Ingredient] = set()
    for recipe in recipes:
      for ingredient in recipe.ingredients:
        self.add_ingredient(ingredient)
  
  def get_ingredients(self) -> list[Ingredient]:
    # 복사본을 반환해 사용자들이 실수로 내부 데이터를 건드리지 않게 한다.
    return sorted(deppcopy(self.__ingredient), key=lambda ing: ing.name)

  def add_ingredient(self, ingredient: Ingredient):
    target_ingredient = self._get_matching_ingredient(ingredient)
    if target_ingredient is None:
      self.__ingredients.add(ingredient)
    else:
      # 아래 연산을 위해서는 Ingredient 클래스에 __add__를 구현해야 한다!
      target_ingredient += ingredient

  def __disallow_modification_if_confirmed(self):
    if self.__confirmed:
      raise OrderAlreadyFinalizedError('Order is confirmed - changing it is not allowed')
    
  def confirm(self):
    self.__confirmed = True
```

## 컨텍스트 관리자

[p.274] 식료품점 리스트를 처리할 때 주의할 사항은 다음과 같다.

1. `Grocery List`는 식료품점 리스트와 각 가게에서 구매할 식료품들의 리스트로 구성돼 있다. 각 식료품은 앱에서 주문을 완료할 때까지 임시 저장된다. 식료품들은 서로 다른 가게에서 올 수도 있으며 앱은 아이템을 가장 싸게 파는 가게를 찾아준다.
2. 사용자가 `Grocery List`를 확정하면 주문이 완료된다. 식료품점에 임시 저장된 목록들은 삭제되고 배달이 시작된다.

호출하는 코드 관점에서는 다음과 같이 쓸 수 있다.

```python
order = Order(recipes)

display_order(order)
wait_for_user_order_confirmation()
if order.is_confirmed():
  grocery_inventory = get_grocery_inventory()
  grocery_list = GroceryList(order, grocery_inventory)
  grocery_list.reserve_items_from_stores()
  wait_for_user_grocery_confirmation(grocery_list)
  if grocery_list.is_confirmed():
    grocery_list.order_and_unreserve_items()
    deliver_ingredients(grocery_list)
  else:
    grocery_list.unreserve_items()
```

하지만 위 인터페이스는 사용자가 코드를 오용하기 쉽다. 예를 들어 사용자가 주문을 확인하지 않는 등의 예외가 발생한다면? 이런 일이 발생하면 임시 저장 항목들이 지워지지 않고 남아있을 것이다. 내 코드를 호출하는 쪽에서 알아서 처리하기를 원할 수도 있으나, 이는 자칫 실수하기 쉽다.  

작업이 종료됐을 때 어떤 함수가 자동으로 동작하기를 기다리는 것은 파이썬에서는 자주 일어난다. 파일 열기/닫기, 세션 인증/로그아웃, 데이터베이스 명령 일괄 처리/제출이 그 예이다. 이런 확인은 `with` 구문을 통해 할 수 있다.

```python
with open(filename, 'r') as handle:
  print(handle.read())
```

위 코드는 `with` 구문이 종료될 때 열렸던 파일을 닫도록 구현되어 있다.  
식료품 목록 자동 삭제에 이를 활용하기 위해서 **컨텍스트 관리자**를 도입해야 한다.

```python
from contextlib import contextmanager

@contextmanager
def create_grocery_list(order: Order, inventory: Inventory):
  grocery_list = _GroceryList(order, inventory)
  try:
    yeild grocery_list
  finally:
    if grocery_list.has_reserved_items():
      grocery_list.unreserve_items()
```

`@contextmanager`로 데코레이션된 모든 함수는 `with` 구문과 함께 사용할 수 있다. `_GroceryList` 인스턴스를 만들었고 이를 `yeild`한다. 이는 수행을 멈추고 생산된 값을 호출 코드로 반환시킨다. 사용자는 이제 다음과 같이 사용할 수 있다.

```python
if order.is_confirmed():
  grocery_inventory = get_grocery_inventory()
  with create_grocery_list(order, grocery_inventory) as grocery_list:
    grocery_list.reserve_items_from_stores()
    wait_for_user_grocery_confirmation(grocery_list)  
    grocery_list.order_and_unreserve_items()
    deliver_ingredients(grocery_list)
```

### 마치며

[p.278] 여러분의 타입이 나타내는 도메인의 개념과 사용자가 이런 타입과 상호작용하는 방식을 고민해야 한다. 구축된 인터페이스는 직관적으로 느껴져야한다. 적용하기는 쉬워야하고 사용자의 오용은 막아야 한다.
