---
title: "Robust Python 12 - 견고한 파이썬"
excerpt: "Robust Python 12 - 견고한 파이썬"
date: 2022-11-22 01:00:00 +0900
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


# 12장. 하위 타입

[p.279] 하위 타입을 올바르게 사용하면 코드베이스를 쉽게 확장할 수 있으며 코드베이스의 다른 부분들을 손상시킬 우려 없이 새로운 동작을 도입할 수 있다. 그러나 하위 타입을 만들 때는 신중해야 한다. 하위 타입의 관계를 잘 작성하지 못하면 예기지 못한 상황으로 인해 코드베이스의 견고성이 떨어질 수 있다.  

OOP에 관심이 있다면 『Head First Object-Oriented Analysis and Design』(O'Reilly, 2006)을 추천한다.
{: .notice--primary}


## 상속

