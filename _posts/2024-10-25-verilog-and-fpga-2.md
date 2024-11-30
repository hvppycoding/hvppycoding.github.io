---
title: "Part2-1. Combinational Logic & Sequential Logic"
excerpt: ""
date: 2024-10-25 16:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

## Contents

- 이론
  - Combinational Logic
  - Sequential Logic
  - FSM
- 실습
  - Verilog 기본 문법 및 개념
  - Verilog Modeling
  - Testbench의 개념 및 활용
  - Task & Function

## Combinational Logic

- Combinational logic의 개념
- Gate logic의 종류 상세 설명
- Boolean Algebra
- Karnaugh Map
- Combinational logic의 대표 예시
- Additional Logic

### Overview

- What is Combinational Logic?
  - output이 gate와 input의 조합에 의해 결정되는 logic
  - time-independent logic
  - no feedback 회로
- These are basic gates which used in general
  - NOT, AND2, OR2, NAND2, NOR2, XOR2, XNOR2

### Gate Logic

#### NOT gate

![not_gate]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/not_gate.png){: .align-center}

- nmos의 gate에 1을 넣으면 source와 drain이 연결된다.
- pmos의 gate에 0을 넣으면 source와 drain이 연결된다.
- 그림에서 A에 1이 들어가면 nmos가 켜진다. OUT에는 0이 연결된다.
- 그림에서 A에 0이 들어가면 pmos가 켜진다. OUT에는 1이 연결된다.

#### NAND/NOR gate

![nand_nor_gate]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/nand_nor_gate.png){: .align-center}

- NAND의 pMOS는 병렬로 연결되어 있다. 둘 중 하나만 켜져도 출력이 1이 된다.

#### AND/OR gate

![and_or_gate]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/and_or_gate.png){: .align-center}

- AND 구현을 위해서는 NOT 연산이 추가로 필요하다. NAND가 간단하여 많이 쓰인다.

#### XOR

![xor_gate]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/xor_gate.png){: .align-center}

- A, B 값이 다를 때 1을 출력한다.
- A, B 값이 같을 때 0을 출력한다.

#### XNOR

![xnor_gate]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/xnor_gate.png){: .align-center}

#### N-input Gate Logic

- 앞에서는 2-input gate들만 살펴보았다.
- 실제 라이브러리에는 3-input, 4-input, 5-input gate도 있다.

### Boolean Algebra

- 왜 필요한가? logic expression을 최소화할 수 있다.
- $!(AB + A\bar{B})$는 사실 $\bar{A}$로 간단한 식과 동일한 동작을 한다.
- Basic expression:
  - AND: $AB$, $A\cdot B$
  - OR: $A+B$
  - NOT: $\bar{A}$
- Basic Law
  - commutative law: $A+B = B+A$, $AB = BA$
  - associative law: $(A+B)+C = A+(B+C)$, $(AB)C = A(BC)$
  - distributive law: $A(B+C) = AB+AC$, $A+BC = (A+B)(A+C)$
  - redundance law: $A=A\cdot 1=A+0$, $1=A+\bar{A}=1+A$, $0=A\bar{A}=A\cdot 0$
  - identity law: $A=A+A=AA=A\bar{B} + AB=(A+B)(A+\bar{B})$
  - De morgan's Theorem: $\overline{A+B} = \bar{A}\cdot\bar{B}$, $\overline{AB} = \bar{A}+\bar{B}$
- Practice
  - $AB + ABC + A\bar{B} + \bar{A}BC = ?$
  - $=A + BC$

### Karnaugh Map

- 시각화하여 풀기 좋다. 4-input 까지는 쉽게 만들 수 있는 방법이다.

![karnaugh_map]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/karnaugh_map.png){: .align-center}

## Sequential Logic

- Sequential logic의 개념
- Latch & Flip-flop
- Clock, Reset
- Sequential logic의 대표 예시

### Overview

- What is Sequential Logic?
  - output이 ionput 뿐 아니라 output state에 의해 결정될 수도 있다.
  - EN(Clock): output Q 값이 변할 수 있다.
  - Reset: output Q 값을 초기화한다.

### Latch 구조 및 동작

#### SR Latch

- SR NOR Latch
  - 0: 파랑
  - 1: 빨강

![sr_nor_latch]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/sr_nor_latch.png){: .align-center}

#### Gated SR Latch

- Level-sensitive

![gated_sr_latch]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/gated_sr_latch.png){: .align-center}  

#### Gated D Latch

- S, R이 동시에 1이 되지 않도록 한다.

![gated_d_latch]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/gated_d_latch.png){: .align-center}  

### D Flip-Flop의 구조 및 동작, 활용

#### Flip-Flop

- Flip-Flop
  - Latch와 다르게 Flip-Flop은 clock(enable)이 high일 때 output Q 값이 변하지 않는다.
  - rising 이나 falling edge에서 Q의 값을 바꾼다.
- D Flip-Flop
  - Verilog design에서 가장 많이 사용된다.
  - D(input) 값은 clock rising edge에 Q(output)에 반영된다.
  - 6 NAND gates로 구성된다.

![d_flip_flop]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/d_flip_flop.png){: .align-center}  

#### D Flip-Flop - Behavior

- Clock이 0인 경우

![dff_behavior_1]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_behavior_1.png){: .align-center}

- Clock이 0에서 1로 바뀐 경우

![dff_behavior_2]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_behavior_2.png){: .align-center}  

![dff_behavior_3]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_behavior_3.png){: .align-center}

![dff_behavior_4]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_behavior_4.png){: .align-center}

- Clock이 1이나 0으로 고정되었을 때 데이터가 안 바뀌는지도 중요하다.
- 왼쪽: data 신호가 기존 0에서 1로 바뀐 경우(기존과 바뀌는게 없다)
- 오른쪽: data 신호가 기존 1에서 0으로 바뀐 경우(좀 더 신호가 바뀌긴하나 Q에는 영향 없다)

![dff_behavior_5]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_behavior_5.png){: .align-center}  

- Clock이 1 → 0 → 1로 바뀌는 케이스 확인해보자

![dff_behavior_6]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_behavior_6.png){: .align-center}

#### D Flip-Flop Symbol

![dff_symbol]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_symbol.png){: .align-center}

#### D Flip-Flop Verilog coding

![dff_verilog_coding]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-2/dff_verilog_coding.png){: .align-center}

### Clock

- Toggle 해주는 신호
- 주요 Properties
  - frequency: (1/period)
  - duty cycle: clock 주기 중 신호 1이 차지하는 비율(50%가 이상적)
  - jitter: 노이즈로 인해 clock이 정확한 주기를 갖지 못한다
- Clock frequency 결정하기
  - Period > Gate delay + Setup time + Hold time + Jitter 여야 한다.
