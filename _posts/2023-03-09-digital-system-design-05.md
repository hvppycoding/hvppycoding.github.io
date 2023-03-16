---
title: "Digital System Design 05"
excerpt: "LUT based Design"
date: 2023-03-09 02:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
mathjax: "true"
---

## LUT based Design

1. LUT의 의미와 LUT 기능을 가지는 logic component들을 알아본다.
2. Logic을 LUT를 사용해서 구현하는 절차를 이해한다.

### LUT(Look-Up Table)

Input → LUT → Output  

### ROM의 구조

ROM은 두 개의 파트(decoder + memory array)로 구성된다. decoder에 의해 address line에 따라 하나의 line이 선택된다.  

![2023-03-09-rom.png]({{site.baseurl}}/assets/images/2023-03-09-rom.png)

n-address가 주어질 때 $2^n - 1$ 개의 output line이 존재한다. 1이 연결되면 NMOS가 Conduct처럼 동작하여 세로 선의 Voltage가 0가 된다(보라색). NMOS가 달려있지 않은 line은 Voltage가 1이다.

![2023-03-09-rom2.png]({{site.baseurl}}/assets/images/2023-03-09-rom2.png)

초기 ROM에는 모든 Cell에 Transistor가 만들어져있다. 특정 line에 1을 write하기 위해서는 해당 Tr을 끊어주는 방식으로 프로그래밍을 하여야 한다.  

- Combinational Logic으로 주어졌을 때 ROM으로 구현하기: 모든 Input 조합에 대해 Truth table을 만든 후 프로그래밍하면 된다.

### MUX(Multiplexor)

LUT를 구현하는 방법은 2가지(ROM, MUX)가 있다. MUX는 Select 입력에 따라 Output에 연결될 Input line이 결정된다. 이를 이용하여 Add 회로 1개를 Mux+Demux를 이용하여 공유할 수 있다.  
MUX를 이용해서 다음과 같이 Logic을 구현할 수 있다.  

![2023-03-09-mux.png]({{site.baseurl}}/assets/images/2023-03-09-mux.png)

C와 F의 관계를 이용하여 아래와 같이 MUX 사이즈를 줄일 수도 있다.  

![2023-03-09-mux2.png]({{site.baseurl}}/assets/images/2023-03-09-mux2.png)

### FPGA

### 정리하기

- LUT 크기는 input size에 따라 결정되며, logic의 복잡도와는 무관하다.
- Cell 기반 설계에 비해서 LUT 기반 설계의 유리한 점은 설계 시간을 단축하는데 있다.
- LUT를 구현할 수 있는 HW Components로는 ROM, MUX이다.
- MUX의 주 사용 용도로는 wire routing, LUT로 logic 구현 두가지가 있다.
- FPGA 내부를 구성하는 요소 중 핵심 2가지 구성 요소는 LUT(Combination logic), FF(memory/storage)이다.
