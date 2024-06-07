---
title: "Chapter 1. Introduction"
excerpt: "VLSI Physical Design"
date: 2024-06-04 09:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

### 교재

VLSI Physical Design: From Graph Partitioning to Timing Closure, Andrew B. Kahng, Jens Lienig, Igor L. Markov, Jin Hu

### Electronic Design Automation (EDA)

무어의 법칙 - 동일 면적에 들어가는 트랜지스터의 수가 18개월마다 2배씩 증가한다.
설계 요소가 2배가 되면 동일한 방식으로 설계 시 2배의 시간이 든다.
EDA 도구도 설계 효율화를 위해 발전하고 있다.

### EDA History

- ~1965: Manual design
- ~1975: Layout editors
- ~1985: Logic synthesis(버클리) → Synopsys, Cadence
- ~1990: EDA 황금기, 산업체 투자, 많은 최적화 기법들이 EDA에 적용됨
- ~2000: Physical design → Manufacturing 관련 고려 필요
- ~현재: Design for Manufacturability(DFM), Optical Proximity Correction(OPC) 등 미세공정에 대한 고려 필요, IP 재사용

### VLSI Design Flow

1. System Specification
2. Architecture Design
3. Functional Design and Logic Design: Gate-level design
4. Circuit Design: 특정 부분을 transistor-level로 설계(Optional), 이까지 단계에서 Netlist 정보 산출됨
5. Physical Design
  - Partitioning: Netlist가 너무 클 경우 Partitioning을 통해 나눈다.
  - Chip Planning: Partitioning된 부분을 Die에 어떻게 배치할지 결정
  - Placement: Partition 내부의 각 Cell 배치
  - Clock Tree Synthesis: Clock Network 동시화 트리
  - Signal Routing: Net을 연결
  - Timing Closure: Clock period 제약 조건을 만족시키는지 확인
6. Pyshical Verification and Signoff
  - Voltage Drop(IR Drop) 고려
7. Fabrication
  - Die
8. Packaging and Testing
9.  Chip

### VLSI Design Styles

- Layout editor: 아날로그 회로 설계 시 사용, 전체의 약 10% 정도 차지
- Common digital cells: Transistor를 직접 에디팅하지 않고, 미리 만들어진 Cell을 사용(AND, OR, ...) → 동일한 Row height를 가지며, Row-based ASIC 설계 수행
- Over the cell(OTC): Cell 위로 시그널 라우팅
- Layout with macro cells: Macro block 배치(standard cell 배치와 라우팅 시 blockage로 작용)
- Field programmable gate arrays(FPGAs): Manufacturing 없이 동작 프로그래밍 가능한 회로, Design 빠르게 구현할 수 있으며 프로토타이핑에 적합하다.

### Layout Layers and Design Rules

- 설계할 때 공정을 고려해서 디자인을 해야한다.

![Inverter]({{site.baseurl}}/assets/images/2024-06-04-inverter.png){: .align-center}

- Metal 1과 2 사이에는 Via가 존재한다.

#### Design Rules

- Size rules: segment의 길이가 일정 길이 이상이어야 한다(minimum length rule). minimum width rule도 존재한다. 
- Seperation rules: 두 segment 사이의 거리가 일정 거리 이상이어야 한다.
- Overlap rules: Overlap이 있다면 일정 길이 이상이어야 한다.

![2024-06-04-design-rules.png]({{site.baseurl}}/assets/images/2024-06-04-design-rules.png){: .align-center}

### Physical Design Optimizations

- Technology constraints: 공정 기술에 따라 제약이 달라진다.
- Electrical constraints: 전기적 특성
- Geometry constraints: Standard cell 위치 제약, Routing 방향(Horizontal, Vertical layer)

### Algorithms and Complexity

- Runtime complexity: big-O notation
- Example: Exhaustively Enumerating All Placement Possibilities
  - Given: *n* cells
  - Task: find a single-row placement of *n* celles with minimum total wirelength by using exhaustive enumeration
  - Solution: The solution space has *n!* placement options
- 많은 문제들이 NP-hard이다. → Heuristic algorithm 사용

#### Heuristic Algorithms

- Deterministic: 알고리즘에 의해 확정적으로 결과가 나오며, repeatable하다.
- Stochastic: Randomness를 사용하여 결과가 나오며, repeatable하지 않다. e.g. Simulated Annealing

- Heuristic algorithm 일반적 접근법
  - Constructive: incomplete solution에 component들을 채워가며 complete solution을 얻는 방식
  - Iterative: complete solution으로 시작하여 반복적으로 현재의 solution을 개선하는 방식

![2024-06-04-heuristic-algorithms.png]({{site.baseurl}}/assets/images/2024-06-04-heuristic-algorithms.png){: .align-center}

### Graph Theory Terminology

- Graph: Edge가 2개의 vertex를 연결하는 자료구조
- Hypergraph: Circuit의 Net은 Hyperedge 모델링 가능(3개 이상의 vertex를 연결)

### Common EDA Terminology

![2024-06-04-netlist.png]({{site.baseurl}}/assets/images/2024-06-04-netlist.png){: .align-center}