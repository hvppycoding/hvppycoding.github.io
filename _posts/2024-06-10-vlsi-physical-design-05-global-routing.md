---
title: "Chapter 5. Global Routing"
excerpt: "VLSI Physical Design"
date: 2024-06-10 06:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

![2024-06-10-global-routing-flow.svg]({{site.baseurl}}/assets/images/2024-06-10-global-routing-flow.svg){: .align-center}

### Introduction

placement, netlist, technology information이 주어졌을 때,

- necessary wiring 결정
  - e.g., net topologies and specific routing segments, to connect these cells
- respecting constraints 만족
  - e.g., design rules and routing resource capacities, and
- routing objectives 최적화
  - e.g., minimizing total wirelength and maximizing timing slack.

#### Terminology

- Net: 같은 electric potential을 가지는 pin의 집합
- Netlist: Set of all nets.
- Congestion: Where the shortest routes of several nets are incompatible because they traverse the same tracks. 영역별로 사용할 수 있는 metal track에 제한이 있기 때문.
- Fixed-die routing: Chip outline이 고정되어 있고, routing resources도 한정됨.
- Variable-die routing: Routing이 불가능할 경우, die를 증가시켜 New routing tracks 추가될 수 있음.

#### General Routing Problem

![2024-06-10-general-routing-problem.svg]({{site.baseurl}}/assets/images/2024-06-10-general-routing-problem.svg){: .align-center}

- Technology information: 레이어 정보(Horizontal, Vertical), Width, Spacing 등

Routing 결과 예시

![2024-06-10-general-routing-problem-result.svg]({{site.baseurl}}/assets/images/2024-06-10-general-routing-problem-result.svg){: .align-center}

#### Routing Introduction

![2024-06-10-routing-introduction.svg]({{site.baseurl}}/assets/images/2024-06-10-routing-introduction.svg){: .align-center}

1. Global routing: 대략적인 routing 수행. Metal layer assignment는 Global routing에서 주로 이루어진다.(구현에 따라 다르며, Detailed routing에서도 가능함)
2. Detailed routing: Global routing 결과 바탕으로 세부적인 track assignment 수행
3. Timing-Driven Routing: critical net에 대한 net topology 최적화 및 resource allocation
4. Large Single-Net Routing: Power, Ground, 저항이 낮은 상위 metal layer에 주로 배치
5. Geometric Techniques: Clock routing은 die의 모든 flip-flop에 clock signal을 공급해야 한다. 이 때 clock signal이 모든 flip-flop에 최대한 동시에 도달하도록 routing 수행 필요하다. → 길이 balance를 맞춰야 한다.

#### Global Routing Introduction

- Wire segments가 시험적으로 칩 레이아웃에 배치됨
- Chip area를 coarse routing grid로 표현
- Available routing resources가 edges with capacities in a grid graph로 표현됨
- Nets are assigned to these routing resources

#### Terminology and Definitions

- Routing Track: Horizontal wiring path
- Routing Column: Vertical wiring path
- Routing Region: Region that contains routing tracks or columns
- Uniform Routing Region: 동일한 간격의 horizontal/vertical grid
- Non-uniform Routing Region: Horizontal and vertical boundaries that are aligned to external pin connections or macro-cell boundaries resulting in routing regions that have differing sizes
- Channel: Pin 연결을 위한 routing region(과거의 Two-layer routing 방식)

![2024-06-10-routing-channel.svg]({{site.baseurl}}/assets/images/2024-06-10-routing-channel.svg){: .align-center}

- Capacity: 사용 가능한 routing track 혹은 column의 수
  - Single-layer routing에서는 capacity는 channel height $h$를 pitch $d_{pitch}$로 나눈 값이다.
  - Multilayer routing에서 capacity $\sigma$는 모든 레이어의 capacity를 합한 값이다.
    - $\sigma(Layers) = \sum_{layer \in Layers} \left \lceil \frac{h}{d_{pitch}} \right \rceil$

![2024-06-10-capacity.svg]({{site.baseurl}}/assets/images/2024-06-10-capacity.svg){: .align-center}

- Switch box: 수평 및 수직 채널이 교차하는 지점

![2024-06-10-switch-box.svg]({{site.baseurl}}/assets/images/2024-06-10-switch-box.svg){: .align-center}

![2024-06-10-t-junction.svg]({{site.baseurl}}/assets/images/2024-06-10-t-junction.svg){: .align-center}

![2024-06-10-gcells.svg]({{site.baseurl}}/assets/images/2024-06-10-gcells.svg){: .align-center}

### Optimization Goals

Global routing seeks to

- 주어진 placement가 routable한 지 판단
- coarse routing 결정(for all nets within available routing regions)

Considers goals such as

- minimizing total wirelength, and
- reducing signal delays on critical nets

### Representations of Routing Regions

- Routing regions을 효율적인 data structure로 표현해야 함
- Routing context is captured using a graph, where
  - nodes represent routing regions and
  - edges represent adjoining regions
- Capacities are associated with both edges and nodes to represent available routing resources

![2024-06-10-grid-graph-model.svg]({{site.baseurl}}/assets/images/2024-06-10-grid-graph-model.svg){: .align-center}

### The Global Routing Flow

1. routing regions 정의(Region definition)
  - Layout area가 routing region들로 나누어짐
  - Nets can also be routed over standard cells
  - Regions, capacities, connection이 graph로 표현됨
2. Mapping nets to the routing regions (Region assignment)
  - Each net of the design is assigned to one or several routing regions to connect all of its pins
  - Routing capacity, timing and congestion affect mapping
3. Assigning crosspoints along the edges of the routing regions (Midway routing)
  - Routes are assigned to fixed locations or crosspoints along the edges of the routing regions
  - Enables scaling of global and detailed routing
  - Optional한 과정으로, detailed routing에서도 가능, global routing에서 고정 가능한 부분을 고정하여 detailed routing을 빠르게 수행할 수 있음

### Single-Net Routing

#### Rectilinear Routing

![2024-06-10-rectilinear-routing.svg]({{site.baseurl}}/assets/images/2024-06-10-rectilinear-routing.svg){: .align-center}

RMST에서 pin-to-pin connection이 되어야 함(공유 없이). 런타임이 빠르다. WL = 11  
RSMT에서는 Steiner point를 추가하여 공유 가능. WL = 9  
좋은 Steiner point 구하기가 쉽지 않다.

RSMT의 특징
- p-pin net의 RSMT는 0~p-2개의 Steiner point를 가질 수 있다.
- terminal pin의 degree는 1, 2, 3, 4 중 하나이다.
- Steiner point의 degree는 3, 4 중 하나이다.
- RSMT는 항상 net의 Minimum bounding box 내에 존재한다.
- total edge length $L_{RSMT}$는 적어도 minimum bounding box의 half perimeter
  - $L_{RSMT} \geq \frac{L_{MBB}}{2}$

![2024-06-10-rmst-to-rsmt.svg]({{site.baseurl}}/assets/images/2024-06-10-rmst-to-rsmt.svg){: .align-center}

쉽게 만들 수 있는 RMST로부터 RSMT를 만들어보자.

##### Hanan grid

- The Hanan grid consists of the lines x = xp, y = yp that pass through the location (xp,yp) of each terminal pin p 
- The Hanan grid contains at most (n2-n) candidate Steiner points (n = number of pins), thereby greatly reducing the solution space for finding an RSMT

![2024-06-10-hanan-grid.svg]({{site.baseurl}}/assets/images/2024-06-10-hanan-grid.svg){: .align-center}

##### Sequential Steiner Tree Heuristic

1. rectilinear distance로 가장 가까운 pin pair를 찾고, minimum bounding box(MBB)를 구한다.
2. MBB 위의 어떤 point $p_{MBB}$와 후보 pin 중 가장 가까운 pin $p_C$로 point pair $(p_{MBB}, p_C)$를 구한다.
3. $p_{MBB}$와 $p_C$의 MBB를 만든다.
4. $p_{MBB}$가 놓여있는 L-shape를 T에 추가한다. 만약 $p_{MBB}$가 pin이라면 아무 L-shape를 추가한다.
5. Step 2로 돌아가서 반복한다.

![2024-06-10-heuristic-rsmt.svg]({{site.baseurl}}/assets/images/2024-06-10-heuristic-rsmt.svg){: .align-center}

#### Global Routing in a Connectivity Graph

![2024-06-10-connectivity-graph.svg]({{site.baseurl}}/assets/images/2024-06-10-connectivity-graph.svg){: .align-center}

##### Algorithm Overview

1. Define routing regions
2. Define connectivity graph
3. Determine net ordering: Net 배치할 순서 결정(성능에 영향 크다)
4. Assign tracks for all pin connections in Netlist: 각 pin 연결을 위한 기본적인 track 차지하게 둔다.
5. Consider each net
  - Free corresponding tracks for net's pins
  - Decompose net into two-pin subnets
  - Find shortest path for subnet connectivity graph
  - If no shortest path exists, do not route, otherwise, assign subnet to the nodes of shortest path and update routing capacities
5. If there are unrouted nets, goto Step 5, otherwise END

#### Finding Shortest Paths with Dijkstra’s Algorithm

생략

#### Finding Shortest Paths with A* Search

Cost function에 estimated distance를 추가한다.

Bidirectional A* Search: Source와 Target에서 양방향으로 탐색한다.

### Full-Netlist Routing

- **Simultaneously:** e.g., integer linear programming
- **Sequentially:** e.g., one net at a time
  - When certain nets cause resource contention or overflow for routing edges, sequential routing requires multiple iterations: **rip-up and reroute**

#### Routing by Integer Linear Programming

- Linear Program(LP)
  - Constraints
  - Optimization function
- Integer Linear Program(ILP)
  - Linear program where every variable can only assume integer values
  - NP-hard
- Three inputs:
  - W × H routing grid G
  - Routing edge capacities
  - Netlist
- Two sets of variables
  - k Boolean variables $x_{net1}$, $x_{net2}$, … , $x_{netk}$, each of which serves as an indicator for one of k specific paths or route options, for each net $net \in Netlist$
  - k real variables $w_{net1}$, $w_{net2}$, … , $w_{netk}$, each of which represents a net weight for a specific route option for $net \in Netlist$
- Two types of constraints
  - Each net must select a single route (mutual exclusion)
  - Number of routes assigned to each edge (total usage) cannot exceed its capacity  

![2024-06-10-routing-by-ilp-formulation.svg]({{site.baseurl}}/assets/images/2024-06-10-routing-by-ilp-formulation.svg){: .align-center}

![2024-06-10-routing-by-ilp-example-1.svg]({{site.baseurl}}/assets/images/2024-06-10-routing-by-ilp-example-1.svg){: .align-center}

![2024-06-10-routing-by-ilp-example-2.svg]({{site.baseurl}}/assets/images/2024-06-10-routing-by-ilp-example-2.svg){: .align-center}

![2024-06-10-routing-by-ilp-example-3.svg]({{site.baseurl}}/assets/images/2024-06-10-routing-by-ilp-example-3.svg){: .align-center}

#### Rip-Up and Reroute (RRR)

![2024-06-10-rip-up-and-reroute.svg]({{site.baseurl}}/assets/images/2024-06-10-rip-up-and-reroute.svg){: .align-center}

임시로 violation을 허용하고, 모든 net을 라우팅 한 후, 몇몇 net을 rip-up한 후 다른 방식으로 reroute하는 것을 반복 수행하여 라우팅하는 방식이다.

### Modern Global Routing

General flow for modern global routers, where each router uses a unique set of optimizations:

![2024-06-10-general-modern-global-routers-flow.svg]({{site.baseurl}}/assets/images/2024-06-10-general-modern-global-routers-flow.svg){: .align-center}

#### Pattern Routing

- route 패턴들을 몇 가지 미리 정의해두고, 이를 이용하는 방식이다. 
- pattern routing에서 주로 사용하는 패턴은 L-shape, Z-shape, U-shape 등이 있다. (Via를 적게 사용하는 패턴을 우선 시)

![2024-06-10-pattern-routing.svg]({{site.baseurl}}/assets/images/2024-06-10-pattern-routing.svg){: .align-center}

#### Negotiated-Congestion Routing

- 각각의 edge e에 대해 코스트 $cost(e)$가 할당된다. 이 cost에는 edge e에 대한 demand가 반영된다.
- 만약 net의 segment가 e를 지나면 cost(e)만큼 코스트가 든다.
- net의 cost는 net이 차지한 모든 edge의 cost의 합이다.
  - $cost(net) = \sum_{e \in net} cost(e)$
- edge congestion $\varphi(e)$에 따라 $cost(e)$가 증가한다. edge congestion은 해당 edge를 지나가는 net을 edge capacity로 나눈 값이다.
  - $\varphi(e) = \frac{demand(e)}{capacity(e)}$
- 높은 $cost(e)$는 net이 e를 사용하는 것을 피하고 다른 덜 사용된 edge를 찾도록 유도한다.
- Iterative routing approaches(Dijkstra’s algorithm, A* search)를 사용하여 cost를 최소화하는 route를 찾을 수 있다.

상용 툴의 global routing 알고리즘은 black box로 자세히 알 수는 없다.
{: .notice--warning}

### Summary

- **Global routing**
  - Input: netlist, placement, obstacles + (usually) routing grid
  - Partitions the routing region (chip or block) into global routing cells (gcells)
  - Considers the locations of cells within a region as identical
  - Plans routes as sequences of gcells
  - Minimizes total length of routes and, possibly, routed congestion
  - May fail if routing resources are insufficient
    - Variable-die can expand the routing area, so can't usually fail
    - Fixed-die is more common today (cannot resize a block in a larger chip)
  - Interpreting failures in global routing
    - Failure with many violations => must restructure the netlist and/or redo global placement
    - Failure with few violations => detailed routing may be able to fix the problems

- **Detailed routing**
  - Input: netlist, placement, obstacles, global routes (on a routing grid), routing tracks, design rules
  - Seeks to implement each global route as a sequence of track segments
  - Includes layer assignment (unless that is performed during global routing)
  - Minimizes total length of routes, subject to design rules

- **Timing-driven routing**
  - Minimizes circuit delay by optimizing timing-critical nets
  - Usually needs to trade off route length and congestion against timing
  - Both global and detailed routing can be timing-driven

- **Large-Net Routing**
  - Nets with many pins can be so complex that routing a single net warrants dedicated algorithms
  - Steiner tree construction
    - Minimum wirelength, extensions for obstacle-avoidance
    - Nonuniform routing costs to model congestion
  - Large signal nets are routed as part of global routing and then split into smaller segments processed during detailed routing

- **Clock Tree Routing / Power Routing**
  - Performed before global routing to avoid competition for resources occupied by signal nets

- Usually ~50% of the nets are two-pin nets, ~25% have three pins, ~12.5% have four, etc.
  - Two-pin nets can be routed as L-shapes or using maze search (in a connectivity graph of the routing regions)
  - Three-pin nets usually have 0 or 1 branching point
  - Larger nets are more difficult to handle

- **Pattern routing**
  - For each net, considers only a small number of shapes (L, Z, U, T, E)
  - Very fast, but misses many opportunities
  - Good for initial routing, sometimes is sufficient

- **Routing pin-to-pin connections**
  - Breadth-first-search (when costs are uniform)
  - Dijkstra's algorithm (non-uniform costs)
  - A*-search (non-uniform costs and/or using additional distance information)

- Minimum Spanning Trees and Steiner Minimal Trees in the rectilinear topology (RMSTs and RSMTs)
  - RMSTs can be constructed in near-linear time
  - Constructing RSMTs is NP-hard, but feasible in practice
- Each edge of an RMST or RSMT can be considered a pin-to-pin connection and routed accordingly
- Routing congestion introduces non-uniform costs, complicates the construction of minimal trees (which is why A*-search still must be used)
- For nets with <10 pins, RSMTs can be found using look-up tables (FLUTE) very quickly

- **Routing by Integer Linear Programming (ILP)**
  - Capture the route of each net by 0-1 variables, form equations constraining those variables
  - The objective function can represent total route length
  - Solve the equations while minimizing the objective function (ILP software)
  - Usually a convenient but slow technique, may not scale to largest netlists (can be extended by area partitioning)

- **Rip-up and Re-route (RRR)**
  - Processes one net at a time, usually by A*-search and Steiner-tree heuristics
  - Allows temporary overlaps between nets
  - When every net is routed (with overlaps), it removes (rips up) those with overlaps and routes them again with penalty for overlaps
  - This process may not finish, but often does, else use a time-out

- Both ILP-based routing and RRR can be applied in global and detailed routing
  - ILP-based routing is usually preferable for small, difficult-to-route regions
  - RRR is much faster when routing is easy

- Initial routes are constructed quickly by pattern routing and the FLUTE package for Steiner tree construction - very fast
  - Several iterations based on modified pattern routing to avoid congestion - also very fast
    - Sometimes completes all routes without violations
    - If violations remain, they are limited to a few congested spots
- The main part of the router is based on a variant of RRR called Negotiated-Congestion Routing (NCR)
  - Several proposed alternatives are not competitive
- NCR maintains "history" in terms of which regions attracted too many nets
- NCR increases routing cost according to the historical popularity of the regions
  - The nets with alternative routes are forced to take those routes
  - The nets that do not have good alternatives remain unchanged
  - Speed of increase controls tradeoff between runtime and route quality
