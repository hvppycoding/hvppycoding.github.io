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
- Field programmable gate arrays(FPGAs): Manufacturing 없이 동작 프로그래밍 가능한 회로

### Layout Layers and Design Rules

### Physical Design Optimizations

### Algorithms and Complexity

### Graph Theory Terminology

### Common EDA Terminology