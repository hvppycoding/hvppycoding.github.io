---
title: "Digital System Design 08"
excerpt: "Latch and Flip-flop"
date: 2023-03-13 16:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
mathjax: "true"
---

## Latch and Flip-flop

1. Latch의 구조와 장단점 및 작동 원리를 이해한다.
2. Flip-flop의 구조와 Latch와의 다른 작동 원리를 파악한다.

### Sequential Logic

Latch나 Flip-flop이 디지털 시스템에 들어가면 그 시스템을 Sequential Logic이라고 부른다(더 이상 Combinational Logic이 아니다). 예를 들어 도어락 시스템을 생각해보자. 비밀번호가 몇 번째 자리까지 입력되었는지 "기억"해야한다.

![2023-03-13-sequential-logic.png]({{site.baseurl}}/assets/images/2023-03-13-sequential-logic.png)

### R-S Latch

- R=1, S=0일 때 Q = 0
- R=0, S=1일 때 Q = 1
- R=0, S=0일 때 이전 상태를 Memory
- 직전이 R=1, S=1이었으면 R=0, S=0 시 unstable해진다. R=1, S=1 사용 안하는 것으로..

![2023-03-14-rs-latch.png]({{site.baseurl}}/assets/images/2023-03-14-rs-latch.png)

### State Transistion Diagram

### Gated or Level-sensitive Latch

Enable 신호가 On일 때(High level일 때)만 동작하도록 개선.  

![2023-03-14-level-sensitive-latch.png]({{site.baseurl}}/assets/images/2023-03-14-level-sensitive-latch.png)

### Combining Latches

- 단순히 Latch 두 개를 연결하면 하나의 Cycle 안에 모두 통과될 수도 있다.

### Master-Slave Filp-flop

- Flip-flop은 Edge Sensitive 동작을 한다. Latch를 2개 붙였기 때문에 공간 많이 차지하나, 타이밍 맞추기에 유리하다.  

### 1s Catching Problem

- Hold 상태(R=0, S=0)일 때 glitch 발생 시 전파되는 문제가 있다.  

### D Flip-flop

R=0, S=0인 경우를 사용하지 않도록 D 신호로 묶어서 사용한다.  

### Timing Definition

- setup time: Clock 전에 data가 일정 시간 전에 도착해야 한다.  
- hold time: Clock 이후 data가 일정 시간 유지되어야 한다.
- propagation delay: 신호가 통과되는데 걸리는 시간

### 정리하기

- RS latch는 11의 입력은 사용하지 않는다.
- Latch와 flip-flop의 근본적인 차이는 Latch는 Level-sensitive이고 Flip-flop은 Edge-sensitive이다.
- Master-slave flip-flop은 Gated latch의 Transparency 문제를 해결한다.
- D Flip-flop은 Master-slave flip-flop의 1s catching 문제를 해결한다.
- Flip-flop에 연관된 timing 요소 3가지는 setup time, hold time, propagation delay이다.

### 문제

1. Master-slave flip-flop은 총 몇 개의 transistor로 구성되나? 약 32개
2. Positive edge-triggered flip-flop과 Negative edge-triggered flip-flop은 같은 수의 transistor가 사용된다. True
3. Flip-flop의 setup time, hold time, propagation delay 값들 간에는 서로 연관성이 없고 상수로 주어진다. True(계산 시 편리하기 때문, 실제로는 연관성이 있을 수도 있으나 모델링 어려움)
4. Flip-flop은 전력 소모를 많이 하는 요소이다. 이유는?
   1. Clock Speed가 빨라 Switching이 많다.
   2. Clock 신호 전달을 위한 Buffer에서 소모하는 전력이 많다.
