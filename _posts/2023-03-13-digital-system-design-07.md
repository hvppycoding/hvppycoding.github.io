---
title: "Digital System Design 07"
excerpt: "HDL Coding for Combinational Logic"
date: 2023-03-13 15:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
mathjax: "true"
---

## HDL Coding for Combinational Logic

1. Combinational logic을 만들어 내는 HDL coding 구문을 파악한다.
2. 다양한 HDL 구문 의미의 차이점을 이해한다.

### Continuous Assignment

Verilog HDL(Hardwar Description Language)

```verilog
module combo(A, B, C, D);   // "combo"라는 module design
  input A, B, C;    // Inputs
  output D;   // Outputs

  // &: AND gate
  // |: OR gate
  // A, B, C 중 하나라도 값의 변화가 있을 때 아래 값을 evaluation한다.
  assign D = (A & B) | C;
  // "assign" 키워드는 "combinational logic" 구현에 사용된다.
endmodule
```

![2023-03-13-combo-module.png]({{site.baseurl}}/assets/images/2023-03-13-combo-module.png)

### Logic Synthesis

위의 구현은 AND/OR gate가 필요하여 실제 구현 시 비효율적일 수 있다(Tr이 많이 요구됨). NAND로 Logic Conversion한다. Drive의 Load가 증가할 시 Driving Strength가 모자랄 수 있다. 이를 개선하기 위해 추가적인 Buffer를 삽입할 수도 있다.  

![2023-03-13-logic-synthesis.png]({{site.baseurl}}/assets/images/2023-03-13-logic-synthesis.png)

### Gates(Structural Description)

```verilog
// gate를 직접 선언하는 방식
// 설계의 구조를 모두 알고 있어야만 한다.
// 보통은 CAD 툴을 이용하여 설계 중이던 design을 저장하는 식으로 사용함.
module synGate(f, a, b, c);
  output f;
  input a, b, c;
    // and A(a1, a, b, c)는 a, b, c가 input, a1은 output
    and A(a1, a, b, c);
    and B(a2, a, ~b, ~c);
    and C(a3, ~a, o1);
    or D(o1, b, c);
    or E(f, a1, a2, a3);
endmodule
```

아래는 위와 동일한 function module에 대해 assign 구문을을 사용한 예시이다.

```verilog
module synAssign(f, a, b, c);
  output f;
  input a, b, c;

  assign f=(a&b&c) | (a&~b&~c) | (~a&(b|c));
endmodule
```

- gate 선언 문 앞에 "#10"과 같은 문법으로 gate 딜레이를 설정 가능하고, 이는 Simulation에서 적용된다. synthesis에서는 무시된다.
- synthesis에서는 clock period 관련 timing만 만족시킨다.
- "x"는 don't care를 뜻한다.
  - `assign y = (a==b) ? 1'bx : c;`
- 비교문에 "x"가 사용될 시 synthesize 실패한다.
  - `assign y = (a === 1'bx) ? 1'bx : c;`

### Procedural Assignment

```verilog
module synCombinationalAlways(f, a, b, c);
  output f;
  input a, b, c;
  reg f;  // "f = b"와 같은 구문이 있을때 f를 reg에 등록 필요하다.

  // always block: a, b, c 중 하나라도 변화가 있으면 아래 block 1회 수행
  always @(a or b or c) 
    if (a==1)
      f = b;
    else
      f = c;
  // f = a·b + a'·c
endmodule
```

Procedure Assignment로 Sequential logic과 Combinational logic 모두 구현 가능하다. Procedure Assignment이 Combinational logic이 되는 첫 조건은 always문의 sensitivity list와 always 내부에서 사용되는 input set이 같아야 한다는 점이고, 두 번째 조건은 모든 control path에 대해 define이 되어 있어야 한다는 점이다.

### Sensitivity list / Input set이 다를 때

![2023-03-13-sensitivity-list-result.png]({{site.baseurl}}/assets/images/2023-03-13-sensitivity-list-result.png)

### Alternatives

동일한 구조를 Gate로도 구현 가능하고, MUX를 사용해서 구현도 가능하다.

### 정리하기

- Combinational logic을 만들 수 있는 coding
  - Continuous assignment
  - Gates
  - Procedural assignment
- Always block의 두 가지 개념
  - Sensitivity list and Input set
  - Control path
- If statement 사용을 통해 생성 가능한 logic
  - MUX
  - Latch
  - Flip-Flop

### 문제

![2023-03-13-problems.png]({{site.baseurl}}/assets/images/2023-03-13-problems.png)