---
title: "Python: Coding Guidelines"
excerpt: "Tooling, Testing and Packaging"
date: 2022-12-09 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Robust Python
---

## 1. 개요

- 코딩 가이드 라인(PEP8)
- 타입 어노테이션과 mypy
- 파이썬 패키지 생성
- continuous integration 툴
- Pylint, Flake8, Autopep8, Black 등 툴
- 디버깅, 프로파일링, 타이밍, 유닛 테스팅 사용법
- 모던 HTML 문서 생성

Requirements

- 기본적인 OS와 터미널 사용법
- 파이썬 기초 지식
- python 3.10 version
- vscode()

## 2. 코딩 가이드라인과 Docstring

### 참고 사이트들

- Overall PEP8 Guide
  - https://www.python.org/dev/peps/pep-0008/
- Important sub-chapters
  - Imports: https://www.python.org/dev/peps/pep-0008/#imports  
  - Indentation: https://www.python.org/dev/peps/pep-0008/#indentation  
  - Comments: https://www.python.org/dev/peps/pep-0008/#comments  
  - Naming Convention: https://www.python.org/dev/peps/pep-0008/#naming-conventions  
  - Programming Recommendations: https://www.python.org/dev/peps/pep-0008/#id51  
- Docstrings
  - VS Code Extension: Autodocstring
  - PEP 257: https://www.python.org/dev/peps/pep-0257/  
  - Google: https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html  
  - NumPy: https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_numpy.html#example-numpy  
  - ReST: https://sphinx-rtd-tutorial.readthedocs.io/en/latest/docstrings.html  
  - Type Annotations: https://sphinxcontrib-napoleon.readthedocs.io/en/latest/index.html#type-annotations  

### PEP Guidelines

```python
# 첫 블록은 standard library import
import sys
import os
import typing

# 두 번째 블록은 써드파티 패키지
import pylint 

# 세 번째 블록은 현재 패키지(First party)
from .my_lib import print_hello

# 파라미터의 수가 3개 이상이면 라인을 나누는 것을 추천한다.
def my_function_with_many_params(
    param1: int,
    param2,
    param3,
    param4,
    param5: float,
):
    return "Hello World!"   # 문자열 시 "나 ' 중 하나를 선택해서 일관적으로 사용하라.

def main() -> None:
    user1 = "Jan"
    user2 = "Max"
    user3 = "Marcus"
    
    # 리스트도 수가 많아진다면 라인을 나누자.
    my_list = [
      user1, 
      user2, 
      user3
    ]

    print_hello()
```

### Code Linter: Pylint

- `pylint 파일명.py` 명령어를 통해 기본적인 코드 검사를 할 수 있다.
- `pylint 폴더명` 명령어를 통해 폴더 내의 코드들을 검사할 수 있다.
- `rcfile` 생성하여 Lint에서 체크할 규칙을 설정할 수 있다.
- python 파일 상단에 `# pylint: disable=invalid-name`과 같은 주석을 통해 파일별 관리도 가능하다.
- [pylint 사용법](https://pylint.pycqa.org/en/latest/user_guide/usage/run.html)

### Code Linter: Flake8

- `flake8 파일명.py` 명령어를 통해 flake8로 코드 검사할 수 있다.
- pylint보다 속도가 빠르다.
- pylint와 비슷하게 config 파일로 설정이 가능하다.

### Code Linter: isort

- import 라이브러리를 Future, Standard, Thirdparty, Firstparty 순으로 자동 정렬해준다.
- `isort 파일명.py`를 통해 파일 자동 수정할 수 있다.
- 만약 파일을 수정하지 않고 dry run만 하기 위해서는 `isort --diff 파일명.py` 사용하자.
- pylint와 비슷하게 config 파일로 설정이 가능하다.

### Code Linter: autopep8

- PEP8 코딩 스타일로 자동 포매팅해준다.
- `autopep8 파일명.py`로 수정후 결과를 볼 수 있다.
- `autopep8 --diff 파일명.py`로 변경되는 지점을 확인할 수 있다.
- 파일을 바로 수정하기 위해서는 `autopep8 --in-place 파일명.py` 명령어를 사용해야 한다.

### Code Linter: black

- 많이 사용되고 있는 포매팅 툴이다. Aggressive한 편이며, config 요소가 적다.
- `python -m black {source_file_or_directory}`
- `--diff` 옵션을 사용하면 변경점 미리 확인할 수 있다.

### Docstring: Numpy Style

- docstring style은 vscode 설정에서 원하는 스타일을 설정할 수 있다.
- docstring 시 '와 " 중 어떤 문자를 사용할 것인지도 설정 가능하다.

```python
class Vector2D:
    def __init__(self, x=0.0, y=0.0):
        '''Create a vector2d object.

        Parameters
        ----------
        x : float, optional
            The x-coordinate of the 2d vector, by default 0.0
        y : float, optional
            The y-coordinate of the 2d vector, by default 0.0

        Raises
        ------
        TypeError
            You must pass in int/float values for x and y!
        '''
        if isinstance(x, float) and isinstance(y, float):
            self.x = x
            self.y = y
        else:
            raise TypeError('You must pass in int/float values for x and y!')
```

## 디버깅, 유닛테스팅, 타이밍

### Unit-Testing with unittest

- `unittest`는 기본 표준 라이브러리이다.
- 다음은 유닛테스트 예시이다. 테스트 파일 컨벤션은 보통 `tests` 디렉터리 안에 `test_테스트대상.py` 파일을 사용한다.

```python
import unittest

from fastvector import Vector2D


class VectorTests(unittest.TestCase):
    def setUp(self) -> None:  # setUp 메서드는 Entrance point이다.
        self.v1 = Vector2D(0, 0)
        self.v2 = Vector2D(-1, 1)
        self.v3 = Vector2D(2.5, -2.5)

    def test_equality(self) -> None:  # test_* 는 테스트 메서드 
        self.assertNotEqual(self.v1, self.v2)
        expected_result = Vector2D(-1, 1)
        self.assertEqual(self.v2, expected_result)

    def test_add(self) -> None:
        result = self.v1 + self.v2
        expected_result = Vector2D(-1, 1)
        self.assertEqual(result, expected_result)

    def test_sub(self) -> None:
        result = self.v2 - self.v3
        expected_result = Vector2D(-3.5, 3.5)
        self.assertEqual(result, expected_result)

    def test_mul(self) -> None:
        result1 = self.v1 * 5
        expected_result1 = Vector2D(0.0, 0.0)
        self.assertEqual(result1, expected_result1)
        result2 = self.v1 * self.v2
        expected_result2 = 0.0
        self.assertEqual(result2, expected_result2)

    def test_div(self) -> None:
        result = self.v3 / 5
        expected_result = Vector2D(0.5, -0.5)
        self.assertEqual(result, expected_result)


if __name__ == "__main__":
    unittest.main()
```

- vscode 확장에서 테스트 기능을 제공하며, test를 탐색/수행할 수 있다.

![unittest example]({{site.baseurl}}/assets/images/2022-12-12-unittest.png)

### Unit-Testing with pytest

- `unittest` 라이브러리에서는 동일 메서드에 대해 여러 테스트케이스를 실험하기 위해서는 `test_*` 함수를 여러 번 작성해야 한다.
- `pytest`에서는 이를 간단하게 할 수 있으며 또한 여러 유용한 feature를 제공하기 때문에 `pytest`를 사용하는 것을 추천한다.

```python
from typing import Any

import pytest
from fastvector import Vector2D


V1 = Vector2D(0, 0)
V2 = Vector2D(-1, 1)
V3 = Vector2D(2.5, -2.5)


@pytest.mark.parametrize(
    ('lhs', 'rhs', 'exp_res'),
    (
        (V1, V2, Vector2D(-1, 1)),
        (V1, V3, Vector2D(2.5, -2.5)),
        (V3, V2, Vector2D(1.5, -1.5)),
        (V3, V1, Vector2D(2.5, -2.5)),
        pytest.param(V3, V1, Vector2D(2.0, -2.0), marks=pytest.mark.xfail) # 실패해야하는 코드
    )
)
def test_add(lhs: Vector2D, rhs: Vector2D, exp_res: Vector2D) -> None:
    assert lhs + rhs == exp_res

@pytest.mark.skip(reason="Not implemented") # skip하고 싶을 때 사용한다.
def test_abs() -> None:
    pass
    
@pytest.mark.parametrize(
    ('x', 'y'),
    (
        (-1, None),
        (None, -1),
        (None, None),
    )
)
def test_raises(x: Any, y: Any) -> None:
    with pytest.raises(TypeError):  # Error가 raise되는 것을 체크한다.
        _ = Vector2D(x, y)
```

## Packaging

### Modules and Packages - 1

상대경로에 위치한 package에 접근 가능하다.

```bash
📂my_package         # python package
 └─📝__init__.py     # python package의 main
📝main.py            # from my_package import <my_package/__init__.py 내 정의된 변수/함수> 가능
```

### Modules and Packages - 2

```bash
📂my_package         # python package
 ├─📝__init__.py     # python package의 main
 └─📂printing        # sub-package
    └─📝__init__.py  # sub-package의 main
📝main.py            # from my_package.printing import <my_package/printing/__init__.py 내 정의된 변수/함수> 가능
```

### Modules and Packages - 3

```python
'''
📂my_package
 ├─📝__init__.py
 ├─📂printing
 │  ├─📝__init__.py    # from ._printing import print_hello_world (_printing에 선언된 함수를 printing package main에 포함)
 │  └─📝_printing.py   # _로 시작하는 module은 private함을 나타냄
 └─📂version
    └─📝__init__.py    # __version__ 선언됨
📝main.py
'''

# main.py
from my_package.printing import print_hello_world
from my_package.version import __version__
# 이렇게도 쓸 수는 있다.(프로그래밍적으로 private한 것은 아님)
# from my_package.printing._printing import print_hello_world 


def main() -> None:
    print_hello_world()
    print(__version__)
```

### Modern Python Packages

- [템플릿 예시](https://github.com/franneck94/Python-Project-Template-Eng)

#### Project layout

```bash
📂fastvector
 ├─📝README.md  # 패키지 설명 문서 
 ├─📝.editorconfig  # Editor 규칙 설정
 ├─📝.gitattributes  # git commit 시 설정
 ├─📝.gitignore  # git에서 무시할 파일 설정
 ├─📝.pre-commit-config.yaml
 ├─📝pyproject.toml
 ├─📝requirements.txt  # 필요 라이브러리
 ├─📝setup.py  # 
 ├─📝setup.cfg  # 라이브러리, version 등 설정
 ├─📂tests
 │  ├─📝__init__.py
 │  └─📝test_vector.py
 ├─📂docs
 │  ├─📝api.md
 │  └─📝index.md
 ├─📂fastvector
 │  ├─📝__init__.py
 │  ├─📝vector.py
 │  └─📝version.py
 └─📂tests
    ├─📝__init__.py
    └─📝test_vector.py
```

자세한 `setup.py`와 `setup.cfg` 관련 자세한 내용은 `setuptools`의 [user guide](https://setuptools.pypa.io/en/latest/userguide/index.html)를 참고할 수 있다.

다음은 `setup.cfg` 파일의 예시이다.

```conf
[metadata]
name = my_package
version = attr: my_package.VERSION
description = My package description
long_description = file: README.rst, CHANGELOG.rst, LICENSE.rst
keywords = one, two
license = BSD 3-Clause License
classifiers =
    Framework :: Django
    Programming Language :: Python :: 3

[options]
zip_safe = False
include_package_data = True
packages = find:
install_requires =
    requests
    importlib-metadata; python_version<"3.8"

[options.package_data]
* = *.txt, *.rst
hello = *.msg

[options.entry_points]
console_scripts =
    executable-name = my_package.module:function

[options.extras_require]
pdf = ReportLab>=1.2; RXP
rest = docutils>=0.3; pack ==1.1, ==1.3

[options.packages.find]
exclude =
    examples*
    tools*
    docs*
    my_package.tests*
```

### Mkdocs and Github Pages

Mkdocs를 통해 Jekyll 페이지 생성할 수 있고, github에 publish할 수 있다.
