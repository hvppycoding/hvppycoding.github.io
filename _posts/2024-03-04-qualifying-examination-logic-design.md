---
title: "Qualifying Examination - Logic Design"
excerpt: "Logic Design"
date: 2024-03-04 09:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - SNU
classes: wide
mathjax: "true"
---

## 2020-1

1\. (5) Answer the followings for the logic structure.

![2020-1 1]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-1-1.drawio.svg){: .align-center}

 A. (1) What's the fin-in(s) of gate or1?  
 B. (1) What's the fan-out of gate and2?  
 C. (1) Specify re-convergent paths if exist?  
 D. (2) Convert the logic structure into the structure with 2-input NOR and NOT gates.  

----------

2\. (10) Given a Boolean function $F(A, B, C) = m0 + m1 + \pmb{d3} + \pmb{d4} + m6$, answer the following.  
 A. (3) Find a minimized SoP(sum of products) of F using Karnaugh-map method. Show your process.  
 B. (2) Similar to problem 2.A, find a minimized PoS(product of sums) of F using Karnaugh-map method. Show your process.  
 C. (2) List all prime implicant(s) of F.  
 D. (3) List all essential(s) of F. In addition, draw the covering table of F, which is used in the Quine-McClusky Method.  

----------

3\. (12) Answer the following.  
 A. (4) Draw a Mealy State Transition Diagram(STD) for detecting input pattern 101 for every 3-bit inputs(i.e., no-overlap). You don't need to minimize the number of states.  
 B. (4) Minimize the STD obtained in problem 3.A by applying **Implication Chart method**.  
 C. (4) Implement the STD obtained in problem 3.B using **a shift REGISTER and a COUNTER** together with a few discrete gates such as AND, OR, NOT. You should use the number of discrete gate(s) minimally.  

----------

4\. (13) Answer the following. (A, B, C, D, E, F, G, and H are all 4-bit numbers.)  
 A. (1) What's the 2s complement of 001101?  
 B. (4) Draw a **4-bit** ripple carry adder (RCA) using FAs as basic elements. The RCA should support the computation of not only $F = A + B$ but also $F = A - B$. Include overflow/underflow detection logic in your RCA design.  
 C. (4) We want to implement $G = A - B - C$ by using two RCAs designed in problem 4.B. Draw the implementation.  
 D. (4) Suppose 2-input MUX delay=4, the SUM output delay of FA=5, the CARRY output delay of FA=6, INV gate delay=1. 2-input AND = 2-input OR gate delay = 2.5. What is the longest delay from inputs A, B, C to output G of the implementation you produced in problem 4.C. **Specify the longest path and its delay value**.  

----------

5\. (10) A sequential logic design **DESIGN_TEST** is shown below. (We assume all net delays are 0.) Answer the followings.  

![2020-1 5]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-1-5.drawio.svg){: .align-center}

 A. (5) If there is or are hold time violation(s) when x0 = x1 = 0, fix the violation(s). Justify your answer.  
 B. (5) What is the minimal value of clock period which does not violate all setup time constrains in DESIGN_TEST when x0 = x1 = 0. Justify your answer.  

----------

## 2020-2

1\. Answer the following questions.  

a. (7) Given the following PLA(Programmable Logic Array), represent each of the four outputs F1, F2, F3, and F4 as a minimized sum-of-products expression.  

![2020-2 1]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-2-1.png){: .align-center}

b. (10) We like to implement the four outputs in problem 1.a with PAL(Programmable Array Logic) as shown below. Notice that each output is connected to 2-input OR gate. You cannot add new wires or gates. Just mark x's on the intersection points of wires which have to be connected. The spare OR gate can be used freely if it is necessary.  

![2020-2 2]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-2-2.png){: .align-center}

----------

2\. (15) Implement the following state diagram into a synchronous Mealy FSM using output-oriented encoding and draw your implemented FSM. Try to implement with the smallest number of gates.  

![2020-2 3]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-2-3.png){: .align-center}

----------

3\. Answer the following questions  

a. (8) Draw a state transition table from the following state transition diagram for Moore machine.  

![2020-2 4]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-2-4.png){: .align-center}

b. (10) Using the implication chart method, reduce the state machine to an equivalent machine with the minimal number of states. Tell which states are merged to form new states, and represent the new state machine by illustrating a state transition diagram.  

----------

## 2021-1

1\. Drive a simplified **Sum-of-Products(SOP)** of the following truth table using Karnaugh map. and draw the gate-level structure of the simplified SOP using 2-input AND, 2-input OR, and INVERTER gates only.  

| ABC | F |
|-----|---|
| 000 | 1 |
| 001 | 1 |
| 010 | 0 |
| 011 | 1 |
| 100 | 1 |
| 101 | 0 |
| 110 | 0 |
| 111 | 1 |

----------

2\. Implement the function F specified in the truth table introduced in problem 1 using a **single** 4:1 multiplexor component.(S1 and S0 are two select ports of the multiplexor and should be respectively connected to B and C; it is allowed to use any number of INVERTER gates for the implementation, if needed.)  

{% capture solution2021-1-2 %}
**Solution Example**:
* You can include lists
* and even fenced code blocks:

```html
<html>
  <body>Some body.<body>
</html>
```

![2020-2 4]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-2-4.png){: .align-center}

{% endcapture %}

<div class="notice--warning">{{ solution2021-1-2 | markdownify }}</div>

----------

3\. Reimplement the 4:1 multiplexor-based design produced in problem 2 by using only **2:1 multiplexor components**. (Reimplemented design should use the smallest number of 2:1 multiplexors; it is allowed to use any number of INVERTER gates for the implementation, if needed.)  

## 2021-2

1\. (30) You have a boolean function F as below:  
$F(a, b, c, d) = \sum{m(1, 2, 3, 5, 7) + d(10, 11, 12, 13, 14, 15)}$  
// d means "don't care"

(1) Draw F's karnaumap  

| AB ＼ CD | 00 | 01 | 11 | 10 |
|----------|----|----|----|----|
| 00 | | | | |
| 01 | | | | |
| 11 | | | | |
| 10 | | | | |

(2) Write F's prime implicants and essential prim implicants  
Prime implicants:  
Essential prime implicants:  

----------

2\. (40)  

(a) RS latch is shown below with its input-output truth table.  

![2021-2 2a]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2021-2-2a.png){: .align-center}

Draw its gate diagram using two inverters and two 2-input NAND gates.  

(b) Gated/level-sensitive latch uses clock-based input(CLK) to solve the problem of RS latch.  

* What is the proble of RS latch solved by a gated/level sensitive latch?
* Draw the gate diagram of gated/level-sensitive latch using one clock and four 2-input NAND gates.

(c) Below is a master-slave flip-flop using two RS-latch blocks with extra gates.  

![2021-2 2c]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2021-2-2c.png){: .align-center}

Master-slave flip-flop sovles a key problem of the gated/level-sensitive latch. What is the problem? How dos this master-slave flip-flop solve the problem?

(d) Draw a D flip-flop which solves a critical problem of a master-slave flip-flop(above). What is the problem? How does this D flip-flop solve the problem?  

![2021-2 2d]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2021-2-2d.png){: .align-center}

----------
