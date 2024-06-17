---
title: "Chapter 6. Detailed Routing"
excerpt: "VLSI Physical Design"
date: 2024-06-17 06:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

![2024-06-17-detailed-routing-flow.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-flow.svg){: .align-center}

![2024-06-17-detailed-routing-in-routing-flow.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-in-routing-flow.svg){: .align-center}

### Detailed Routing

- Detailed routing의 목적은 signal net의 route segments를 특정한 routing track, via, metal layer에 assign하는 것이다. 이 때 global route 결과에 최대한 부합하도록 한다.
- global routing과 유사
  - connection을 위해 physical wires를 사용
  - Estimating the wire resistance and capacitance, 설계가 timing requirements를 만족하는지 확인
    - $5\micron$ 이하의 길이는 linear delay model로 근사할 수 있고, 그 이상은 Elmore delay model로 근사
- routing regions 내에서 Detailed routing techniques 적용됨
  - [channels](#channel-routing-algorithms), [switchboxes](#switchbox-routing), and [global routing cells](#over-the-cell-routing-algorithms)
- Detailed routers는 manufacturing rules과 manufacturing faults를 고려해야 한다.[Modern Challenges in Detailed Routing](#modern-challenges-in-detailed-routing)

- Detailed Routing Stages
  - Assign routing tracks
  - Perform entire routing, 100% 라우팅 되도록 한다.
  - Search and repair – physical design rules 준수하도록 고친다(locally).
  - Perform optimizations, e.g. redundant vias (reduce resistivity, better yield)

### Terminology

![2024-06-17-detailed-routing-terminology.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-terminology.svg){: .align-center}

![2024-06-17-detailed-routing-terminology2.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-terminology2.svg){: .align-center}

![2024-06-17-detailed-routing-terminology3.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-terminology3.svg){: .align-center}

- **Horizonal Constraint**
  - 가정: horizontal routing에 1 layer만 쓰인다.

![2024-06-17-detailed-routing-horizontal-constraint.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-horizontal-constraint.svg){: .align-center}

![2024-06-17-detailed-routing-vertical-constraint.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-vertical-constraint.svg){: .align-center}

### Horizontal and Vertical Constraint Graphs

#### Horizontal Constraint Graphs

![2024-06-17-detailed-routing-horizontal-constraint-graphs.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-horizontal-constraint-graphs.svg){: .align-center}

- `S(col)`: column `col`을 지나가는 net 집합

![2024-06-17-detailed-routing-horizontal-constraint-graphs2.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-horizontal-constraint-graphs2.svg){: .align-center}

- $S(c)$의 크기가 5이므로 최소 5개 track이 필요하다.

![2024-06-17-detailed-routing-graphs-representation.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-graphs-representation.svg){: .align-center}

#### Vertical Constraint Graphs

![2024-06-17-detailed-routing-vertical-constraint-graphs.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-vertical-constraint-graphs.svg){: .align-center}

- 같은 column에 있는 net 사이에 directed edge를 추가한다. 예를 들어 두 번째 column에서 B가 C의 위에 있으므로 B에서 C로 edge를 추가한다.
- 그려진 Graph에서 Cycle이 없어야 한다. Cycle이 있으면 Cyclic confict가 발생하며, 추가적인 Column이 필요하다.

![2024-06-17-detailed-routing-cyclic-conflict.svg]({{site.baseurl}}/assets/images/2024-06-17-detailed-routing-cyclic-conflict.svg){: .align-center}

### Channel Routing Algorithms

#### Left-Edge Algorithm

- VCG(Vertical Constraint Graph)와 zone representation을 기반으로 각 track의 usage를 최대화한다.
  - VCG: track에 할당하는 net의 순서
  - Zone representation: 같은 track을 공유할 수 있는 net 찾기 위함
- 각 net은 하나의 horizontal segment만 사용한다.

```cpp
// Input: channel routing instance CR
// Output: track assignments for each net

curr_track = 1        // start with topmost track
nets_unassigned = Netlist
while (nets_unassigned != Ø)  // while nets still unassigned
    VCG = VCG(CR)             // generate VCG and zone
    ZR = ZONE_REP(CR)         // representation
    SORT(nets_unassigned,start column)  // find left-to-right ordering
                                        //  of all unassigned nets
    for (i =1 to |nets_unassigned|)
          curr_net = nets_unassigned[i]
          if (PARENTS(curr_net) == Ø &&     // if curr_net has no parent
          (TRY_ASSIGN(curr_net,curr_track)) // and does not cause
                                            // conflicts on curr_track,
          ASSIGN(curr_net,curr_track)       // assign curr­_net
          REMOVE(nets_unassigned,curr_net)
    curr_track = curr_track + 1    // consider next track 
```

![2024-06-17-left-edge-algorithm1.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm1.svg){: .align-center}

- zone representation은 horizontal constraint graph 도출 중 그렸던 것과 유사하다.

![2024-06-17-left-edge-algorithm2.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm2.svg){: .align-center}

- VCG에서 parent가 없으면서 서로 conflict가 없는 net들을 찾아 track에 할당한다.
- 할당 후 net들을 지운다.

![2024-06-17-left-edge-algorithm3.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm3.svg){: .align-center}

- 이를 반복한다.

![2024-06-17-left-edge-algorithm4.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm4.svg){: .align-center}

![2024-06-17-left-edge-algorithm5.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm5.svg){: .align-center}

![2024-06-17-left-edge-algorithm6.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm6.svg){: .align-center}

![2024-06-17-left-edge-algorithm7.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm7.svg){: .align-center}

...

![2024-06-17-left-edge-algorithm8.svg]({{site.baseurl}}/assets/images/2024-06-17-left-edge-algorithm8.svg){: .align-center}

#### Dogleg Routing

- net-splitting을 통해 Left-edge algorithm을 개선한다.
- 장점:
  - VCG의 conflict를 완화한다.
  - track의 숫자를 줄일 수도 있다.

![2024-06-17-dogleg.svg]({{site.baseurl}}/assets/images/2024-06-17-dogleg.svg){: .align-center}

p-pin net을 (p-1)개의 horizontal segments로 나눈다. net splitting은 column이 net pin을 포함하는 경우에 일어난다. net splitting 수행 후 left-edge algorithm을 따른다.

![2024-06-17-net-splitting.svg]({{site.baseurl}}/assets/images/2024-06-17-net-splitting.svg){: .align-center}

### Switchbox Routing

![2024-06-17-switch-box.svg]({{site.baseurl}}/assets/images/2024-06-17-switch-box.svg){: .align-center}

- 네 방향의 pin connection을 수행해야 한다. 각 방향의 pin vector가 주어진다.

![2024-06-17-net-splitting-flow.svg]({{site.baseurl}}/assets/images/2024-06-17-net-splitting-flow.svg){: .align-center}

### Over-the-Cell Routing Algorithms

- Standard cell들이 routing channel 없이 back-to-back으로 배치된다.
- Metal layer는 gcell로 이루어진 coarse routing grid로 표현된다.
- gcell 단위로 global routing된 후 detailed routing이 이루어진다.
- Standard cell로 방해되지 않는 부분들은 routing에 사용된다.

### Modern Challenges in Detailed Routing

- high-performance design에 사용하기 위한 Metal layer width 조합 찾는 것이 중요
- Detailed routing이 어려워지고 있다.
  - 서로 다른 width의 wire를 연결하는 via는 작은 wire pitch layer에서 추가적인 resource가 필요하다.
  - lithography technique에서 각 layer마다 routing 방향을 엄격하게 지키도록 요구한다.
- Semiconductor manufacturing yield는 detailed routing의 주요 관심 사항이다.
  - Redundant vias, wire segments as backups(via doubling, non-tree routing)
  - Manufacturability constraints(design rules) 제약이 강해짐
  - Forbidden pitch rules prohibit routing wires at certain distances apart, but allows smaller or greater spacings  
- Detailed router는 manufacturing rules과 manufacturing faults에 미치는 영향을 고려해야 한다.
  - Via defects: via doubling during or after detailed routing
  - Interconnect defects: add redundant wires
  - Antenna-induced defects: detailed routers limit the ratio of metal to gate area on each metal layer

![2024-06-17-antenna-effect.svg]({{site.baseurl}}/assets/images/2024-06-17-antenna-effect.svg){: .align-center}

(b)에서 metal에 charge가 되어 poly에 문제가 생긴다.

![2024-06-17-antenna-effect-fix.svg]({{site.baseurl}}/assets/images/2024-06-17-antenna-effect-fix.svg){: .align-center}