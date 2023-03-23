---
title: "[논문 리뷰] DREAMPlace"
excerpt: "Deep Learning Toolkit-Enabled GPU Acceleration for Modern VLSI Placement"
date: 2023-03-15 11:00:00 +0900
mathjax: "true"
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - 논문 리뷰
---

## DREAMPlace: Deep Learning Toolkit-Enabled GPU Acceleration for Modern VLSI Placement

### Abstract

VLSI 회로의 placement는 설계에서 중요한 단계 중 하나이다. 본 논문에서는 GPU accelerated placement framework인 DREAMPlace를 소개한다. DREAMPlace는 global placement에서 state-of-the-art multi-threaded placer RePlAce 대비 quality 저하 없이 30x 속도 향상을 이루었다.

### 1. Introduction

Placement는 VLSI 설계에서 중요한 & 시간이 걸리는 단계이다. 스탠다드 셀들의 피지컬 레이아웃에서의 위치를 결정하기 때문에 Placement quality는 이후 스테이지(routing, post-layout optimization)에 큰 영향을 미친다. 또한 placement 솔루션은 routed wire-length와 congestion을 상당히 잘 예측한다. Commercial design flow에서 core placement engine을 여러 번 수행하여 design closure를 달성한다. 현재 placement는 큰 design에 대해 수 시간이 걸리고 design iteration을 늦춘다. 그러므로 ultra-fast & high quality placement는 항상 요구된다.  

Analytical placement는 현재 VLSI placement에서 state-of-the-art이다. 이는 nonlinear optimization problem을 푼다. analytical placement가 high-quality solution을 제공하지만, 이는 상대적으로 느리다. analytical placement problem의 간단한 설명은 다음과 같다. circuit을 hypergraph $H = (V, E)$로 기술하자. $V$는 vertices(cells)를 뜻하고, $E$는 hyperedges(nets)이다. $x$와 $y$는 cell의 위치를 나타낸다. analytical placement의 목표는 wirelength를 최소화하고 레이아웃이 겹치지 않는 cell들의 위치를 결정하는 것이다.  

placement를 가속화하기 위해서, 현존하는 병렬화는 대부분 partitioning을 통한 multi-threaded CPU 방식이다. thread 수가 올라감에 따라 속도는 5x 정도에 saturation되고 있고 quality 2-6% degradation이 발생한다. Cong은 analytical placement에 GPU acceleration을 연구하였다. They combined clustering and declustering with nonlinear placement optimization. nonlinear placement part를 parallelizing하여 평균 12x 속도 향상과 1% 미만의 quality degradation 결과가 report되었다. Lin은 wirelength gradient computation과 area accumulation GPU 가속 테크닉을 제안하였으나, 실험은 real operation(density cost computation) 고려 실패, real analytical placement flow 부족하였다. 현재 placement 연구는 well-maintained public frameworks가 부족하고 높은 development overhead로 인해 시스템적으로 새 알고리즘을 검증하는 것이 어려워지고 있다.  

본 연구에서 deep learning 툴킷인 `PyTorch`로 개발된 GPU-accelerated analytical placer `DREAMPlace`를 제안한다. DREAMPlace는 state-of-the-art anlytical placement algorithm인 ePlace/RePlAce family이다. 하지만 framework는 NTUplace와 같은 다른 analytical placer와 호환 가능하도록 일반화된 방식으로 설계되었다. 주요 contribution은 다음과 같다.

- 효율적인 GPU 구현을 제안한다. analytical placement like wirelength and density computation
- global placement와 legalization에서 (multi-threaded CPU 구현 대비) quality degradation 없이 30x 속도 향상이 가능함을 보였다. 구체적으로는 백만개의 셀 설계에 대해 1분 안에 수행되었다. 프레임워크는 10 million cells까지 거의 선형적 scalability를 유지한다.

소스 코드는 [Github](https://github.com/limbo018/DREAMPlace)에 release되어 있다. Placement 문제를 deep learning 문제로 캐스팅하는 것은 toolkit을 사용하여 placement를 해결하는 것을 목표로 한다. 

### 2. Preliminaries

background와 motivation을 review한다.

#### 2.1 Analytical Placement

Analytical placement는 대개 3-step으로 이루어져 있다. global placement(GP), legalization(LG), detailed placement(DP). 

### 단어

- essentially: 근본적으로
- analogy: 비유, 유사점