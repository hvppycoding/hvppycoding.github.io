---
title: "Robust Python 05 - 견고한 파이썬"
excerpt: "Robust Python 05 - 견고한 파이썬"
date: 2022-11-05 17:30:00 +0900
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

# 5장. 컬렉션 타입

`count_house()`는 학생 리스트로부터 각 기숙사별 학생 수를 반환해준다.

```python
# collection.py
from collections import defaultdict


class Student:
    def __init__(self, name: str, house: str):
        self.name:str = name
        self.house:str = house

def count_house(students: list) -> dict:
    counter: dict = defaultdict(lambda: 0)
    for student in students:
        counter[student.house] += 1
    return counter


if __name__ == '__main__':
    students = [
        Student('Harry Potter', 'Gryffindor'),
        Student('Ronald Weasley', 'Gryffindor'),
        Student('Hermione Granger', 'Gryffindor'),
        Student('Draco Malfoy', 'Slytherin'),
        Student('Gregory Goyle', 'Slytherin'),
        Student('Ernie Macmillan', 'Hufflepuff'),
    ]

    result = count_house(students=students)
    print(result)
```

하지만 어떤 컬렉션을 입력으로 받는지는 알 수 있지만 컬렉션 안에 어떤 요소들이 있는 지 정보는 없다. 이를 위해 컬렉션 내 타입을 지정하기 위해 대괄호 구문을 사용해 정보를 추가할 수 있다.


```python
# collection2.py
HouseToCountMapping = dict[str, int]
def count_house(students: list[Student]) -> HouseToCountMapping:
    counter: dict = defaultdict(lambda: 0)
    for student in students:
        counter[student.house] += 1
    return counter
```


### TypedDict

딕셔너리 내에 이종 데이터를 저장하는 경우 사용된다. 이런 경우는 보통 이종 데이터를 피할 수 없는 경우인데, JSON, YAML, TOML, XML, CSV 모두 해당 데이터 형식을 딕셔너리로 바꿔주는 파이썬 모듈들이 존재하며, 이는 이종 데이터가 된다. 타입 체커나 사용자는 어떤 키와 값이 가능한지 알 수 없다.   
예를 들어 도서 검색 앱에서 저자 정보 제공을 하고 싶다고 하자. API를 사용하여 저자 정보를 얻기 위한 코드를 다음과 같이 작성했다.
```python
writer_information = get_writer_information_api(book)
# print writer's name
print(writer_information["name"])
```
이 코드를 검토할 때 코드가 올바른지(name이라는 키가 존재하는지) 어떻게 확인할 것인가?
두 가지 옵션이 존재한다.
1. API 문서를 보고 필드를 확인한다. - 문서가 정확해야 한다.
2. 코드를 돌려보고 딕셔너리를 출력해본다. - 테스트와 실제 결과가 일치해야 한다.

이는 코드의 견고성을 하락시킨다. `TypedDict`는 API에 대한 정보들을 타입 시스템에 반영할 수 있다.

```python
from typing import TypedDict

class InstitutionInformation(TypedDict):
    name: str

class WriterInformation(TypedDict):
    name: str
    age: int
    institutions: list[Institution]
    books: list[str]

writer_information: WriterInformation = get_writer_information_api(book)
```

이제 타입 체커는 딕셔너리 구조를 체킹해줄 수 있고, 코들르 읽는 사람들은 외부 검색 없이 결과를 추론할 수 있다.
(`TypedDict`는 타입 체커를 위해서만 존재한다. 코드 실행에는 아무 영향이 없으며 실행할 때에는 그냥 딕셔너리로 취급된다.)

### 제네릭

제네릭 타입<sup>generic type</sup>은 사용자가 무슨 타입을 쓰든지 상관하지 않겠다는 것을 가리킨다.
리스트 순서를 거꾸로 만드는 함수를 생각해보자.

```python
def reverse(coll: list) -> list:
    return coll[::-1]
```

반환되는 리스트가 입력받은 리스트와 동일한 타입을 담고 있다는 거승ㄹ 어떻게 표현할 수 있을까?
이를 위해서 파이썬에서 제공하는 `TypeVar`라는 제네릭을 사용한다.

```python
from typing import TypeVar

T = TypeVar('T')

def reverse(coll: list[T]) -> list[T]:
    return coll[::-1]
```

이런 패턴을 사용해 클래스를 정의할 수도 있다. 아래 예시는 제네릭 타입에 대해 사용할 수 있는 `Graph`를 정의한 후, `Author`과 `Book` 타입에 활용한 예시이다.

```python
from collections import defaultdict
from typing import Generic, TypeVar


Node = TypeVar("Node")
Edge = TypeVar("Edge")

class Book:
    def __init__(self, title):
        self.title = title

class Author:
    def __init__(self, name):
        self.name = name

class Graph(Generic[Node, Edge]):
    def __init__(self):
        self.edges: dict[Node, list[Edge]] = defaultdict(list)

    def add_relation(self, node: Node, to: Edge):
        self.edges[node].append(to)
    
    def get_relations(self, node: Node) -> list[Edge]:
        return self.edges[node]


author_graph: Graph[Author, Book] = Graph()

books: list[Book] = [
    Book('Harry Potter 1'),
    Book('Harry Potter 2'),
    Book('The Lord of the Rings'),
]

author: list[Author] = [
    Author('J. K. Rowling'),
    Author('J. R. R. Tolkien'),
]

author_graph.add_relation(author[0], books[0])
author_graph.add_relation(author[0], books[1])
author_graph.add_relation(author[1], books[2])

# author_graph.add_relation(books[1], books[2])
# >> generic.py:44: error: Argument 1 to "add_relation" of "Graph" has incompatible type "Book"; expected "Author"
# Found 1 error in 1 file (checked 1 source file)
```

### 기존 타입의 변경

기존 딕셔너리나 리스트와 같은 컬렉션 타입의 동작을 조금 변경하고 싶다면 어떻게 해야할까? 컬렉션의 모든 기능을 재작성하는 것은 어렵다. 일반적으로는 `dict`를 상속받아 구현하려고 할 것이다. 

```python
class CaseInsensitiveDict(dict):
    def __getitem__(self, k):
        if isinstance(k, str):
            return super().__getitem__(k.lower())
        else:
            return super().__getitem__(k)

    def __setitem__(self, k, v):
        if isinstance(k, str):
            super().__setitem__(k.lower(), v)
        else:
            super().__setitem__(k, v)

d = CaseInsensitiveDict()
d["Harry"] = "Gryffindor"

print(d["HARRY"])
# > Gryffindor
print(d.get("HARRY", "Not Found!"))
# > Not Found!
```

위 코드에서부터 `__getitem__()` 메서드를 오버라이드했지만 `get()` 함수에는 적용이 되지 않은 것을 알 수 있다. 내장 컬렉션 타입들은 성능을 우선하여 만들어져있으며 많은 메서드가 속도를 위해 인라인 코드를 사용하기 때문이다. 이 경우에는 `collections.UserDict`라는 쓰기 쉬운 타입이 있다.

```python
from collections import UserDict

class CaseInsensitiveDict(UserDict):
    def __getitem__(self, k):
        if isinstance(k, str):
            return self.data[k.lower()]
        else:
            return self.data[k]

    def __setitem__(self, k, v):
        if isinstance(k, str):
            self.data[k.lower()] = v
        else:
            self.data[k] = v

d = CaseInsensitiveDict()
d["Harry"] = "Gryffindor"
print(d["HARRY"])
# > Gryffindor
print(d.get("HARRY", "Not Found!"))
# > Gryffindor
```

이제는 `Not Found!`가 발생하지 않는다. `UserDict`와 유사하게 `UserList`, `UserString`도 존재한다. `UserSet`은 존재하지 않는데, 세트는 `collection.abc`가 필요하다.

### collections.abc

`collections.abc` 모듈의 추상화 기반 클래스<sup>ABC, Abstract Based Classes</sup>는 사용자 정의 컬렉션을 생성하기 위한 기반 클래스들을 제공한다. `collections.abc`를 사용하여 사용자 정의 컬렉션을 만들려면 타입에 따라 특정 추상 메서드들을 구현해주어야 한다. 이러한 필수 메서드들을 구현하면 abc는 다른 함수들은 자동으로 완성한다. `collections.abc`의 [문서](https://docs.python.org/3/library/collections.abc.html)에서 구현해야하는 메서드 목록을 확인할 수 있다. `collections.abc.Set`의 경우 `__contains__`, `__iter__`, `__len__`을 구현해야 한다.

```python
import collections.abc
from typing import Iterator

class CaseInsensitiveSet(collections.abc.Set):
    def __init__(self, str_set: set[str]):
        self._data = set()
        for item in str_set:
            self._data.add(item.lower())

    def __contains__(self, k: str) -> bool:
        return k.lower() in self._data
    
    def __iter__(self) -> Iterator[str]:
        return iter(self._data)

    def __len__(self) -> int:
        return len(self._data)


cis = CaseInsensitiveSet({'Harry', 'Ron', 'Hermione'})
for k in cis:
    print(k)
# hermione
# harry
# ron

print('HARRY' in cis)
# True

print(list(cis | CaseInsensitiveSet({'Albus'})))
# ['harry', 'ron', 'albus', 'hermione']
```