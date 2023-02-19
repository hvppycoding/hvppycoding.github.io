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
