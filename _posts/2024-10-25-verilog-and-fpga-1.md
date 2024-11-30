---
title: "Part1. Verilog 소개 및 Concept"
excerpt: ""
date: 2024-10-25 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

## 수업목표

- Verilog가 어떻게 반도체가 되는지
- Verilog로 어떻게 반도체를 만드는지
- 이론부터 실전까지
- PART1: Verilog 소개 및 Concept
- PART2: Digital Logic 및 Verilog Coding 기초
- PART3: SoC에서 자주 사용되는 Protocol 이해 및 실습
- PART4: Computer Architecture 및 실습
- PART5: Xilinx FPGA 이해 및 실습, HW편
- PART6: Xilinx FPGA 이해 및 실습, SW편

## Contents

### 이론

- 반도체 field 소개 및 Verilog 주요 사용처
- Verilog로 묘사하는 것
- Verilog와 다른 설계와의 비교

### 실습

- Free tool 소개

## 반도체 field 소개 및 Verilog 주요 사용처

![semiconductor-ecosystem]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-1/semiconductor-ecosystem.png){: .align-center}

- IDM(Integrated Device Manufacturer): 제품 기획부터 생산까지 모두 하는 회사
- Chipless: 설계된 코드만 제공하는 회사. 칩을 만드는 것을 주관하진 않는다.
- Fabless: Fab이 없지만 제품에 대한 책임은 진다. Foundry와 Packaging은 외주화.
- Design house: Design을 Foundry에 넘기는 역할(검토 수행)
- Foundry: 전공정(Foundry) +  후공정(Packaging)
- OSAT: 후공정만 하는 회사
- Fabless → Design house → Foundry → Fabless

**어느 회사에서 Verilog를 많이 쓸까?** Chipless/Fabless/IDM에서 많이 쓰며, Design house에서는 검토를 위해 알아야한다. 제품별로 구분하면, Memory 회사 보다는 System 반도체 회사에서 주력으로 사용한다.
{: .notice--warning}

## Verilog로 묘사하고자 하는 것

### Hierarchical Language

![hierarchical_language]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-1/hierarchical_language.png){: .align-center}

- Module: 베릴로그에서의 설계 단위
- 예를 들어, Top moudule에서 내부의 module의 연결 정보와 동작 정보를 정의해야한다.

### Cycle-based results

- Clock, Cycle
  - 대부분의 Verilog는 clock을 가지고 있고, 이에 따라 동작한다.
- Parallel behavior
  - 여러 line들이 동시에 동작할 수 있다.

이런 특성으로 인해 디버깅 시 파형을 확인하는 경우가 많다.

## Verilog와 다른 설계 방법과의 비교

### Comparison with C function

- C function
  - Function으로 간 후 다시 나온다(Return).
- Verilog Module
  - 회로를 Hierarchical하게 구현한 것
  - Return과 같은 Concept이 없다.

### What is Difference?

- C/C++
  - Language for SW
  - 각 라인이 순차적으로 동작한다.
  - 수행할 행위를 나타낸다(e.g.`z=a+b`는 덧셈을 수행하라는 것).
- Verilog
  - Language for HW
  - 각 Cycle에 각 라인이 Process되어야 한다.
  - 구현되어야하는 로직을 나타낸다(e.g. `z=a+b`는 덧셈 로직을 구현하라는 것).

### What will you design?

- Analog: 속도, integrity 크기를 매우 최적화해야할 때 사용한다.
- Digital: 복잡한 동작을 구현할 때 사용한다.

![analog_vs_digital]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-1/analog_vs_digital.png){: .align-center}

## Free tool 소개

### What do you need?

- Simulator and Debugger are essential
- Synopsys/Cadence/Siemens(Mentor) EDA
  - 비싸다.
- Free tools!
  - Icarus Verilog
  - edaplayground.com

본 강의에서는 edaplayground.com을 주로 사용할 것이다. 구글 연동으로 로그인이 가능하다.

![edaplayground]({{site.baseurl}}/assets/images/2024-10-25-verilog-and-fpga-1/edaplayground.png){: .align-center}