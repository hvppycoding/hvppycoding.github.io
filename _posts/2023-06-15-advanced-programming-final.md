---
title: "시험 대비용"
excerpt: "Advanced Programming Methodologies 12"
date: 2023-06-15 02:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Tips

- Loop Invariant: 반복문이 실행되는 동안 항상 참인 조건
  - Initialization, Maintenance, Termination을 보여야 함
  - Termination 시 i값에 주의할 것 `i = 1 to n`이면 Termination 시 `i = n + 1`임
- Induction: Inductive Hypothesis 가설 설정 후
  - Base Case, Inductive Step이 만족함을 보여야 함
- **배열 초기화** 꼼꼼히 하자
- 증명해야할 것 같으면 증명도 쓰자
- 동치임을 증명해야하면 양방향으로 증명이 필요하다

## NP-Completeness

- 보여야할 것
  1. NP에 속한다: 후보 답안이 주어졌을 때 그것을 확인하는데 polynomial time이 걸린다
  2. NP-Hard다.

## Reduction

- 문제 B를 잘 풀어주는 Black-Box가 있다고 하자.
- 문제 A를 문제 B로 Polynomial Time Reduction이 가능하고 문제 A를 문제 B의 결과를 활용해서 풀 수 있다고 하자. 즉, 문제 B에 대한 Polynomial Solution이 있으면 문제 A도 Polynomial 시간 안에 풀 수 있게 된다.
- [문제 A] ─(Reduction, 변환)→ [문제 B]
- 문제 A의 사례 $\alpha$를 문제 B의 사례 $\beta$로 바꾸되 다음 성질을 만족하면 다항식 시간 변환(reduction)이라고 하고 $\alpha \le_p \beta$로 표기한다.
  1. 변환은 다항식 시간에 이루어진다.
  2. 두 사례의 답은 일치한다. $\alpha$가 Yes이면 $\beta$도 Yes이고, $\alpha$가 No이면 $\beta$도 No이다. 혹은 대우인 $\beta$가 Yes일 때 $\alpha$도 Yes임을 보여도 된다.

### NP-Hard

모든 NP 문제 L에 대하여 $L \le_p A$이면 문제 A는 NP-Hard이다.  

→ 모든 NP 문제에 대한 증명 어렵다. 알려진 하나의 NP-Hard 문제에 대해 Reduction을 통해 증명한다.

### NP-Complete

다음 두 가지 조건을 만족하면 NP-Complete이다.

1. NP다
2. NP-Hard다

