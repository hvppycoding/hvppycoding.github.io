---
title: "Code Review: Spyder IDE"
excerpt: "Code Review: Spyder IDE"
date: 2023-02-10 23:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Robust Python
---

## 1. 개요

Python IDE인 Spyder 코드를 분석해보자.  
소스코드는 [Spyder Github](https://github.com/spyder-ide/spyder)에서 확인할 수 있다.  
작성 시 코드는 [이 곳](https://github.com/hvppycoding/spyder)에 fork하였다.  

## 2. 환경 세팅

테스트 수행한 환경은 Windows 11, Python 3.9이며, [Contributing to Spyder](https://github.com/spyder-ide/spyder/blob/master/CONTRIBUTING.md) 문서를 참고하여 실행할 수 있다.

### Repository Clone하기

```sh
git clone https://github.com/hvppycoding/spyder.git
```

### 가상 환경 생성 및 의존 패키지 설치

```sh
python3.9 -m venv spyder-env
./spyder-env/Scripts/activate
pip install -e .
```

### Spyder 실행하기

```sh
python bootstrap.py
```

![2023-02-11-spyder.png]({{site.baseurl}}/assets/images/2023-02-11-spyder.png)
