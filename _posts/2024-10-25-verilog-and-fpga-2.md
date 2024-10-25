---
title: "Part2. 반도체 설계 기반 Verilog & FPGA"
excerpt: "Digital Logic 및 Verilog Coding 기초"
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

