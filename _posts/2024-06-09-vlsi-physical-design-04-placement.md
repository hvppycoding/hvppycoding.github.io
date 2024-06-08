---
title: "Chapter 4. Global and Detailed Placement"
excerpt: "VLSI Physical Design"
date: 2024-06-09 06:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

![2024-06-09-placement-flow.svg]({{site.baseurl}}/assets/images/2024-06-09-placement-flow.svg){: .align-center}

### Introduction

![2024-06-09-placement-intro.svg]({{site.baseurl}}/assets/images/2024-06-09-placement-intro.svg){: .align-center}

현재 공정에서는 주로 2D Placement가 필요하다. Standard Cell들은 대부분 Height가 모두 동일하다.

![2024-06-09-placement-steps.svg]({{site.baseurl}}/assets/images/2024-06-09-placement-steps.svg){: .align-center}

### Optimization Objectives

![2024-06-09-placement-objs.svg]({{site.baseurl}}/assets/images/2024-06-09-placement-objs.svg){: .align-center}

Placement에서 고려할 사항들

1. Total wirelength
  - 모든 net의 길이의 합을 줄인다. (placement 단계에서 정확한 값은 알 수 없고, 예측값을 사용)
  - Critical net에 대해 가중치를 더 주는 것도 가능하다.
2. Number of Cut Nets
  - Global Placement에서 cell을 건너는 net의 수를 줄인다.
  - 최종적으로는 wirelength를 줄이는 것이 목표이다.
3. Wire Congestion
  - Metal line의 수가 제한되어 있으므로, 이를 고려해야 한다.
  - 결국 나중의 Routability를 고려하는 것이다.
4. Signal Delay
  - Critical path net의 길이는 짧은 것이 좋다.

#### Optimization Objectives - Total Wirelength

![2024-06-09-total-wirelength-obj.svg]({{site.baseurl}}/assets/images/2024-06-09-total-wirelength-obj.svg){: .align-center}

좌측의 Placement가 wirelength가 짧다.

Wirelength를 예측하는 것이 중요하다. 정확도도 중요하지만, Runtime도 중요하다.

![2024-06-09-total-wirelength-estimation1.svg]({{site.baseurl}}/assets/images/2024-06-09-total-wirelength-estimation1.svg){: .align-center}

- Half-perimeter wirelength(HPWL)
  - Net의 bounding box의 가로 세로 길이의 합.
  - 주로 사용되는 metric으로, 계산이 빠르고 합리적인 예측값을 제공한다.
- Complete graph(clique)
  - $p(p-1)/2$개 edge의 모든 manhattan distance를 더한다.
  - 결국 필요한 edge는 $p-1$개이므로 필요하므로 결과값에 $2/p$를 곱하여 예측한다.
- Monotone chain
  - pin을 한 축(x) 기준으로 sorting하여 순차 연결
- Star model
  - Timing critical net일 때는 source로부터 target까지 거리가 중요하므로, 각각 shortest 연결을 하는게 유리할 수 있다.
  - Star model로 이를 예측한다.



### Global Placement
  
#### Min-Cut Placement

#### Analytic Placement

#### Simulated Annealing

#### Modern Placement Algorithms

### Legalization and Detailed Placement