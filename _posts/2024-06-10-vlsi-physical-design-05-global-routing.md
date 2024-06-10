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

### Optimization Goals
### Representations of Routing Regions
### The Global Routing Flow
### Single-Net Routing
#### Rectilinear Routing
#### Global Routing in a Connectivity Graph
#### Finding Shortest Paths with Dijkstra’s Algorithm
#### Finding Shortest Paths with A* Search
###	Full-Netlist Routing
#### Routing by Integer Linear Programming
#### Rip-Up and Reroute (RRR)
###	Modern Global Routing	
#### Pattern Routing
#### Negotiated-Congestion Routing 
