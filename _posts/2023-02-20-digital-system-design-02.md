---
title: "Digital System Design 02"
excerpt: "Logic Algebra and Logic Expression"
date: 2023-02-20 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
mathjax: "true"
---

## 1. Boolean Algebra based Simplification

- Boolean Algebra에 관한 axioms과 theorems을 이해한다.
- Logic simplification을 위해 Boolean Algebra가 logic에 적용되는 과정을 학습한다.

### Combinational Logic

- Combinational Logic의 정의: output이 현재의 input에 의해서 결정되는 디지털 시스템
- Memoryless: 과거 sequence는 결과에 영향을 미치지 않는다.

- Combination logic을 표현하는 방법들
  - Boolean algebra expression - 오늘 다룰 주제
  - Wired up logic gates(schematic)
  - Truth table
  - Hardware description language

### Boolean algebra

- Mathematical foundation of digital systems
- Boolean operation 순서: Complement(\`) → AND(·) → OR(+)
- 괄호로 순서 변경할 수 있다.
- 예1) $\bar{A}\cdot{B}+{C} = ((\bar{A})\cdot{B})+C$
- 예2) $\bar{A}+{B}\cdot{C} = (\bar{A})+(B\cdot{C})$

### Boolean algebra axioms

1. B contains at least two elements: a, b
2. Closure
   - $a + b$ is in B
   - $a \cdot b$ is in B
3. Commutativity
   - $a + b = b + a$
   - $a \cdot b = b \cdot a$
4. Associativity
   - $a + (b + c) = (a + b) + c$
   - $a \cdot (b \cdot c) = (a \cdot b) \cdot c$
5. Identity
   - $a + 0 = a$
   - $a \cdot 1 = a$
6. Distributivity
   - $a + (b \cdot c) = (a + b) \cdot (a + c)$
   - $a \cdot (b + c) = (a \cdot b) + (a \cdot c)$
7. Complementarity
   - $a + \bar{a} = 1$
   - $a \cdot \bar{a} = 0$

### Dutality vs DeMorgan

1. Duality는 theorem에 대한 theorem
   - Boolean equation에 적용됨
   - 예를 들어 $Y=X \cdot Y + Y$ 임을 증명하였으면, 이 식의 Dual인 $Y=(X + Y) \cdot Y$도 옳음
2. DeMorgan
   - Boolean expression에 적용됨
   - $f=\overline{a\bar{b}+c}$ $=\overline{a\bar{b}} \cdot \bar{c}$ $=(\bar{a}+b)\cdot \bar{c}$

### 정리하기

- Boolean algebra simplification은 manual보다는 컴퓨터의 도움을 받아 이루어진다.
- 널리 적용되는 theorem으로는 Uniting과 Consensus theorems이 있다.
- DeMorgan's theorem은 Boolean function에 적용된다.
- Duality theorem은 Boolean equation에 적용된다.
- Two-level과 Multi-level logic 생성에 모두 적용된다.

## 2. Two Level Logic

1. Two level logic의 대표적인 표현 방법을 학습한다.
2. Two level logic의 여러 구현 방법과 변환되는 원리를 이해한다.

### Minimizing the Number of Gates and Wires

- 하나의 function에 대해 여러가지 구현이 가능하다(2레벨, 멀티레벨, XOR 게이트 등)
- 우리는 Sum of Product(1단계는 곱연산, 2단계는 합연산으로 구성) 위주로 다룰 것
- Canonical Forms(출발점)
  - Boolean Expression의 표준 형태
  - 두 가지의 form이 있음(sum-of-products(SOP), product-of-sums(POS))

### Sum-of-products(SOP) canonical form

다음과 같은 Truth table을 예로 들자.  

| A | B | C | F | F' |
|---|---|---|---|----|
| 0 | 0 | 0 | 0 | 1 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 0 | 1 |
| 0 | 1 | 1 | 1 | 0 |
| 1 | 0 | 0 | 0 | 1 |
| 1 | 0 | 1 | 1 | 0 |
| 1 | 1 | 0 | 1 | 0 |
| 1 | 1 | 1 | 1 | 0 |

먼저 1이 되는 경우들을 써보면  
F = 001 011 101 110 111  
이를 식으로 나타낼 수 있다.  
F = A'B'C + A'BC + AB'C + ABC' + ABC  
이를 Canonical form이라고 한다.  
반대로 F'는 다음과 같다.  
F' = A'B'C' + A'BC' + AB'C'  

SOP 형태에서 논리식을 간단하게하는 과정을 통해 minimal form을 얻을 수 있다.

### Product-of-sums(POS) canonical form

위와 동일한 Truth Table에서 F에 대한 식의 POS를 구하기 위해선 F=0인 부분을 사용해야 한다(maxterm).
F = 000 010 100  
F = (A + B + C)(A + B' + C)(A' + B + C)  

### POS ↔ SOP

- F'의 SOP를 구한다.
  - F' = A'B'C' + A'BC' + AB'C'
- 드 모르간 적용하여 F의 POS 형태 구할 수 있다.
  - (F')' = (A'B'C' + A'BC' + AB'C')'
  - F = (A+B+C)(A+B'+C)(A'+B+C)

반대로 POS 형태로부터 SOP를 구할 수도 있다.  

### Four altenative two-level implementations of F

동일한 Function에 대해 다른 형태로 구현할 수 있다.  

![2023-02-26-two-level-implementations.png]({{site.baseurl}}/assets/images/2023-02-26-two-level-implementations.png)

### Waveforms for the four alternatives

앞의 네 가지 구현에 대해 glitch를 제외하면 동일한 function을 갖는다.  

![2023-02-26-two-level-waveforms.png]({{site.baseurl}}/assets/images/2023-02-26-two-level-waveforms.png)

### Incompletely specified functions

- Don't care output의 경우 'X'로 표기한다. minimization 시 이를 활용할 수 있다.
- 하나의 output에 대해 해당 output이 1이 되어야하는 input set(on-set), 0이 되어야하는 input set(off-set), 그리고 don't care set이 존재한다.

### Multi Level Logic

2-level logic 대비 게이트와 wire 수를 줄일 수 있으나, logic의 단계 수가 증가하여 delay가 늘어날 수 있다.  

- Z = ADF + AEF + BDF + BEF + CDF + CEF + G
  - 6개의 3-input AND 게이트와 1개의 7-input OR 게이트 필요
- Z = (A+B+C)(D+E)F + G
  - 1개의 3-input OR 게이트와 2개의 2-input OR 게이트, 1개의 3-input AND 게이트 필요