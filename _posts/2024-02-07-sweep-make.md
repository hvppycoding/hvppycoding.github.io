---
title: "`make` variable sweep"
excerpt: ""
date: 2024-02-07 08:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Tools
mathjax: "true"
---

## Objective

Makefile에서 변수를 선언하고 이를 이용하는 방법을 알아보자.

### Makefile 예시

다음과 같은 `Makefile`을 만들자.  
이 `Makefile`은 `NAME`과 `AGE`라는 두 개의 환경변수를 사용한다.  
만약 `NAME`이 없다면 `NONAME`으로, `AGE`가 없다면 `30`으로 설정한다.

```makefile
# Set the "NAME" environment to "NONAME" if not exist
export NAME ?= NONAME
# Set the "AGE" environment to "30" if not exist
export AGE ?= 30

all:
	@echo "My name is ${NAME}."
	@echo "I'm ${AGE} years old."
```

### 실행 결과

`make`를 실행하면 다음과 같은 결과를 얻을 수 있다.

```text
My name is NONAME.
I'm 30 years old.
```

## 환경 변수 설정

첫 번째로 `export`를 이용하여 환경 변수를 설정할 수 있다.  

```sh
export NAME=John
export AGE=25
```

shell에서 위와 같이 환경 변수를 설정하고 `make`를 실행하면 다음과 같은 결과를 얻을 수 있다.  

```text
My name is John.
I'm 25-years old.
```

다시 `unset`을 이용하여 환경 변수를 제거하자.

```sh
unset NAME
unset AGE
```

## `make` 실행 시 환경 변수 설정

`make` 실행 시 환경 변수를 설정할 수 있다.

```sh
make NAME=Jane AGE=22
```

위와 같이 `make`를 실행하면 다음과 같은 결과를 얻을 수 있다.

```text
My name is Jane.
I'm 22-years old.
```

## Variable Sweep

여러 조건에 대해 sweep하고 싶을 경우가 있다.
그런 경우 아래와 같은 `run` 스크립트를 만들 수 있다.

```sh
make NAME=Jack AGE=64
make NAME=Tom AGE=33
make NAME=Eva AGE=28
```

스크립트에 실행 권한을 주고 실행하자.

```sh
chmod +x run
./run
```

다음과 같은 결과를 얻을 수 있다.

```text
My name is Jack.
I'm 64-years old.
My name is Tom.
I'm 33-years old.
My name is Eva.
I'm 28-years old.
```
