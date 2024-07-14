---
title: "Back-side Clock Delivery Network Design"
excerpt: ""
date: 2024-07-15 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

참고 논문: A METHODOLOGY FOR BACK-SIDE CLOCK DELIVERY NETWORK DESIGN COMPATIBLE WITH COMMERCIAL EDA FLOWS

## Summary

- Semiconductor industry에서 back-side power delivery network(BS-PDN)은 활발히 연구되고 있으나, clock에 대한 연구는 별로 없다.
- 본 연구는 clock routing에 back-side metals을 사용하는 첫 EDA methodology
- 지능적으로 clock tree를 partition(back vs front)하여 clock과 full-chip metrics를 개선하면서 through-silicon-via 사용을 최소화한다.
- 기존의 front-side only 접근법 대비 latency 31.9%, skew 11.9%, clock-power 9.4% 개선
- 70% glitch, 14.1% wirelength, 28.9% worst negative slack 개선 달성(full-chip metrics)

## 1. Introduction

- higher performance computing 요구 증가, advanced node manufacturing cost 증가
- standard cell delay 감소, wire delay가 상대적으로 dominant 해짐
- Metal stack은 주로 Mx, My, Mz metal group으로 구성됨(아래 그림 참고)
- logic size가 shrinking하면서, Mx layer pitch 감소 → resistance 증가
- 이 상황에서 EDA 툴은 My/Mz 레이어를 이용한 signal routing을 늘리는 경향
- 그런데 My/Mz 레이어는 signal, clock, power delivery network가 공유하여 사용함
- high-performance 입장에서 ideal solution은 signal routing을 위한 공간 확보 필요
- 평균적으로 20-30%가 Clock Delivery Network(CDN)에 사용됨
- Switching frequency 증가는 power consumption 악화 → power-efficient CDN 필요
- 다른 issue는 Signal Integrity(SI)
- wire pitch가 감소 → wire간 coupling capacitance 증가
- SI 개선을 위해 Spacing, shielding 제안되었으나, routing congestion cost
- clock과 signal routing이 공유하는 레이어를 줄여 metal resource utilization 개선 가능
- substrate의 back-side에 metal layer를 이용하여 power delivery를 하는 Back-side Power Delivery Network(BS-PDN)에 영감을 받아, clock을 back-side metal layer로 전파하는 방법 제안
- 이를 Back-side-CDN(BS-CDN)이라 부르겠음

### Figure CDN

![2024-07-15-front-side-vs-back-side-clock.svg]({{site.baseurl}}/assets/images/2024-07-15-front-side-vs-back-side-clock.svg){: .align-center}

### Contributions

- back-side clock delivery를 design, simulation analysis를 통해 front-side와 비교하는 첫 연구임. industry relevance와 credibility를 위해 commercial quality full-chip design with commercial RTL 사용하였고, commercial 16nm와 28nm technology 사용, commercial EDA tools 사용함
- clock와 full-chip metric을 최적화하는 metal layer 방법론 제안하였고, commercial flow를 수정하는 방법 제안
- metal layer optimization에 대해 폭넓은 연구(front-side metal layer for signal and clock)를 통해 효과를 분석함. 또한 clock routing parameter가 clock과 full-chip metric에 미치는 영향을 sensitivity 분석함.

## 2. Background

- Clock tree는 하나의 source와 여러 sink로 구성됨
- Clock Tree Synthesis(CTS)는 최소의 pwoer dissipation으로 sink 딜레이의 균형을 맞추고자 함
- Clock net은 대개 3개 파트로 나누어짐: leaf, trunk, top
  - leaf: sink에 연결된 net
  - trunk: leaf를 제외한 모든 net
  - top: trunk net들 중 일부를 top net으로 정의함(optimization purpose)
  - 위쪽의 그림 참고
- industry standard Place-and-Route(PnR) 툴은 다양한 input parameter를 가짐
  - target skew, ID; 사용 가능한 buffer/inverter cells; fanout, transition, capacitance limits, user-defined skew-groups; Non-Default Routes(NDR)
- BS-PDN은 power distribution이 back-side metal layer에서 이루어지는 power delivery scheme이다.
- BS-PDN을 위해 nano-scale Through-Silicon-Vias(nTSV)가 사용된다.
- 이 방법론은 signal과 power delivery를 분리하여 less congested, voltage-drop resilient, high-performance design 가능하게 함
- nTSV를 사용하면 substrate thinning 현상이 관측되고, thermal issue가 발생할 수 있음
- 이로부터 영감을 받아, 동일한 컨셉을 CDN으로 확장함. 목표는 better clock quality와 lower routing congestion으로부터 high-performance low-power design을 달성하는 것.

## 3. Methodology

### 3.1. Overview of the Approach

net assignment for each metal group은 다음과 같음.

![2024-07-15-bs-cdn-net-assignment.svg]({{site.baseurl}}/assets/images/2024-07-15-bs-cdn-net-assignment.svg){: .align-center}

| Nets | Preferred routing layer range |
|------|--------------------------------|
| Signal | Mx + My Mz (Front-side) |
| Clock leaf | Mx (Front-side) |
| Clock trunk | Mb (Back-side) |
| Clock top | Mb (Back-side) |

back-side layer를 Mb 그룹으로 분류하고, clock top과 trunk net들을 이곳에 assign함.
이를 위해서 툴이 top과 trunk nets를 2개의 back-side Back-End-of-Line(BEOL) layer에 라우팅하도록 하였음.
back-side BEOL은 clock만 사용할 수 있도록 하였음. (signal net과 clock을 최대한 분리하기 위해)
leaf net은 Mx 레이어에서 만들어지도록 하였는데, 그 이유는 back-side에서 clock leaf까지 라우팅 할 시 nTSV가 너무 많아짐으로 인해 cost-effective하지 않기 때문임.

[첫 그림](#figure-cdn)(c)는 BS-CDN의 clock distribution을 나타내고 있음.
clock이 front-side port로부터 들어오고 top/trunk net일 경우 back-side로 간다.
각 clock buffer의 clock은 front-side에서 들어오며 back-side로 다시 나간다.
clock은 leaf net은 앞서 말한대로 front-side에서 라우팅된다.

이 방법론 적용 시, signal net들은 My/Mz layer 공간을 확보하여 performance 향상.
better clock quality로 timing degradation impact를 최소화하여 clock skew 감소.
또한, insertion delay가 낮아져 cross-corner variation도 줄어들 것으로 예상됨.(when multi-mode-multi-corner timing analysis with on-chip variation is considered)

또한, clock quality로 인해 power consumed by the design이 줄어들 것이다.
buffers on the substrate는 물리적으로 더 가까워 clock wirelength가 줄어들고, back-side layer는 lesser resistance를 가지므로 buffer가 덜 필요할 것이다. Signal routing도 개선되어 총 wirelength와 total buffering이 줄어들 것이다. 이는 power consumption을 줄일 것으로 예상한다.

마지막으로, clock에 영향을 받는 signal net이 줄어들어 SI 개선된다. 더 나은 parasitic으로 인해 cross-coupling capacitance가 줄어들 것이다. 설계자는 NDR을 정의하는 manual intervention을 줄일 수 있을 것이다.

아래 플로우 차트는 전체적인 Flow를 나타낸다. commercial EDA flow에 수정이 필요한 부분은 분홍색으로 하이라이트되어 있다. Logic synthesis 툴은 Synopsys, Physical design tool은 Cadence를 사용하였다.
추가적으로 EDA 벤더에서 제공하는 API를 사용해 우리의 methodology를 수행하는 스크립트를 작성하였다.
제안된 방법론을 구현하기 위한 주요한 수정사항들은 이후 섹션들에서 다룰 것이다.

![2024-07-15-back-side-clock-routing-methodology-flow.svg]({{site.baseurl}}/assets/images/2024-07-15-back-side-clock-routing-methodology-flow.svg){: .align-center}

Our proposed back-side clock routing methodology flow.
{: .custom-caption}

### 3.2. Metal Stack and Parasitics Update

