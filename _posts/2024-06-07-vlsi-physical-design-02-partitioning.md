---
title: "Chapter 2. Netlist and System Paritioning"
excerpt: "VLSI Physical Design"
date: 2024-06-07 06:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

![2024-06-07-partitioning.png]({{site.baseurl}}/assets/images/2024-06-07-partitioning.png){: .align-center}

### Introduction

![2024-06-07-partitioning-introduction.png]({{site.baseurl}}/assets/images/2024-06-07-partitioning-introduction.png){: .align-center}

- Block을 나누었을 때 cut을 줄이는 것이 중요하다.

### Optimization Goals

- graph $G(V, E)$가 주어졌을 때, *k*개의 disjoint subgraph로 나누는 것이 목표이다
- 디테일 목표는?
  - Partition 간의 connection을 줄인다.
  - 각 partition이 design constraint를 만족시킨다.
  - 각 partition이 balanced한 것이 좋다.
- 이 문제는 NP-hard이다.
- 1970, 80년대에 heuristic algorithm들이 제안되었다.

### Kernighan-Lin (KL) Algorithm

- *2n*개 노드를 가진 그래프가 주어짐. 각 노드의 weight는 동일하다.
- 목표: minimum cut cost인 2 disjoint subset $A$와 $B$로 나눈다. $|A| = |B| = n$를 만족하도록 한다.

어떤 노드 $v$를 반대 subset으로 옮길 때 cost를 $D(v)$라고 하자.

$$D(v) = |E_c(v)| - |E_{nc}(v)|$$

- $E_c(v)$: cutline에 의해 cut되는 v의 incident edge의 수
- $E_{nc}(v)$: cutline에 의해 cut되지 않는 v의 incident edge의 수

High cost ($D > 0$)는 그 노드를 옮겨야 함을 의미한다.

![2024-06-07-kl-algorithm-1.png]({{site.baseurl}}/assets/images/2024-06-07-kl-algorithm-1.png){: .align-center}

노드 *a*와 *b*를 서로 교환하는 경우를 고려해보자.

$$\Delta g = D(a) + D(b) - 2 \times c(a, b)$$

- $c(a, b)$: $a$와 $b$ 사이의 edge weight(here 1), 없으면 0

$\Delta g$이 양수이면 교환하는 것이 유리하다.

![2024-06-07-kl-algorithm-2.png]({{site.baseurl}}/assets/images/2024-06-07-kl-algorithm-2.png){: .align-center}

#### Maximum positive gain $G_m$ of a pass

The maximum positive gain $G_m$ corresponds to the best prefix of $m$ swaps within the swap sequence of a given pass.

![2024-06-07-kl-algorithm-example.png]({{site.baseurl}}/assets/images/2024-06-07-kl-algorithm-example.png){: .align-center}

본 예시에서 $m = 2$일 때 Maximum positive gain $G_m$은 8이 된다.
$m = 2$까지 적용하고 다시 $D$를 계산하여 알고리즘을 재수행한다.
가능한 $G_m$이 0이 될 때까지 반복한다.

#### Extensions of the Kernighan-Lin (KL) Algorithm

- Unequal partition sizes: 두 partition의 크기가 다를 때 → dummy node를 추가하여 KL 알고리즘을 적용한다.
- Unequal node weights: 노드별 size가 크게 차이나는 경우
- k-way partitioning: two-way partitioning을 여러 번 적용하여 2 → 4 → 8과 같이 증가시킬 수 있다.

### Fiduccia-Mattheyses (FM) Algorithm

- KL에서는 두 노드를 교환하는 것을 고려했다면, FM에서는 한 노드를 다른 partition으로 이동시키는 것을 고려한다.
- Partition size를 완전히 균등하게 만들지 않고, 일정 range 내에서 관리한다.
- 3개 이상의 핀을 가진 net의 cut cost 계산시 hypergraph로 관리한다.

#### Gain $\Delta g(c)$ for cell $c$

$$\Delta g(c) = FS(c) - TE(c)$$

- $FS(c)$: $c$에 연결된 net이면서 동일 partition의 cell에 연결되지 않은 net의 수, moving force
- $TE(c)$: $c$에 연결된 uncut net의 수, retention force

![2024-06-07-fm-algorithm-example.png]({{site.baseurl}}/assets/images/2024-06-07-fm-algorithm-example.png){: .align-center}

Cell 1: FS(1) = 2 TE(1) = 1 Δg(1) = 1
Cell 2: FS(2) = 0 TE(2) = 1 Δg(2) = -1
Cell 3: FS(3) = 1 TE(3) = 1 Δg(3) = 0
Cell 4: FS(4) = 1 TE(4) = 1 Δg(4) = 0
Cell 5: FS(5) = 1 TE(5) = 0 Δg(5) = 1

Δg(1)과 Δg(5)가 최선이므로 둘 중 하나를 이동시킨다.

#### Balance criterion

- Balance criterion을 만족하는 경우에만 이동시킨다.
- 가장 큰 cell도 이동시킬 수 있어야한다.

$$r \cdot area(V) - area_{max}(V) \leq area(A) \leq r \cdot area(V) + area_{max}(V)$$

- $r$: ratio factor
- $area_{max}(V)$: maximum cell area

이동한 cell은 고정시킨 후, KL과 마찬가지 방식으로 이동을 반복한다. 
step 중 gain이 최대인 조합 중 balance가 가장 좋은 solution을 선택하고, 이를 base로 하여 pass gain이 0이 될 때까지 반복한다.

### Runtime difference between KL & FM

- Runtime of partitioning algorithms
  - KL is sensitive to the number of nodes and edges
  - FM is sensitive to the number of nodes and nets(hyperedges)
- Asymptotic complexity of partitioning algorithms
  - KL: cubic time complexity *per pass*
  - FM: linear time complexity *per pass*

### Framework for Multilevel Partitioning

#### Clustering

- Component들을 grouping하여 전체적인 graph의 크기를 줄인다.
- Cluster size가 너무 unbalanced하지 않도록 제한을 둔다.

![2024-06-07-clustering.png]({{site.baseurl}}/assets/images/2024-06-07-clustering.png){: .align-center}

clustering의 예시{: .custom-caption}

#### Multilevel Partitioning

![2024-06-07-multilevel-partitioning.png]({{site.baseurl}}/assets/images/2024-06-07-multilevel-partitioning.png){: .align-center}

문제 사이즈를 줄여 문제를 푼 뒤, 이를 바탕으로 문제를 풀어나갈 수 있다. Detail한 건 나중에 생각하고, Global한 문제를 먼저 해결하는 아이디어.

### Summary

- Circuit netlist는 그래프로 표현될 수 있다.
- Partitioning은 graph 노드를 disjoint partition들로 나누는 것이다.
  - 각 파티션의 크기는 제한됨
  - Objective: minimize the number connections between partitions