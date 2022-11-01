---
title: "Robust Python 01 - 견고한 파이썬"
excerpt: "Robust Python 01 - 견고한 파이썬"
date: 2022-11-01 23:00:00 +0900
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

# 1장. 견고한 파이썬

[p.33] 이 책에서는 강력한 시스템을 구축하는 방법을 배운다. 코드베이스는 항상 유지 관리가 되는 것이 아니기 때문에 새로운 상황에 적응을 잘 할 수 있어야 한다. 향후의 유지 관리자는 자신이 견고한 코드베이스에서 작업한다는 믿을이 있어야 하며 여러분의 코드베이스는 이런 강점을 전달해야 한다. 여러분의 파이썬 코드는 향후의 유지 관리자가 코드를 분리하고 재구성을 할지라도 실패를 줄이도록 작성돼야 한다.

[p.38] 유지 관리가 가능한 클린 코드를 위해 노력해야 하는 이유는 무엇일까? 왜 견고성을 신경 써야 할까? 대답은 소통에 있다. 여러분은 정적 시스템을 전달하는 것이 아니다. 코드는 계속 변경된다. 그리고 유지 보수 관리자도 시간이 지남에 따라 바뀐다는 것을 생각해야 한다. 코드를 작성하는 목적은 가치의 전달이며 다른 개발자 역시 당신의 코드를 통해 신속한 가치 전달을 할 수 있어야 한다.

예제 1. 이해하기 어려운 코드
```python
# adjust_recipe_wrong.py

# Take a meal recipe and change the number of servings
# by adjusting each ingredient
# A recipe's first element is the number of servings, and the remainder
# of elements is (name, amount, unit), such as ("flour", 1.5, "cup")
def adjust_recipe(recipe, servings):
    old_servings = recipe.pop(0)
    factor = servings / old_servings
    new_recipe = {ingredient: (amount*factor, unit) 
                  for ingredient, amount, unit in recipe} 
    new_recipe["servings"] = servings
    return new_recipe

    
def test_adjust_recipe():
    old_recipe = [2, ("flour", 1.5, "cups")]
    adjusted = adjust_recipe(old_recipe, 4)
    assert {"servings": 4, "flour": (3, "cups")} == adjusted
    
    # THIS IS WRONG BEHAVIOR, we should have emptied the list
    assert old_recipe != []


test_adjust_recipe()
```

예제 2. 개선된 코드
```python
# adjust_recipe.py

# Take a meal recipe and change the number of servings
# by adjusting each ingredient
# A recipe's first element is the number of servings, and the remainder
# of elements is (name, amount, unit), such as ("flour", 1.5, "cup")

def adjust_recipe(recipe, servings):
    new_recipe = [servings]
    old_servings = recipe[0]
    factor = servings / old_servings
    recipe.pop(0)
    while recipe:
        ingredient, amount, unit = recipe.pop(0)
        # please only use numbers that will be easily measurable
        new_recipe.append((ingredient, amount * factor, unit))
    return new_recipe


def test_adjust_recipe():
    old_recipe = [2, ("flour", 1.5, "cups")]
    adjusted = adjust_recipe(old_recipe, 4)
    assert [4, ("flour", 3, "cups")] == adjusted
    assert old_recipe == []


test_adjust_recipe()
```

예제 3. 더 개선된 코드
```python
# adjust_recipe_improved.py
import copy

from dataclasses import dataclass
from fractions import Fraction
from typing import List

@dataclass
class Ingredient:
    name: str
    amount: Fraction
    units: str

    def adjust_proportion(self, factor: Fraction):
        self.amount *= factor

@dataclass
class Recipe:
    servings: int
    ingredients: list[Ingredient]

    def clear_ingredients(self):
        self.ingredients.clear()

    def get_ingredients(self):
        return self.ingredients
    
def adjust_recipe(recipe, servings):
    """
    Take a meal recipe and change the number of servings
    :param recipe: a <code>Recipe<code> indicating what needs to be adjusted
    :param servings: the number of servings
    :return Recipe: a recipe with serving size and ingredients adjusted
                    for the new servings.
    """
    # create a copy of the ingredients
    new_ingredients = list(recipe.get_ingredients())
    recipe.clear_ingredients()
    for ingredient in new_ingredients:
        ingredient.adjust_proportion(Fraction(servings, recipe.servings))
    return Recipe(servings, new_ingredients)


def test_adjust_recipe():
    old_recipe = Recipe(2, [Ingredient('flour', 1.5, 'cups')])
    adjusted = adjust_recipe(old_recipe, 4)
    assert Recipe(4, [Ingredient('flour', 3, 'cups')]) == adjusted
    assert old_recipe.ingredients == []


test_adjust_recipe()
```
[p.41] 주석에는 원래의 의도가 잘 드러나 있다. 
- `Recipe` 클래스를 사용하고 있다. 이를 통해 몇 가지 작업을 추상화할 수 있다.
- 인원수<sup>Servings</sup>는 특별한 경우로 처리됐던 리스트의 첫 번째 요소가 될 필요가 없이 `Recipe` 클래스의 명시적 부분이 됐다. 이는 호출 코드를 크게 단순화하고 우발적인 충돌을 방지한다.
- 내가 이전 레시피의 재료들을 삭제하려는 의도가 분명하게 드러났다. `pop(0)`이 필요한 이유도 분명해졌다.

[p.43] 비용은 소통을 위한 노력을 측정한 것이다. 여러분은 제공되는 가치의 소통을 위한 시간과 비용이 얼마나 들었는지 측정해야 한다. 여러 분이 가치를 만들려면 코드만 작성하고 다른 소통 채널을 제공하지 않는 것이 기본이 돼야 한다. 추가적인 소통 채널이 발생할 때 비용을 평가하기 위한 팩터는 다음과 같다.
- 발견 가능성: 쉽게 검색이 되는가?
- 유지 비용: 정확한가? 정보를 얼마나 자주 업데이트 해야하는가? 정보가 업데이트되지 않으면 어떤 문제가 발생하는가?
- 생산 비용: 소통을 만들기 위한 시간과 비용은 얼마나 드는가?

아래 그림은 몇 가지 소통 방법에 대한 비용과 근접성의 관계를 보여준다.

![소통 방법의 비용 및 근접성 그래프]({{site.baseurl}}/assets/images/2022-11-01-robust-python-01.svg)

[p.46] 여러분의 코드베이스는 여러분의 결정이나 의견, 작업 방식 등을 명확하게 전달하는 데 최적의 수단이 될 수 있다.  
하지만 이런 가정이 성립하려면 코드는 소비를 위한 비용 역시 낮아야 한다. 다시 말하면 여러분의 의도가 코드에 고스란히 전달돼야 하며, 여러분의 목표는 코드를 읽는 사람이 이해하기까지의 시간을 줄이는 것이다. 이상적인 관점에서는 사람들은 여러분의 실행 결과를 읽을 필요는 없으며 함수 시그니처<sup>function signature</sup>만 읽으면 된다. 잘 정의된 타입, 주석, 변수명의 사용을 통해 함수 시그니처는 여러분의 코드가 무슨 일을 하는지 투명하게 보여줘야 한다.

그래프를 보고 잘못 생각하기 쉬운 것이 "내가 원하는 것이 바로 셀프 도큐멘팅 코드군"이라고 판단하는 것이다(제일 낮은 근접성과 비용에 위치하기 때문). 물론 코드는 그 자체로 무엇이 수행될지 명확히 기록돼야 한다. 하지만 이것으로 모든 소통의 케이스를 커버할 수 없다. 예를 들어 버전 관리는 변경 이력의 정보를 제공한다. 설계 문서는 하나의 파일에 국한되지 않는 포괄적인 그림을 얘기한다. 회의는 실행 계획을 맞춰주는 중요한 이벤트가 될 수 있다. 발표는 수많은 청중에게 한 번에 내용을 전달할 수 있다. 이 책은 코드에서 무엇을 할 수 있을지를 다루고 있지만 다른 소통 수단의 가치도 저버리지 않기 바란다.
{: .notice--warning}
