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
  - Timing critical net일 때는 source로부터 sink까지 거리가 중요하므로, 각각의 sink로 shortest 연결을 하는게 유리할 수 있다.
  - Star model로 이를 예측한다.

![2024-06-09-total-wirelength-estimation2.svg]({{site.baseurl}}/assets/images/2024-06-09-total-wirelength-estimation2.svg){: .align-center}

- Rectilinear minimum spanning tree (RMST)
  - Polynomial time
- Rectilinear Steiner minimum tree (RSMT)
  - Steiner point를 추가하여 Wirelength를 줄인다.
  - 가장 정확도가 높은 모델, 하지만 runtime 긴 편이다.
- Rectilinear Steiner arborescence model (RSA)
  - 각 sink로 Manhattan distance로 연결되면서 가장 total wirelength가 짧은 tree를 찾는다. (잘 사용되지 않음)
- Single-trunk Steiner Tree(RSMT)
  - trunk 중간에 큰 가지가 있고, 이를 기준으로 뻗어나옴.

#### Half-perimeter wirelength (HPWL)

- Wirelength 예측에 선호되는 방식은 HPWL 방식이다.
- 빠르고, 2-pin, 3-pin net은 RSMT와 동일한 결과를 제공한다.
- 실제 Circuit 대비 에러가 8% 정도로 나타남 (Chu, ICCAD 04)

#### Total wirelength with net weights

placement $P$에 대해 total weighted wirelength는 다음과 같이 정의된다.

$$L(P) = \sum_{net \in P} {w(net) \cdot L(net)}$$

- $w(net)$: net의 weight
- $L(net)$: net의 예측 wirelength

#### Cut sizes of a placement

Global vertical cutline과 horizontal cutline을 넘어가는 수를 계산한다.

$$L(P) = \sum_{v \in V_P} \psi_P(v) + \sum_{h \in H_P} \psi_P(h)$$

- $\psi_P(cut)$: cutline $cut$을 넘어가는 net들

![2024-06-09-cut-sizes-example.svg]({{site.baseurl}}/assets/images/2024-06-09-cut-sizes-example.svg){: .align-center}

#### Routing congestion of a placement

사용 가능한 routing track 대비 요구되는 track의 비율을 고려한다. 만약 capacity 대비 demand ratio가 1보다 크다면, 나중에 routing이 어려워질 수 있다.

#### Circuit timing of a placement

Interconnect delay를 고려하여 placement를 수행하는 방법이다. Critical path를 판별하기 위해서 static timing analysis 수행을 통해 AAT, RAT 계산 필요하다.

- Actual Arrival Time (AAT): AAT(v) represents the latest transition time at a given node v measured from the beginning of the clock cycle 
- Required Arrival Time (RAT): RAT(v) represents the time by which the latest transition at v must complete in order for the circuit to operate correctly within a given clock cycle.
- $AAT(v) \leq RAT(v)$를 만족해야 한다.

### Global Placement

- Partitioning-based algorithms
  - netlist와 layout을 작은 sub-netlist와 sub-region으로 나눈다.
  - 반복을 통해 optimal하게 처리할 수 있을 때까지 작게 나눈다.
  - Deailed placement는 작은 영역에 대해 optimal solver를 통해 수행된다.
  - Example: min-cut placement (KL, FM algorithm)
- Analytic techniques
  - 전체 netlist가 엮여있는 식으로 모델링
  - numerical analysis를 통해 optimal placement를 찾는다.
  - 제약 조건이 많아지면 수행하기 어려워진다(고려할 것이 많은 Detailed placement에는 적용 어렵고, global placement에 적합)
  - Examples: quadratic placement, force-directed placement
- Stochastic algorithms
  - 확률적으로 옮겨서 cost function을 최적화한다.
  - Example: simulated annealing, genetic algorithm

![2024-06-09-global-placement-methods.svg]({{site.baseurl}}/assets/images/2024-06-09-global-placement-methods.svg){: .align-center}

#### Min-Cut Placement

- Divide and Conquer 방식
- Kernighan-Lin (KL) algorithm
- Fiduccia-Mattheyses (FM) algorithm

![2024-06-09-min-cut-example.svg]({{site.baseurl}}/assets/images/`2024-06-09-min-cut-example.svg`){: .align-center}

1 → 2 → 3 → 4 순서로 partitioning 수행

```cpp
// Input: netlist Netlist, layout area LA, minimum number of cells per region cells_min
// Output: placement P
P = Ø
regions = ASSIGN(Netlist,LA)     // assign netlist to layout area
while (regions != Ø)             // while regions still not placed
    region = FIRST_ELEMENT(regions)  // first element in regions
    REMOVE(regions, region)          // remove first element of regions
    if (region contains more than cell_min cells)
        (sr1,sr2) = BISECT(region)  // divide region into two subregions
                                    //  sr1 and sr2, obtaining the sub-
                                    //  netlists and sub-areas
        ADD_TO_END(regions,sr1)  // add sr1 to the end of regions
        ADD_TO_END(regions,sr2)  // add sr2 to the end of regions
    else
       PLACE(region)    // place region
       ADD(P,region)    // add region to P 
```

![2024-06-09-min-cut-example-1.svg]({{site.baseurl}}/assets/images/2024-06-09-min-cut-example-1.svg){: .align-center}

![2024-06-09-min-cut-example-2.svg]({{site.baseurl}}/assets/images/2024-06-09-min-cut-example-2.svg){: .align-center}

#### Min-Cur Placement - Terminal Propagation

![2024-06-09-min-cut-terminal-propagation.svg]({{site.baseurl}}/assets/images/2024-06-09-min-cut-terminal-propagation.svg){: .align-center}

위의 예시에서 2↔4 사이의 net을 고려하지 않고 3, 4를 나누어서 wirelength가 길어졌다. 이를 해결하기 위해 terminal propagation을 사용한다.

- Min-cut Placement의 장점:
  - 빠르다
  - Objective function 조정이 가능하다
  - Large circuit에 대해 Hierarchical 접근이 가능하다.
- Min-cut Placement의 단점:
  - Randomized, chaotic algorithm
  - 하나의 cutline을 최적화 할 때 최종 결과에서 다른 곳에 routing congestion이 발생할 수 있다.

#### Analytic Placement - Quadratic Placement

- Objective funtion을 Quadratic function으로 정의한다.
- (weighted) squared Euclidean distance의 합이 placement objective function이 된다.

$$L(P) = \frac{1}{2} \sum_{i,j=1}^{n} c_{ij} \left( (x_i - x_j)^2 + (y_i - y_j)^2\right)$$

- $n$: the total number of cells
- $c_{ij}$: the connection cost between cell $i$ and $j$
- Manhattan distance가 더 정확하지만, Euclidean distance를 대신 사용한다.
- 제약 조건은 Linear equation을 사용한다.
- Only two-point-connection만 고려할 수 있다. (3-point 이상은 X)
- 이 방식의 장점은 2차식의 성질을 이용하여 미분을 0으로 만드는 linear equation을 풀어 objective function을 최소화한다.
- 각각의 Dimension은 따로 고려할 수 있다.
- Convex quadratic optimization problem
- 편미분을 0으로 만들어 x, y 값을 찾을 수 있다.

where A is a matrix with $A[i][j] = -c(i,j)$ when $i \neq j$, and $A[i][i] =$ the sum of incident connection weights of cell i.  
X is a vector of all the x-coordinates of the non-fixed cells, and bx is a vector with $bx[i] =$ the sum of x-coordinates of all fixed cells attached to i.  
Y is a vector of all the y-coordinates of the non-fixed cells, and by is a vector with $by[i] =$ the sum of y-coordinates of all fixed cells attached to i.  

![2024-06-09-quadratic-placement-equation.svg]({{site.baseurl}}/assets/images/2024-06-09-quadratic-placement-equation.svg){: .align-center}

![2024-06-09-quadratic-placement-example.png]({{site.baseurl}}/assets/images/2024-06-09-quadratic-placement-example.png){: .align-center}

y좌표에 대해서도 유사한 방식으로 풀 수 있다.

- Mechanical analogy: mass-spring system
  - 스프링으로 물체들이 연결된 상황과 유사하다.
  - 문제는 많은 overlap cell이 발생한다.
- Quadratic placer의 Second stage: cell을 분산시켜 overlap을 줄인다.
- 방법:
  - fake nets를 추가하여 cell을 anchor로 당기게 한다.
  - Geometric sorting and scaling: cell을 축소시켜서 겹치지 않게 하는 방식으로 보임.
  - Repulsion forces, etc.

![2024-06-09-quadratic-placement-second-stage.svg]({{site.baseurl}}/assets/images/2024-06-09-quadratic-placement-second-stage.svg){: .align-center}

- Advantages of Quadratic Placement
  - Mathematical term으로 정의됨
  - Numerical analysis 알고리즘과 소프트웨어 활용 가능
  - Large circuit에 적용 가능
  - Stability
- Disadvantages
  - Connections to fixed objects가 반드시 필요하다. (e.g. I/O pads, pins of fixed macros)

#### Analytic Placement - Force-Directed Placement

- Cell과 Wire를 mass-sping system 방식으로 모델링한다.
- Cell 사이의 Attraction force는 거리에 비례한다.
- Cell들은 최종적으로 force equilibrium에 settle한다. → Minimize wirelength
- 연결된 cell a, b에 대해 b로 인해 a에 가해지는 attraction force $\vec{F_{ab}}$
  - $\vec{F_{ab}} = c(a, b)\cdot (\vec{b} - \vec{a})$
  - $c(a, b)$: connection weight(priority) between a, b
  - $(\vec{b} - \vec{a})$: Euclidean 평면에서 a, b의 위치 차이 벡터
- Cell $i$에 가해지는 net force $\vec{F_i}$
  - $\vec{F_i} = \sum_{c(i, j) \neq 0} \vec{F_{ij}}$
  - $N(i)$: cell $i$에 연결된 모든 cell들의 집합
- Zero-force target(ZFT): force들의 합을 최소화하는 위치

![2024-06-09-quadratic-placement-zft.svg]({{site.baseurl}}/assets/images/2024-06-09-quadratic-placement-zft.svg){: .align-center}

##### Basic force-directed placement

- 반복적으로 모든 cell을 ZFT 위치로 이동시킨다.
- x-, y-방향 힘을 0으로 만들어야 한다.
  - $\sum_{c(i, j) \neq 0} c(i, j) \cdot (x_j^0 - x_i^0) = 0$
  - $\sum_{c(i, j) \neq 0} c(i, j) \cdot (y_j^0 - y_i^0) = 0$

![2024-06-09-force-directed-example-1.svg]({{site.baseurl}}/assets/images/2024-06-09-force-directed-example-1.svg){: .align-center}

![2024-06-09-force-directed-example-2.svg]({{site.baseurl}}/assets/images/2024-06-09-force-directed-example-2.svg){: .align-center}

##### Force-directed Placement Algorithm

```cpp
// Input: set of all cells V
// Output: placement P
P = PLACE(V)            // arbitrary initial placement
loc = LOCATIONS(P)      // set coordinates for each cell in P
foreach (cell c  V)
    status[c] = UNMOVED
while (ALL_MOVED(V) || !STOP()) // continue until all cells have been
                                //  moved or some stopping
                                //  criterion is reached
   c = MAX_DEGREE(V,status)     // unmoved cell that has largest 
                                //  number of connections
    ZFT_pos = ZFT_POSITION(c)   // ZFT position of c
    if (loc[ZFT_pos] == Ø)      // if position is unoccupied, 
         loc[ZFT_pos] = c       //  move c to its ZFT position
    else
         RELOCATE(c,loc)        // use methods discussed next
    status[c] = MOVED           // mark c as moved 
```

ZFT 위치가 겹치는 경우(p: incoming cell, q: p의 ZFT 위치에 있는 cell)

- 가능하다면, q에 가까운 위치로 p를 이동시킨다.
- Chain move: q의 위치로 cell p를 이동시킨다.
  - Cell q가 옆 위치로 이동, 만약 r이 이 공간이 차지하고 있다면, r을 옆으로 이동시킨다.
  - 이를 반복한다.
- p, q를 swap하였을 때 cost 차이를 계산한다. cost가 줄어들면 swap한다.

![2024-06-09-force-directed-example-3.svg]({{site.baseurl}}/assets/images/2024-06-09-force-directed-example-3.svg){: .align-center}

##### Force-directed Placement - Advantages and Disadvantages

- Advantages:
  - Concept이 간단하고 구현이 쉽다.
  - Global placement를 목표로 하지만, Detailed placement에도 적용 가능하다.
- Disadvantages:
  - Large placement instances에 대해 scalability가 떨어진다.
  - Densest region에 대한 cell speading이 비효율적
  - Solution quality와 runtime 사이에 trade-off 성능이 좋지 않다.
- In practice, cell spreading을 위한 테크닉들이 추가된다.
  - 이를 통해 Scalability와 성능 향상을 이룰 수 있다.

#### Simulated Annealing

Optimization의 여러 영역에서 사용되는 방법이다. 하지만 시간이 오래 걸린다는 단점이 있다.

초기 placement에서 randomly selected cells를 moving/exchanging한다.

- object function을 개선한다면 new placement를 Accept한다.
- If no improvement: Move/exchange is accepted with temperature-dependent (i.e., decreasing) probability

![2024-06-09-simulated-annealing-intro.svg]({{site.baseurl}}/assets/images/2024-06-09-simulated-annealing-intro.svg){: .align-center}

##### Simulated Annealing - Advantages and Disadvantages

- Advantages:
  - 충분한 시간이 주어진다면 global optimum을 찾을 수 있다.
  - Detailed placement에 잘 맞다.
- Disadvantages:
  - 시간이 오래 걸린다.
  - 많은 parameter 튜닝이 요구된다.
  - Randomized, chaotic algorithm
- Practical applications:
  - 복잡한 constraint를 가진 작은 placement instances
  - Detailed placement 시, small windows에 SA 적용 가능
  - FPGA layout(complicated constraints)

#### Modern Placement Algorithms

- 주로 Analytic algorithm 사용
- Solving two challenges:
  - Interconnect minimization
  - Cell overlap removal(spreading)
- Two families
  - Quadratic placers
  - Non-convex optimization placers

#### Quadratic Placers

- Force-directed placement를 사요한 linear equation을 iterative하게 풀어나간다. Conjugate gradient algorithm 사용
- Fake nets를 사용하여 dense cell을 당긴다.

#### Non-convex optimization placers

- Sophisticated differentiable function으로 interconnect model(e.g. log-sum-exp)
- Cell overlap과 fixed obstacle을 추가적은 (non-convex) term 사용
- non-linear Conjugate gradient algorithm 사용해서 최적화
- Sophisticated, slow
- Net clustering을 통해 그룹화하여 큰 단위로 배치를 먼저 수행하여 scalability 향상

#### Modern Placement - Summary

- Pros and Cons
  - Quadratic placers: simple, faster
  - Non-convex optimization placers: better solutions
  - As of 2011, quadratic placers are catching up in solution quality while running 5-6 times faster

### Legalization and Detailed Placement

실제로는 Cell 위치 공간 제약이 있고, 겹쳐서도 안된다. (global placement에서는 이에 대해 완전히 고려하지는 않음).

- Global placement는 legalized되어야 한다.
  - Cell localtion이 power rail과 align되지 않는 상태이다.
  - Small cell overlap이 발생할 수 있다(cell resizing이나 buffer insertion으로 인해)
- Legalization은 legal, non-overlapping placement를 찾는다.
  - Global placement 결과를 최대한 보존
- 다음과 같은 Detailed placement technique으로 Legalization을 향상시킬 수 있다.
  - Wirelength를 줄이기 위해 이웃한 cell swapping
  - Unused space로 cell을 sliding
- Legalization과 Detailed placement는 software 구현에서 함께 구현되는 경우가 많다.

![2024-06-09-legalization-intro.svg]({{site.baseurl}}/assets/images/2024-06-09-legalization-intro.svg){: .align-center}
