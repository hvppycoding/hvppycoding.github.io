---
title: "Chapter 7. Specialized Routing"
excerpt: "VLSI Physical Design"
date: 2024-06-24 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

- Area routing: global, detailed routing 단계를 나누지 않고 바로 metal route를 만드는 방법, design size가 크지 않을 때 사용할 수 있다.
- Non-Manhattan routing
- Clock signals

### Introduction to Area Routing

- The goal of area routing is to route all nets in the design
  - without global routing
  - within the given layout space
  - while meeting all geometric and electrical design rules
- Area routing performs the following optimizations
  - minimizing the total routed length and number of vias of all nets
  - minimizing the total area of wiring and the number of routing layers
  - minimizing the circuit delay and ensuring an even(골고루 배치) wire density
  - avoiding harmful capacitive coupling between neighboring routes
- Subject to
  - technology constraints (number of routing layers, minimal wire width, etc.)
  - electrical constraints (signal integrity, coupling, etc.)
  - geometry constraints (preferred routing directions(레이어별 주 사용 방향이 있으나 약간은 위반할 수 있다), wire pitch, etc.)

### Net Ordering in Area Routing

![2024-06-24-net-ordering-effect.svg]({{site.baseurl}}/assets/images/2024-06-24-net-ordering-effect.svg){: .align-center}

- net routing 순서에 따라 routing 결과가 달라진다. → 이에 대한 고려 필요
- $n$ net에 대해서 $n!$가지의 조합이 존재한다. → Constructive heuristic 사용

![2024-06-24-net-ordering-rule1.svg]({{site.baseurl}}/assets/images/2024-06-24-net-ordering-rule1.svg){: .align-center}

Aspect ratio가 큰 것을 먼저하자.

![2024-06-24-net-ordering-rule2.svg]({{site.baseurl}}/assets/images/2024-06-24-net-ordering-rule2.svg){: .align-center}

Bounding box 안에 핀이 존재는 다른 net을 먼저하자.  
(Cycle이 생기는 경우 가능한 조합 중 하나 선택)

![2024-06-24-net-ordering-rule3.svg]({{site.baseurl}}/assets/images/2024-06-24-net-ordering-rule3.svg){: .align-center}

Net bounding box 안에 포함된 다른 net pin의 수가 많은 net을 나중에 하자.  
(앞의 Rule 2와 유사)

### Non-Manhattan Routing

생략

#### Octilinear Steiner Trees

생략

#### Octilinear Maze Search

생략

### Basic Concepts in Clock Networks

#### Terminology

- **clock routing instance(clock net)**은 $n+1$ terminal을 가지며, $s_0$는 source, $S=\{ s_1, s_2, ..., s_n\}$은 sink이다.
  - $s_i$는 terminal과 location 모두 의미
- clock routing solution은 모든 terminal을 연결하는 wire segment의 집합이다.
  - Two aspects of clock routing solution: topology and geometric embedding
- clock-tree topology (clock tree)는 rooted binary tree $G$ with $n$ leaves(각 sink에 해당)
  - Internal nodes = Steiner points

![2024-06-24-clock-net-terms.svg]({{site.baseurl}}/assets/images/2024-06-24-clock-net-terms.svg){: .align-center}

- clock **skew**: 각 sink별 clock signal arrival time의 (최대) 차이(global skew)
  - $skew(T) = \max_{s_i, s_j \in S} \| t(s_0, s_i) - t(s_0, s_j) \|$
- **Local skew**: flip-flop $i$의 output에서 flip-flop $j$의 input으로 가는 combinational path가 존재할 경우, 두 flip-flop의 clock signal arrival time의 차이
- Zero skew: 모든 clock을 동기화하는 경우(ideal)
- Bounded skew: 완전한 zero skew는 실제로 필요하지 않을 수 있다.
  - non-zero skew bound를 이용한 sign off timing analysis만으로 충분할 수 있다.
- **Useful skew**: local skew를 이용하여 chip timing을 개선하는 것

### Modern Clock Tree Synthesis

1. Initial tree construction(중요)
2. Clock buffer insertion(보강)

#### Constructing Trees with Zero Global Skew

##### H-tree

![2024-06-24-h-tree.svg]({{site.baseurl}}/assets/images/2024-06-24-h-tree.svg){: .align-center}

- H-tree의 symmetry를 이용한 zero skew
- top-level clock distribution에 사용되며, 모든 clock tree에는 적용하기 어렵다.
  - Blockage로 인한 라우팅 방해
  - 실제 sink capacitance와 localtion이 균일하지 않음

##### Method of Means and Medians(MMM)

- 임의 위치의 clock sink 다룰 수 있다.
- Basic idea:
  - terminal들을 같은 사이즈의 두 개의 subset으로 partition
  - 각 subset의 Center of Gravity(COG)를 구하고 연결

![2024-06-24-method-of-means-and-medians.svg]({{site.baseurl}}/assets/images/2024-06-24-method-of-means-and-medians.svg){: .align-center}

##### Recursive Geometric Matching(RGM)

- RGM은 bottom-up fashion으로 접근(MMM은 top-down)
- Basic idea:
  - $n$ sinks를 minimum-cost geometric matching
  - $n$ endpoints를 매칭하는 $n/2$개의 line segments 찾는다.
  - 연관된 sink들에 대해 zero skew를 유지하도록 tapping point를 조절한다.
  - $n/2$ tapping points에 대해 다음 matching step을 진행한다.

![2024-06-24-recursive-geometric-matching.svg]({{site.baseurl}}/assets/images/2024-06-24-recursive-geometric-matching.svg){: .align-center}

##### Exact Zero Skew

- RGM과 유사한 bttom-up process 사용
- 개선점:
  - RC를 고려한 Elmore delay model 고려하여 tapping point 찾는다.
  - wire elongation을 통해 두 subtree의 delay balance를 맞춘다.

![2024-06-24-exact-zero-skew.svg]({{site.baseurl}}/assets/images/2024-06-24-exact-zero-skew.svg){: .align-center}

##### Deferred-Merge Embedding(DME)

- 현재 주로 사용되는 방법
- Bottom-up 후 top-down 수행

![2024-06-24-manhattan-midpoints.svg]({{site.baseurl}}/assets/images/2024-06-24-manhattan-midpoints.svg){: .align-center}

Manhattan midpoints는 한 점이 아니라 여러 곳이 가능하다.

![2024-06-24-tilted-rect-region.svg]({{site.baseurl}}/assets/images/2024-06-24-tilted-rect-region.svg){: .align-center}

![2024-06-24-merging-segment.svg]({{site.baseurl}}/assets/images/2024-06-24-merging-segment.svg){: .align-center}

하위 radius를 고려하여 밸런스를 맞출 수 있도록 상위 노드의 merging segment를 찾는다. 위 예시에서 s3과 s4 사이는 가깝기 때문에 u3에서는 더 멀게 되도록 만들었다.

![2024-06-24-dme-bottom-up-phase.svg]({{site.baseurl}}/assets/images/2024-06-24-dme-bottom-up-phase.svg){: .align-center}

Bottom-up phase에서 정확한 점이 아닌, segment(선)이 결정되었다. top-down phase에서 정확한 위치를 결정한다.

![2024-06-24-dme-top-down-phase.svg]({{site.baseurl}}/assets/images/2024-06-24-dme-top-down-phase.svg){: .align-center}

#### Clock Tree Buffering in the Presence of Variation

- Clock tree의 skew constraint외에도 다른 optimization step이 있다.
  - Geometric clock tree construction
  - Initial clock buffer insertion
  - Clock buffer sizing
  - Wire sizing
  - Wire snaking(wire 꼬불꼬불하게 만들어 delay 늘리는 방안)
  - 전통적인 clock tree 대신 mesh 형태로 만드는 방법도 존재(clock mesh), 안정적이나 metal 요구량 증가한다는 단점이 있다.
- Process, voltage, temperature variation 영향에 대한 모델링과 최적화
  - Variation model encapsulates the different parameters, such as width and thickness, of each library element as well-defined random variables
