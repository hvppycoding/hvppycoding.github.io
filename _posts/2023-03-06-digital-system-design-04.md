---
title: "Digital System Design 04"
excerpt: "Logic Conversion and Cell based Design"
date: 2023-02-26 02:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
mathjax: "true"
---

## 1. Logic Conversion

- Target technology에 맞는 logic을 생성하는 과정을 익힌다.

### AND/OR Conversion to NAND/NAND Networks

AND/OR 기준으로 Optimize를 했지만 결과적으로 NAND/NAND로 변경해주어야 한다.

![2023-03-06-nandnand-conversion.png]({{site.baseurl}}/assets/images/2023-03-06-nandnand-conversion.png)

### AND/OR Conversion to NOR/NOR Networks

![2023-03-06-nornor-conversion.png]({{site.baseurl}}/assets/images/2023-03-06-nornor-conversion.png)

### 정리하기

- POS, SOP는 Techonology independent optimization에 해당한다.
- NAND/NOR로 conversion은 Technology dependent optimization에 해당된다.
- Bubble insertion 알고리즘의 목적은 NAND/NOR로의 conversion을 위해 최소개의 bubble(inverter)를 회로에 추가하는 것이다.
- Network conversion을 위해 널리 사용되는 Boolean Algebra theorem은 De Morgan 법칙이다.

### 문제

AOI gate: AND + OR + INVERTER  

2x2 AOI gate: 앞의 2는 AND Gate가 2개라는 뜻, 뒤의 2는 각 AND Gate의 입력이 2-input AND Gate라는 뜻임.  

## 2. Cell based Design

- 여러 Target logic cell의 구조를 이해한다.
- Target logic cell이 활용되는 과정을 이해한다.

### Standard cells

- Off-chip standard cells
  - 과거에는 패키지 단위로 보드에 장착
- On-chip standard cells
  - Chip 안에서 규격화된 Cell
- Application-Specific Integrated Circuit(ASIC)
  - (Sea of)Gate array based: 동일 Gate를 많이 만들어두고 Metal로 연결
  - Standard cell based

### Technology Metrics

- Gate delay
  - The time delay from an input change to an output change
  - t(90% output final value) - t(90% of input final value)
- Degree of integration
  - The chip area and number of chip packages required to implement a given function in a technology
- Power dissipation
  - Power consumed in performing their logic functions
- Noise margin
  - 회로가 정상적으로 동작하는 voltage noise의 크기
- Fan-out
  - The number of gate inputs to which a given gate output can be connected because of electrical limitations
  - Fan-out이 크면 Drive 약해진다.
- Driving capability
  - The speed of communications between packages components or cells in a chip
- Configurability

### On-chip Cell Layout

- 특정한 기본 Logic gate의 구조(Layout)을 미리 설계한 것
- 각 Cell은 대개 같은 standard height을 갖는다.
- Standard cell library는 다양한 standard cell들의 집합으로 이루어진다.
- Library는 일반적으로 ASCI vendor나 Library group으로부터 공급되어진다.

### Internals in a Standard cell

- **Standard cell**
  - 3 parts(Power/Signal track/Ground)
  - Connection: M1(Y) - V1 - M2 - V1 - M1(A)
  - 3 signal track(unused=free metal segment)

### Cell Based Design: Physical Design Example

2 Interdependent problems @physical design state

- Case #1: Fail(Area/Power가 너무 많은 경우를 극복하자)
- Case #2: Fail(Hold time violation을 극복하자)

### Case #1: Fail(Area/Power)

![2023-03-06-case1-example.png]({{site.baseurl}}/assets/images/2023-03-06-case1-example.png)

![2023-03-08-case1-solution.png]({{site.baseurl}}/assets/images/2023-03-08-case1-solution.png)

Logic Design 후 Physical Design에서 이러한 Violation들을 해결한다. Gate의 Area 같은 정보는 어떻게 알 수 있을까? Cell의 Library 명세에 있다.  

### Case #2: Fail(Hold time violation)

- Solution 1: Delay를 위한 Buffer 추가
- Solution 2: Detour routing

![2023-03-08-case2-solution.png]({{site.baseurl}}/assets/images/2023-03-08-case2-solution.png)

### 정리하기

- Cell designd은 target technology에서 규정한 design rules를 만족하는 design이 되어야 한다.
- Cell의 종류로는 NAND2, NOR2, AOI22, FF, MUX 등이 있다.
- Cell design에 고려 사항으로는 area, timing, power, pin accessibility 등을 들 수 있다.
- Logic synthesis와 physical design에서 사용한 cells들에 맞게 optimization이 이루어진다.

