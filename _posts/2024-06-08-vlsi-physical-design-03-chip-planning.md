---
title: "Chapter 3. Chip Planning"
excerpt: "VLSI Physical Design"
date: 2024-06-08 09:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

![2024-06-08-chip-planning-intro.png]({{site.baseurl}}/assets/images/2024-06-08-chip-planning-intro.png){: .align-center}

큰 Design 회로를 Partitionning 후, 각 Partition들의 물리적 위치, 핀, 파워 그라운드를 구성하는 단계가 Chip Planning이다.

### Introduction

![2024-06-08-chip-planning.png]({{site.baseurl}}/assets/images/2024-06-08-chip-planning.png){: .align-center}

Partitioning된 Block들을 칩 위에 위치를 잡아야 한다(Floorplan). 

![2024-06-08-floorplan-example.png]({{site.baseurl}}/assets/images/2024-06-08-floorplan-example.png){: .align-center}

Block A, B, C를 위와 같이 배치하면 9 square unit area만 사용할 수 있다.

### Optimization Goals in Floorplanning

- 목표 1: Area and shape of the global bounding box
  - 모든 floorplan block을 포함하는 최소 사각형
  - global bounding box의 면적은 top-level floorplan의 area를 나타낸다.
  - 각 block의 모양과 위치를 찾아야 한다.
- 목표 2: Total wirelength
  - Block 간의 긴 connection은 signal propagation delay를 증가시킨다.
- 목표 3: 목표 1 area(F)와 목표 2 total wirelength L(F)를 모두 고려
  - Minimize $\alpha \cdot area(F) + (1 - \alpha) \cdot L(F)$
  - F: floorplan
- 목표 4: Signal delays
  - static timing analysis를 통해 Critical path에 놓여있는 interconnect를 찾아 최소화한다.(But, 이 단계에서 이 정도로 디테일하게 하는 것은 시기 상조로 보임)

### Terminology

- **rectangular dissection**: 겹치지 않는 사각형으로 나누는 것
- **slicing floorplan**: rectangular dissection의 한 방법으로 전체 면적을 가로, 세로로 쪼개 나가는 방식
- **slicing tree**, **slicing floorplan tree**: slicing floorplan을 나타내는 binary tree
  - 각 leaf node는 block을 나타낸다.
  - 각 internal node는 horizontal 또는 vertical cut line을 나타낸다.

![2024-06-08-policy-expression.png]({{site.baseurl}}/assets/images/2024-06-08-policy-expression.png){: .align-center}

Floorplan, Slicing tree, Polish notation 예시
{: .custom-caption}

위와 같은 polish notation 표현식을 통해 나타내는 것도 가능하다.

#### Constraint Graph

Vertical constraint graph(VCG), Horizontal constraint graph(HCG)

![2024-06-08-constraint-graphs.png]({{site.baseurl}}/assets/images/2024-06-08-constraint-graphs.png){: .align-center}

이 두 그래프를 이용해 가로, 세로의 최소 길이를 찾을 수 있다.

#### Sequence Pair

![2024-06-08-sequence-pair.png]({{site.baseurl}}/assets/images/2024-06-08-sequence-pair.png){: .align-center}

- 2개의 sequence로 구성된 sequence pair로 floorplan을 나타낼 수 있다.
- 위 예시에서 A와 B의 관계에 따라 두 block의 상대적 위치를 알 수 있다.

### Floorplan Representations

#### Floorplan to a Constraint-Graph Pair

Floorplan에서 constraint graph를 만드는 방법

상대적 위치에 따라 모든 edge를 추가한다. 만약 다른 노드를 통해 연결될 수 있는 edge는 지운다. 

#### Floorplan to a Sequence Pair

![2024-06-08-floorplan-to-seq-pair.png]({{site.baseurl}}/assets/images/2024-06-08-floorplan-to-seq-pair.png){: .align-center}

#### Sequence Pair to a Floorplan

Sequence pair로부터 가로, 세로 길이를 찾아내려면? 가장 쉬운 것은 HCG, VCG를 만들어서 찾아내는 것이다. Graph를 만들지 않고 바로 찾아내는 방법도 있다.

Longest common subsequence(LCS)를 활용한다. 하지만 보통의 LCS보다 어려운 점은 각각의 character의 weight가 1이 아니라 block의 가로 혹은 세로 길이가 된다는 점이다.

$LCS(S_+, S_-)$는 모든 block에 대한 x 좌표를 리턴한다.
$LCS(S_+^R, S_-)$는 모든 block에 대한 y 좌표를 리턴한다.

알고리즘은 다음과 같다.

```cpp
// Algorithm: Longest Common Subsequence (LCS)
// Input: sequences S1 and S2, weights of n blocks weights
// Output: positions of each block positions, total span L

 for (i = 1 to n)					// initialization
  block_order[S2[i]] = i
  lengths[i] = 0
 for (i = 1 to n)
  block = S1[i]					// current block
  index = block_order[block]
  positions[block] = lengths[index]			// compute block position
  t_span = positions[block] + weights[block]		// finds length of sequence from beginning to block
  for (j = index to n)				// update total length
    if (t_span > lengths[j]) lengths[j] =  t_span
    else break
 L = lengths[n]					// total length is stored here
 ```

![2024-06-08-sequence-pair-to-x.png]({{site.baseurl}}/assets/images/2024-06-08-sequence-pair-to-x.png){: .align-center}

![2024-06-08-sequence-pair-to-y.png]({{site.baseurl}}/assets/images/2024-06-08-sequence-pair-to-y.png){: .align-center}

### Floorplanning Algorithms

- Common goals
  - Minimize total length of interconnect(area 상한선을 넘지 않는 선에서)
  - 혹은 wire length와 area를 동시 최적화

#### Floorplan Sizing

polynomial time 안에 slicing floorplan의 minimum floorplan area를 찾는 알고리즘.
(non-slicing floorplan 문제는 NP-hard)

- 각각의 block에 대해 shape function을 만든다.
- Bottom up: 각각의 block shape function으로부터 top-level floorplan의 shape function을 만든다.
- Top down: 코너포인트에서부터 각 block의 shape function을 trace back한다.

![2024-06-08-floorplan-sizing-1.svg]({{site.baseurl}}/assets/images/2024-06-08-floorplan-sizing-1.svg){: .align-center}

![2024-06-08-floorplan-sizing-2.svg]({{site.baseurl}}/assets/images/2024-06-08-floorplan-sizing-2.svg){: .align-center}

![2024-06-08-floorplan-sizing-3.svg]({{site.baseurl}}/assets/images/2024-06-08-floorplan-sizing-3.svg){: .align-center}

#### Cluster Growth

Initial floorplan을 빨리 만드는 방법. 전체 면적을 최소로 유지하면서 하나씩 block을 추가한다.

![2024-06-08-cluster-growth.svg]({{site.baseurl}}/assets/images/2024-06-08-cluster-growth.svg){: .align-center}

##### Linear Ordering

- New nets: 이미 생성된 ordering에 pin이 없는 net
- Terminating nets: 미배치된 incident block이 없음
- Continuing nets: partially constructed ordering에 최소 1핀, unplaced block의 pin에 최소 1핀이 있는 block

- place될 때마다 각 Net의 Gain을 구해서 가장 큰 Gain을 가진 Net을 넣는다.

$$Gain_m = (\text{#terminating nets of }m) - (\text{#new nets of }m)$$

![2024-06-08-cluster-growth-linear-ordering.png]({{site.baseurl}}/assets/images/2024-06-08-cluster-growth-linear-ordering.png){: .align-center}

![2024-06-08-cluster-growth-linear-ordering2.svg]({{site.baseurl}}/assets/images/2024-06-08-cluster-growth-linear-ordering2.svg){: .align-center}

![2024-06-08-cluster-growth-linear-ordering-result.svg]({{site.baseurl}}/assets/images/2024-06-08-cluster-growth-linear-ordering-result.svg){: .align-center}

랜덤으로 배치한 경우(위쪽)보다 Linear Ordering을 사용한 경우(아래쪽)가 더 짧은 total wirelength를 보인다.

```cpp
// Input: set of all blocks M, cost function C
// Output: optimized floorplan F based on C
F = Ø
order = LINEAR_ORDERING(M)		// generate linear ordering
for (i = 1 to |order|)
      curr_block = order[i]
      ADD_TO_FLOORPLAN(F,curr_block,C)
      // find location and orientation of curr_block 
      // that causes smallest increase based on 
      // C while obeying constraints 
```

- 목적은 total wirelength를 최소화하는 것이다.
- 적당한 솔루션을 찾으며(최적 X), 알고리즘이 쉽고 빠르다.
- simulated annealing과 같은 iterative algorithm을 위한 initial floorplan 생성 시 사용할 수 있다.

#### Simulated Annealing

- Simulated Annealing(SA)는 iterative한 최적화 방법
- initial(arbitrary) solution에서 시작하여 incremental하게 solution을 개선한다. 초기 solution이 좋을수록 최종 solution도 좋을 가능성이 높다.
- 각 iteration에서 현재 솔루션의 local neighborhood solution을 찾는다. 현재 솔루션에 small perturbation을 가하여 새로운 소루션을 만들 수 있다.
- greedy solution과 다르게, SA algorithm은 higher cost인 경우도 accept할 확률을 가져 local optimum에 빠질 확률을 낮춘다. (global optimum을 보장하지는 않는다.)

- **Annealing?**
  - 높은 온도에서는 Unstable한 상태로 빠르게 움직이지만, 점점 낮은 온도로 가면서 안정화된 상태로 이동한다.
  - highly randomized한 상태에서 stable한 상태로 변한다.
  - 자연에서 atom의 low-temperature state는 확률적으로 결정된다.

- **Simulated Annealing**
  - 시작 state $S_{init}$, 새로운 솔루션은 $S_{new}$
  - $r$: random number in [0, 1)

$$e^{\frac{C(S_{new}) - C(S_{init})}{T}} > r$$

```cpp
// Input: initial solution init_sol
// Output: optimized new solution curr_sol
T = T0					// initialization
i = 0
curr_sol = init_sol
curr_cost = COST(curr_sol)
while (T > Tmin)
    while (stopping criterion is not met)
      i = i + 1
      (ai,bi) = SELECT_PAIR(curr_sol)		// select two objects to perturb
      trial_sol = TRY_MOVE(ai,bi)		// try small local change
    trial_cost = COST(trial_sol)
      cost = trial_cost – curr_cost
      if (cost < 0)				// if there is improvement,
            curr_cost = trial_cost		//  update the cost and 
              curr_sol = MOVE(ai,bi)		//  execute the move
      else							
            r = RANDOM(0,1)		// random number [0,1]
            if (r < e –Δcost/T)			// if it meets threshold,
                    curr_cost = trial_cost		//   update the cost and
                    curr_sol = MOVE(ai,bi)		//   execute the move
     T = α ∙ T				// 0 < α < 1, T reduction 
```

#### Optimal Approach - Branch and Bound

- Optimum result를 보장하는 방법
- Excessive runtime
  - 작은 문제에 적용 가능
  - Heuristics 적용 필요
- Decision tree
  - 가지치기를 통해 불필요한 계산을 줄일 수 있다.
  - 각 노드에서 앞으로 가능한 solution의 lower bound를 예측한다. (Lower bound 계산 방법론 필요하다. Lower bound가 tight할수록 가지치기가 더 효과적이다.)
  - 만약 lower bound가 현재 best solution보다 크다면, 해당 노드는 더 이상 탐색하지 않는다.

### Pin Assignment

- Chip planning의 중요한 단계
  1. Floorplanning: block의 위치와 크기를 결정
  2. Pin assignment: 핀 위치
  3. P/G network

![2024-06-08-pin-assignment-intro.svg]({{site.baseurl}}/assets/images/2024-06-08-pin-assignment-intro.svg){: .align-center}

![2024-06-08-pin-assignment-ex1.svg]({{site.baseurl}}/assets/images/2024-06-08-pin-assignment-ex1.svg){: .align-center}

![2024-06-08-pin-assignment-ex2.svg]({{site.baseurl}}/assets/images/2024-06-08-pin-assignment-ex2.svg){: .align-center}

![2024-06-08-pin-assignment-ex3.svg]({{site.baseurl}}/assets/images/2024-06-08-pin-assignment-ex3.svg){: .align-center}

첫 시작 라인은 random하게 선을 1:1로 연결하고, 시계방향으로 돌아가면서 1:1로 순차적으로 연결한다.

첫 시작 라인을 바꿔보며 재시도하며 최적 euclidean distance를 가지는 mapping을 찾는다.

![2024-06-08-pin-assignment-ex4.svg]({{site.baseurl}}/assets/images/2024-06-08-pin-assignment-ex4.svg){: .align-center}

#### External block pin assignment

![2024-06-08-pin-assignment-external.svg]({{site.baseurl}}/assets/images/2024-06-08-pin-assignment-external.svg){: .align-center}

#### Two external block pin assignment

![2024-06-08-pin-assignment-external2.svg]({{site.baseurl}}/assets/images/2024-06-08-pin-assignment-external2.svg){: .align-center}

### Power and Ground Routing

Power Ground 라우팅 = Power Ground 넷을 깔아야 한다.  

각 block에 power, ground를 공급해야 한다. ring 구조를 사용한다.  

![2024-06-08-pg-routing-intro.svg]({{site.baseurl}}/assets/images/2024-06-08-pg-routing-intro.svg){: .align-center}

#### Planar Routing

Planar routing에서는 power, ground가 같은 layer에 깔린다.

![2024-06-08-pg-routing-planar.svg]({{site.baseurl}}/assets/images/2024-06-08-pg-routing-planar.svg){: .align-center}

#### Mesh Routing

- Step 1: Creating a ring
  - A ring is constructed to surround the entire core area of the chip, and possibly individual blocks. For low resistance, these connections and the ring itself are on many layers. For example, a ring might use metal layers Metal2–Metal8(every layer except Metal1).
- Step 2: Connecting I/O pads to the ring
- Step 3: Creating a mesh
  - A power mesh consists of a set of stripes at defined pitches on two or more layers
  - Power mesh는 최상위의 가장 두꺼운 레이어를 사용하고, lower layer에서는 sparse 해진다.(signal routing congestion 방지)
  - 인접 레이어의 power와 많은 via를 통해 연결된다(resistance를 줄이기 위해).
- Step 4: Creating Metal1 rails (power rail, ground rail)
  - Metal1 layer는 power-ground distribution network가 logic gate를 만나는 지점이다.
  - Metal1 rail pitch는 주로 standard cell library에 의해 정해진다. 
  - Standard cell rows는 "back-to-back"으로 배치되어 supply net은 두 인접한 cell row 끼리 공유하여 사용된다.
- Step 5: Connecting the Metal1 rails to the mesh  
  - Metal1 rail과 mesh가 연결된다. via 수에 대한 고려가 필요하다.
  - via stack 선정 시 routability도 고려해야 한다.

**OpenROAD:** The `add_pdn_ring` command is used to define power/ground rings around a grid region. The ring structure is built using two layers that are orthogonal to each other. A power/ground pair will be added above and below the grid using the horizontal layer, with another power/ground pair to the left and right using the vertical layer. Together these 4 pairs of power/ground stripes form a ring around the specified grid. Power straps on these layers that are inside the enclosed region are extend to connect to the ring.
{: .notice--warning}

![2024-06-08-pg-mesh-routing-1.svg]({{site.baseurl}}/assets/images/2024-06-08-pg-mesh-routing-1.svg){: .align-center}

![2024-06-08-pg-mesh-routing-2.svg]({{site.baseurl}}/assets/images/2024-06-08-pg-mesh-routing-2.svg){: .align-center}

우리는 Mesh 층을 얼마나 깔지 결정할 수 있다. M1-M10을 Back End of Line(BEOL)이라고 한다. BEOL은 Power/Ground network(chip planning 단계), clock distribution(CTS 단계), signal routing(routing 단계)에 사용된다. signal routing은 길이가 짧은 편이므로 낮은 layer를 사용한다(저항이 높음).