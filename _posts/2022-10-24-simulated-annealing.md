---
title: "Simulated Annealing"
excerpt: "Simulated Annealing"
date: 2022-10-24 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

# Simulated Annealing

(Wikipedia에서 발췌) Simulated Annealing은 주어진 함수 대한 전역 최적해를 근사하는 확률적 테크닉이다. 특히, search space가 클 때 global optimization을 근사하는 메타휴리스틱이다. search space가 discrete일 때(예를 들어 TSP 문제, 3SAT 문제, Protein structure prediction 문제, Job-shop scheduling 문제) 주로 사용된다. 정해진 시간 내에 정확한 로컬 최적해보다 근사 전역 최적지점을 찾는 것이 더 중요할 때, Simulated Annealing은 선호된다.

이 알고리즘의 이름은 금속 공학의 어닐링(풀림)에서 유래되었다. 이 기법은 물질의 물리적인 특성을 바꾸기 위해 가열/냉각하는 것을 포함한다. 가열/냉각은 모두 물질의 속성으로 열역학적 자유 에너지에 의존한다. 물질 가열/냉각은 온도와 열역학적 자유 에너지나 깁스 에너지에 영향을 미친다. Simulated Annealing은 Computational optimization problem에 쓰일 수 있다(Exact Algorithm은 실패하는). 전역 최적값에 대한 근사 솔루션이지만 이는 많은 실무적인 문제들에 대해서 충분할 수 있다.

SA로 풀리는 문제들은 많은 변수를 가진 objective function으로 formulated 된다. 실제 적용 시 constraint는 주로 objective function에서 penalty를 주는 것으로 적용된다.

Kirkpatrick, Gelatt Jr., Vecchi,에 의해 1983년에 Traveling Salesman Problem에 적용한 것으로 소개되었다. 그들은 현재 이름(Simulated Annealing)을 제안하였다. Slow cooling 구현은 Worse solution을 받아들일 확률을 서서히 줄이는 것으로 해석되었다. Worse Solution을 받아들이는 것은 더 Global optimal solution을 찾기 위한 광범위한 탐색을 가능케 한다. 일반적으로 SA는 다음과 같이 동작한다. Temperature은 점차 양수의 초기값에서 0으로 감소한다. 각 타임스텝에서 알고리즘은 랜덤하게 현재 상태에 가까운 하나의 Solution을 랜덤하게 선택한 후, 해당 Solution의 퀄리티를 측정하고, temperature와 연관된 확률에 따라서 더 좋은/더 나쁜 솔루션으로 옮겨간다. 더 좋은 솔루션을 선택할 확률은 1(혹은 양수)를 유지하고, 더 나쁜 솔루션을 선택할 확류은 0으로 감소해간다.

# Overview
어떤 물리적인 시스템의 `state`와 최소화할 function $E(s)$  
어떤 임의의 initial state에서 minimum possible energy를 가지는 상태로 옮기는 것이 목표이다.

## The basic iteration
각 스텝에서 Simulated Annealing heuristic은 현재 상태 $s$에 대한 이웃 상태 $s^\*$를 고려한다. 그리고 확률적으로 시스템 상태를 $s^\*$로 옮길지 $s$에 머물지 결정한다. 궁극적으로 더 낮은 에너지 상태로 이끌어줄 확률이다. 주로 이 스텝이 어플리케이션에 충분히 좋은 솔루션을 발견할때까지 반복하거나, 계산 budget에 다다를때까지 반복한다.

## The neighbours of a state
Solution optimization에는 어떤 상태의 이웃 상태를 평가하는 것을 포함한다. 새 상태는 주어진 상태를 변경하여 생성된다. 예를 들어 Travelling Salesman Problem에서는 주로 각 상태는 방문해야할 도시의 순열로 정의된다. 그리고 각 상태의 이웃은 해당 상태에서 두 개의 도시를 서로 스왑하여 만들 수 있는 상태들로 설정할 수 있다.