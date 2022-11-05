---
title: "Robust Python 04 - 견고한 파이썬"
excerpt: "Robust Python 04 - 견고한 파이썬"
date: 2022-11-04 11:30:00 +0900
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

# 4장. 타입의 제어

고급 어노테이션 소개

### Optional 타입

```python
# optional.py
class Student:
    def __init__(self, name : str, student_id : int):
        self.name:str = name
        self.student_id:int = student_id
        self.professor:str = ''

    def set_professor(self, professor: str):
        self.professor = professor

students: list[Student] = [
    Student('Harry', 0),
    Student('Jack', 3),
    Student('Edward', 12),
]

def find_student(student_id: int) -> Student:
    for student in students:
        if student.student_id == student_id:
            return student
    return None

def assign_professor(student_id: int, professor_name: str):
    student = find_student(student_id)
    student.set_professor(professor_name)

if __name__ == '__main__':
    assign_professor(12, 'steven')
```

`find_student` 함수에서 해당 id에 일치하는 학생 정보가 없을 경우, `None`을 리턴한다. 이 경우 `None` 타입에 대해 `student.set_professor()` 함수를 호출하게 되므로 오류를 발생시킨다. 이에 대해 타입 체커를 수행해보면 다음과 같은 오류가 발생한다.

```
$ mypy optional.py
> optional.py:20: error: Incompatible return value type (got "None", expected "Student")
  Found 1 error in 1 file (checked 1 source file)
```

`find_student` 함수의 반환 타입을 아래와 같이 변경하자. 이는 함수의 리턴값이 `None`이 될 수도 있음을 나타낸다.
```python
def find_student(student_id: int) -> Optional[Student]
```

다시 타입 체커를 수행해보면, 다음과 같은 오류가 발생한다.

```
$ mypy optional2.py
> optional2.py:28: error: Item "None" of "Optional[Student]" has no attribute "set_professor"
  Found 1 error in 1 file (checked 1 source file)
```

이는 반환 타입이 `None`이 될 수 있으며, `None`에는 `set_professor`라는 속성이 존재하지 않음을 알려준다. 이 오류를 없애기 위해서는 명시적으로 `None` 타입에 대해 체크하는 것이 필요하다. 최종 코드는 다음과 같다.

```python
# optional2.py
from typing import Optional


class Student:
    def __init__(self, name : str, student_id : int):
        self.name:str = name
        self.student_id:int = student_id
        self.professor:str = ''

    def set_professor(self, professor: str):
        self.professor = professor

students: list[Student] = [
    Student('Harry', 0),
    Student('Jack', 3),
    Student('Edward', 12),
]

def find_student(student_id: int) -> Optional[Student]:
    for student in students:
        if student.student_id == student_id:
            return student
    return None

def assign_professor(student_id: int, professor_name: str):
    student = find_student(student_id)
    if student is None:
        print(f'Invalid student id: {student_id}')
        return
    student.set_professor(professor_name)

if __name__ == '__main__':
    assign_professor(12, 'steven')
```


### Union 타입

`Union` 타입은 변수가 다양한 타입이 될 수 있음을 의미한다. 예를 들어 `Union[int, str]`은 `int`와 `str`이 모두 올 수 있음을 뜻한다.
다음 `write_hello` 함수는 파일 경로를 입력으로 받아, 해당 경로에 'hello'라는 텍스트 파일을 저장하는 간단한 함수이다.

```python
#union.py
def write_hello(path):
    with open(path, 'w') as f:
        f.write('hello')


if __name__ == '__main__':
    # hello.txt 파일에 hello가 써진다.
    write_hello('hello.txt')
```

여기에서 파일 경로 대신에 텍스트 I/O도 사용할 수 있도록 추가하면 편리할 것이다.

```python
#union2.py
from io import TextIOWrapper


def write_hello(path_or_io):
    if isinstance(path_or_io, str):
        with open(path_or_io, 'w') as f:
            f.write('hello')
    elif isinstance(path_or_io, TextIOWrapper):
        path_or_io.write('hello')
    else:
        print('path_or_io is neither str nor TextIOWrapper')

if __name__ == '__main__':
    with open('other_path.txt', 'w') as f:
        # other_path.txt 파일에 hello가 써진다.
        write_hello(f)
```

이런 경우 `Union` 타입 어노테이션을 사용할 수 있다.

```python
#union2.py
from typing import Union
from io import TextIOWrapper


def write_hello(path_or_io: Union[str, TextIOWrapper]):
    if isinstance(path_or_io, str):
        with open(path_or_io, 'w') as f:
            f.write('hello')
    elif isinstance(path_or_io, TextIOWrapper):
        path_or_io.write('hello')
    else:
        print('path_or_io is neither str nor TextIOWrapper')

if __name__ == '__main__':
    write_hello('hello.txt')
    with open('other_path.txt', 'w') as f:
        write_hello(f)
```

[p.104] 특히 다음과 같은 경우에 `Union`의 진가가 발휘된다.
- 사용자 입력에 따라 서로 다른 타입의 반환값을 리턴할 때
- 다른 형태의 사용자 입력을 처리할 때(예를 들어 사용자가 문자열 또는 리스트를 입력할 수 있을 때)
- 이전 버전과의 호환성을 위해 다른 타입 반환할 때(요청에 따라 예전 버전의 객체나 새로운 버전의 객체 중 선택 반환)


### Literal 타입

`Literal` 타입은 변수를 일정 값들로 제한시켜주는 역할을 한다.

```python
#literal.py
from typing import Literal
from dataclasses import dataclass


@dataclass
class Card:
    suit: Literal['Spade', 'Club', 'Diamond', 'Heart']
    rank: Literal['A', '2', '3', '4', '5', '6', '7', 
                  '8', '9', '10', 'J', 'Q', 'K']


if __name__ == '__main__':
    c1 = Card('Spade', 'A')
    c2 = Card('Joker', 'A')
```

```
literal.py:13: error: Argument 1 to "Card" has incompatible type "Literal['Joker']"; 
expected "Literal['Spade', 'Club', 'Diamond', 'Heart']"
Found 1 error in 1 file (checked 1 source file)
```

### Annotated 타입
리터럴을 일일이 선언하는 것이 어려울 경우(예를 들어 1~100 사이 숫자) Annotation을 활용 가능, 현재는 체크 도구가 존재하지 않으며 주석 역할로 사용할 수 있다.

### NewType
예를 들어 타입 객체 중 특정 조건을 만족한 객체만 사용하고 싶을 경우 NewType 사용할 수 있다.


### Final 타입

값을 바꿀 수 없는 타입이다.