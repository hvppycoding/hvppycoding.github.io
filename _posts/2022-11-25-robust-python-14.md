---
title: "Robust Python 14 - 견고한 파이썬"
excerpt: "Robust Python 14 - 견고한 파이썬"
date: 2022-11-25 01:00:00 +0900
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

## 14장. pydantic을 이용한 런타임 체크

[p.319] 타입 작성 및 어노테이션은 shifting error left를 위한 행위들이며 오류를 테스트 단계 대신 더 이른 단계, 이상적으로는 개발 단계에서 찾을 수 있게 만드는 효과가 있다.

하지만 모든 오류가 코드 inspection이나 정적 분석으로 쉽게 찾아지지는 않는다. 외부 프로그램(데이터베이스, 네트워크 등)으로부터의 데이터로 작업할 때는 항상 잘못된 데이터가 전달될 위험이 있다. `if`문을 이용한 검증 로직을 작성할 수도 있지만, 복잡도가 증가해 가독성이 떨어지게 된다.  

## 동적 설정
