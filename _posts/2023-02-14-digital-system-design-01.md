---
title: "Digital System Design 01"
excerpt: "Design Flow and Basic elements"
date: 2023-02-14 17:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
---

## 1. Design Flow

### Chip과 전자기기

- Chip: 전자기기 내부에는 회로가 내장되어 작동이 되는데 그 회로가 내장되어 있는 덩어리
- Wafer: 반도체 제작에 사용하는 재료, 한 장의 Wafer에 많은 수의 동일한 die를 찍어내게 됨.  

### 칩 내부

- Chip 내부는 매우 복잡하다. 반도체 transistor와 wire로 구성되어 있음.
- 내부 구조는 설계 방식에 따라 특정 규칙을 준수해야 함.
- ASIC 설계: Application Specific Integrated IC, 최대 성능에 적합. Tr과 연결선이 놓여질 위치 제한이 없다.
- FPGA 설계: 단시간 설계에 적합. Tr과 연결선이 놓여질 위치가 한정되어 있는 설계 방식.

### Chip 내부의 지역별 회로

#### Physical design

- Placement: Logic block들이 칩 상에 위치를 잡음. floorplanning(큰 배치, ex. SRAM, IP 등) → placement(gate 수준 위치)
- Routing: 연결선을 통해 셀들이 이어져야 한다.

### Block 설계

- 설계제약 조건에 따라 (면적, 지연시간, 전력소모 등) 설계 요소들이 다름.
- 설계자의 경험에 따라 또 다른 결과를 낼 수 있음.
- 한 block의 설계 결과는 다른 block의 설계에도 영향을 미친다.
- Block 상호간의 영향을 고려한 설계가 이루어지도록 해야 함.

### Block 내부 Logic

- 한 개의 block 내부 또한 계층적으로 내려가면 gate와 flip-flop으로 이루어짐.
- 논리 수준 단계에서의 최적 설계가 이루어짐.
- 설계 제약 조건에 따라 최적의 설계를 생성함.

### HDL을 이용한 설계 표현

- 설계자가 설계를 위해 받는 입력은 HDL(Hardware Description Language)로 기술된 Spec임.
- 설계자는 이 Spec과 같은 기능을 수행하는 HW Block을 만들게 됨.
- System-level → RTL → logic-level → Physical design을 CAD 툴로 자동 수행

### 시스템 설계

- HDL로는 C++, SystemC, SystemVerilog 등이 있음.
- Simulation/Emulation을 통해 전체 성능을 평가함.
- HW/SW Co-design의 첫 단계임. HW 만들면 동작 빠르나 수정 어렵다.

### 설계 Flow

1. HW/SW partitioning: HW로 구성할 부분과 SW로 구성할 부분을 나눈다.
2. High-level Synthesis: Clock step scheduling, 한 clock에 이루어져야하는 작업이 정해진다.
3. RTL Synthesis: 앞에 나눈 한 clock에 이루어져야하는 작업들이 정해져있다. 이 작업을 최적화 한다.
4. Logic Synthesis: RTL에서 선택된 로직에서 디테일한 구조 최적화하는 과정.
5. Physical Design: Placement & Routing → GDSII
6. Masking
7. Packing

HW는 수정 시 Cost가 크다. 각 단계에 Verification 필요하다.  

### 칩 내부 Basic Components

- Devices: Transistors → Logic gates and cells → Function blocks
- Interconnects: Local signals, Global signals, Clock signals, Power/ground nets

### 정리하기

- HW/SW co-design의 첫 단계는 SYstem-level 설계이다.
- HW 설계의 대표적인 첫 단계는 RTL 설계이다.
- Logic 단계 설계는 특정 기능의 block 내부를 논리적으로 만드는 과정이다.
- Physical design은 Placement와 Routing을 수행하는 단계이다.
- 높은 설계 단계에서는 최적화(예: 전력 소모, 회로 지연시간)의 효과를 많이 내지만 정확성 면에서는 낮다.

## 2. Basic Elements

### 주제

- 다양한 logic element의 내부 구조
- 각 logic element의 작동 원리 이해

### Transistors

- 로직을 이루는 기본 단위
- NMOS(Negative), PMOS(Positive)

| Type | G = 1 | G = 0 |     |
|------|:-------:|:-------:|-----|
| PMOS | 전류 차단 | 전류 연결 | Good for VDD |
| NMOS | 전류 연결 | 전류 차단 | Good for VSS |

CMOS = PMOS + NMOS  

### 기본 Logic element들

- NOT
- NAND
- NOR

### Standard Cell Library(=Technology Library)

- 기본 logic gate의 구조(레이아웃)을 미리 설계한 것
- 대개 같은 height를 갖는다.
- 다양한 standard cell들의 집합으로 이루어진다.
- Library는 일반적으로 ASIC vendor나 Library group으로부터 공급받는다.
- 실제로 제공되는 정보는 실제 레이아웃이 아니라 Timing, Power, Area 같은 정보들이 설계자에게 제공된다.

### MUX/DEMUX

- Multiplexer

### Tri-State Gate

- OE = 0일 때 output은 Float(Z)
- OE = 1일 때 output은 input과 동일

### Implementing MUX using Tri-state gates

- Tri-state gate들을 이용하여 MUX를 만들 수 있다.

### 정리하기

- Transistor는 PMOS와 NMOS의 두 종류가 있으며, CMOS 회로는 PMOS와 NMOS를 대칭적으로 연결해서 안정적인 작동이 되도록 한다.
- 기본적인 Gate로는 NOT, NAND, NOT이 있으며, 다른 Gate들은 기본적인 Gate들을 엮어 만들게 된다.
- MUX와 DEMUX는 wire routing을 위해 널리 사용된다. MUX의 경우는 또한 FPGA의 logic 구현을 위한 LUT(lookup table)로도 널리 사용된다.
- 그 외 널리 사용되는 Gate들로 XOR, Tri-state gate 등이 있다.
