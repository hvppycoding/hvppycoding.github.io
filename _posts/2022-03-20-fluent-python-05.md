---
title: "Fluent Python 05 - 일급 함수"
excerpt: "Fluent Python 05 - 일급 함수"
date: 2022-03-20 23:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Fluent Python
mathjax: "true"
---

**Notice:** 본 글은 책 [『전문가를 위한 파이썬<sup>Fluent Python</sup>』](https://books.google.co.kr/books?id=NJpIDwAAQBAJ&hl=ko&source=gbs_navlinks_s)을 학습하며 정리한 글입니다. 전체 소스 코드는 [Fluent Python github 레포지토리](https://github.com/fluentpython/example-code)에서 확인할 수 있습니다.
{: .notice--info}

# Chapter 5. 일급 함수

파이썬의 함수는 일급 객체다. 프로그래밍 언어 이론가들은 다음과 같은 작업을 수행할 수 있는 프로그램 개체를 '일급 객체'로 정의한다.

- 런타임에 생성할 수 있다.
- 데이터 구조체의 변수나 요소에 할당할 수 있다.
- 함수 인수로 전달할 수 있다.
- 함수 결과로 반환할 수 있다.

## 5.1 함수를 객체처럼 다루기

