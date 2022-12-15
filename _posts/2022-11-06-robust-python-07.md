---
title: "Robust Python 07 - 견고한 파이썬"
excerpt: "Robust Python 07 - 견고한 파이썬"
date: 2022-11-06 20:30:00 +0900
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

## 7장. 실용적 타입 체킹

### MonkeyType

MonkeyType은 파이썬 코드에 자동으로 어노테이팅을 해주는 도구다.  
[깃허브 저장소](https://github.com/pviafore/RobustPython/tree/master/code_examples/chapter7)의 소스 코드를 사용해서 실습을 진행해보자.  

```bash
# MonkeyType 패키지 설치하기
pip install monkeytype

# MonkeyType 수행하기: 해당 파일을 수행하며 SQLite 데이터베이스에 호출 정보를 저장한다.
monkeytype run main.py

# MonkeyType 결과 스텁 파일(함수 시그니처 파일) 확인하기
monkeytype stub pasta_with_sausage
```

stub 출력 결과는 다음과 같을 것이다.

```python
from automated_recipe_maker import Ingredient
from typing import List

def adjust_recipe(recipe: Recipe, servings: int) -> Recipe: ...

def make_pasta_with_sausage(servings: int) -> None: ...

class Receptacle:
    def __init__(self, name: str) -> None: ...
    def add(self, ingredient: Ingredient) -> None: ...
    def remove_ingredients(self, to_ignore: List[str] = ...) -> Ingredient: ...

class Recipe:
    def __init__(self, servings: int, ingredients: List[Ingredient]) -> None: ...
    def clear_ingredients(self) -> None: ...
    def get_ingredient(self, name: str) -> Ingredient: ...
```

결과에 만족한다면 다음 명령어로 실제 파일에 반영할 수 있다.

```bash
monkeytype apply [module명]
```

<div class="notice--warning" markdown="1">
`automated_recipe_maker` 모듈과 `pasta_with_sausage` 모듈 모두 어노테이션을 적용하면 그 이후 `NameError: name 'Receptacle' is not defined`  에러가 발생하며 코드 실행이 되지 않는다. 이는 두 파일 사이에서 Circular Import가 발생하기 때문이다. `automated_recipe_maker.put_receptacle_on_stovetop`에서 `pasta_with_sausage.Receptacle`를 argument로 사용하기 때문에 어노테이션을 위해 이를 import한다. 그런데 `pasta_with_sausage` 모듈에서도 `automated_recipe_maker.Ingredient`를 사용하기 때문에 이를 import한다. 아래와 같은 코드를 추가하여 문제를 해결하였다.

```python
# automated_recipe_maker.py
from __future__ import annotations # Python 3.11 부터는 default 포함된다.
from typing import List, Union, TYPE_CHECKING

if TYPE_CHECKING:
    from pasta_with_sausage import Receptacle
...
```

</div>

코드베이스에 대한 어노테이션을 생성한 후에는 `Union`이 적용된 부분을 검사해보자. `Union`은 함수 코드에 하나 이상의 타입이 전달됨을 의미한다. 의도한 동작인지 확인해볼 필요가 있다.

### Pytype

pytype은 현재(2022-11-06) 윈도우 OS를 지원하지 않는다. Windows is currently not supported unless you use WSL(2022-11-06).
{: .notice--danger}

MonkeyType의 문제 중 하나는 런타임 시에만 타입 어노테이션이 가능하다 점이다. 코드를 수행해보기 어려운 환경에서는 사용하기 어렵다. Pytype은 정적 분석을 통해 어노테이션을 수행한다.  

```bash
pip install pytype

# 폴더 대상으로 수행한다.
pytype code_example/chapter7
```

Pytype은 단지 타입 어노테이션뿐 아니라 타입 체커 및 린터 역할도 한다. Pytype은 추론을 통해 타입 체킹을 수행한다. 이는 타입 어노테이션이 없어도 코드 타입 체크를 하며, 코드베이스에 타입을 작성하지 않고도 타입 체킹을 수행할 수 있다.
