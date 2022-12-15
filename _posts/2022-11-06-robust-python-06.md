---
title: "Robust Python 06 - 견고한 파이썬"
excerpt: "Robust Python 06 - 견고한 파이썬"
date: 2022-11-06 00:30:00 +0900
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

## 6장. 타입 체커의 커스터마이징

[p.145] 훌륭한 코딩 기술은 오래갈 수 있지만 여러분을 한 단계 높은 수준으로 올려주는 것은 사용하는 도구들이다. 코드 편집기, 컴파일러, 운영체제에 대한 공부를 게을리하면 안 된다.

### 타입 체커

mypy는 타입 체킹 시 엄격함의 정도를 조정할 수 있는 옵션을 제공한다. 코드 체크 엄격도와 개발 효율성 사이에서 적절한 선택이 필요하다.

3가지 방법으로 mypy 설정을 조정할 수 있다.

- 커맨드라인 옵션
- 인라인 설정: 대상 코드 파일 상단에 정의할 수 있다. 예를 들어 `# mypy: disallow-any-generics`를 작성하여 `Any`로 설정된 제네릭을 오류로 설정할 수 있다.
- 설정 파일: mypy 실행 시 동일 옵션을 반복적으로 사용하기 어려울 경우 설정 파일을 사용할 수 있다.

### mypy 설정

mypy를 실행하면 현재 디렉터리에서 mypy.ini 설정 파일을 찾는다. 이 파일은 프로젝트에서 지켜야 할 옵션들을 정의하고 있다. 글로벌 옵션/모듈별 옵션을 따로 지정할 수도 있다.

- `--disallow-any-expr`: `x: Any = 1` 오류 처리
- `--disallow-any-generics`: `x: list = [1,2,3,4]` 오류 처리
- `--disallow-untyped-defs`: 타입 어노테이션되지 않은 함수를 오류 처리
- `--disallow-incomplete-defs`: 타입 어노테이션을 일부만 적용한 함수를 오류 처리
- `--disallow-untyped-calls`: 어노테이션된 함수에서 어노테이션 되지 않은 함수를 호출할 때 오류 처리
- `--strict-optional`: 명시적으로 `is None`을 수행해야 한다. (mypy 버전에 따라 default로 옵션 켜져 있음)

mypy는 `None` 값을 `Optional`로 암시적으로 취급한다. 예를 들어 아래 코드에서 `x`는 묵시적으로 `Optional[int]`로 변환된다.  
이러한 암시적 변환을 끄려면 `--no-implicit-optional`을 설정할 수도 있다.

```python
def foo(x: int = None) -> None:
    print(x)
```

### mypy 리포트

- `--html-report`: 얼마나 많은 코드라인이 mypy에 의해 체크되었는지 보여주는 html 파일 생성
- `--any-exprs-report`: 얼마나 많은 `Any`가 사용되었는지 보여주는 리포트 파일 생성

### mypy를 빠르게

mypy는 대규모 코드베이스 체크 시 시간이 오래걸린다. mypy는 이전의 타입 체크 결과를 캐시로 저장(.mypy_cache)하여 변경된 부분만 체크한다. 하지만 코드베이스가 커지면 속도가 떨어지는 것은 어쩔 수 없다. 원격 캐시(버전 관리 시스템에서 커밋 ID별 체커 정보 관리)나 mypy 데몬 모드(`dmypy`)를 고려해보자.


### 기타 타입 체커

- Pyre: 페이스북 제작, mypy 데몬 모드와 유사하게 동작한다. 쿼리를 사용하여 코드베이스를 검사할 수 있다. Pysa라는 정적 코드 보안 분석기가 내장되어 있다.
- Pyright: 마이크로소프트 제작, 유연한 타입 체커이며, VS Code에 플러그인이 가능하다. Pyright 기반으로 동작하는 Pylance는 자동 완성/레퍼런스 검색/호출 그래프 탐색 등의 기능을 제공한다.