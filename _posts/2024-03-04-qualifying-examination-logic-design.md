---
title: "Qualifying Examination - Logic Design"
excerpt: "Logic Design"
date: 2024-03-04 09:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
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

{% capture solution2020-1-1 %}
A. t1, t2  
B. t2  
C. There are 2 re-convergent paths  
(1) a → and1 → t1 → or1 → z와 a → invB → abar → and2 → t2 → or1 → z  
(2) b → and2 → t2 → or1 → z와 b → invA → bbar → and1 → t1 → or1 → z  
D.  

![2020-1 1D Solution]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-1-1d-sol.drawio.svg)  

{% endcapture %}

<div class="notice--warning">{{ solution2020-1-1 | markdownify }}</div>

----------

2\. (10) Given a Boolean function $F(A, B, C) = m0 + m1 + \pmb{d3} + \pmb{d4} + m6$, answer the following.  
 A. (3) Find a minimized SoP(sum of products) of F using Karnaugh-map method. Show your process.  
 B. (2) Similar to problem 2.A, find a minimized PoS(product of sums) of F using Karnaugh-map method. Show your process.  
 C. (2) List all prime implicant(s) of F.  
 D. (3) List all essential(s) of F. In addition, draw the covering table of F, which is used in the Quine-McClusky Method.

{% capture solution2020-1-2 %}
![2020-1 2 Solution]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2020-1-2-sol.drawio.svg)  

{% endcapture %}

<div class="notice--warning">{{ solution2020-1-2 | markdownify }}</div>  

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

3\. Below is an example 3-bit counter, and its finite-state machine(FSM). "-" indicates don't cares.  

![2021-2 3a]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2021-2-3a.png){: .align-center}

Write down the minimized expression for each next state(A+, B+, and C+).  

Draw its minimized circuit diagram using D flip-flop and basic gates.  

----------

## 2022-1

1\. (50) Below is shown the input and output timing waveform of D flip-flop. Answer the followings.  

![2022-1 1]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2022-1-1.png){: .align-center}

A. (10) What is the value of Clock-to-Q propagation delay(T_dCQ)? Specify its waveform range. (Answer example: 5(time x → time x+5 or time x → time x-5))  

B. (10) What is the Clock-to-Q contamination delay (T_cCQ)? Specify its waveform range. (Answer example: 5 (time x → time x+5 or time x → time x-5))  

C. (15) If setup time (T_s) = 5, specify its waveform range. (Answer example: 5(time x → time x+5 or time x → time x-5))  

D. (15) If hold time (T_h) = 6, specify its waveform range. (Answer example: 5(time x → time x+5 or time x → time x-5))  

----------

2\. (50) Below is shown a simple sequential logic circuit. We assume that each D flip-flops has T_dCQ=7, T_cCQ=6, T_s=8, and T_h=6. Answer the followings. (Note that D_max and D_min values specified in each block indicate the longest and shortest apth delays from its input to output, repectively. We assume that wire delay is 0 and there is no clock skew.)  

![2022-1 2]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2022-1-2.png){: .align-center}

A. (25) Calculate the hold time slack on flip-flop FF 1.  

B. (25) Calculate the minimal clock period (i.e., the fastest clock frequency) the sequential circuit can use with no setup time violation.  

----------

## 2022-2

1\. (30) There are four seats A, B, C, and D as shown below. We want to design a combinational logic which receives four inputs A, B, C, D such that x(= A, B, C, or D) = 1 indicates a person sits on chair x and x = 0, otherwise, and produces output Y = 1 if there are people sitting in neighboring chairs each other. For example, out = 1 if ABCD = 0110 or 1110, and out = 0 if ABCD = 0101 or 0001. Note that A and D are regarded as neighbors.  

![2022-2 1]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2022-2-1.drawio.svg){: .align-center}

(A) (10) Draw the truth table.  

(B) (10) Draw the Karnaugh-Map.  

(C) (10) Minimize the Boolean logic  

----------

2\. (40) We want to design a Moore finite state machine (FSM) with two inputs of clock signal 'clk' and data signal 'in', and one output signal 'out' where 'in' stream is continuously coming into the input port of FSM, one per clock cycle. The FSM operates as follows: 'out'=1 whenever data level changes (i.e. the 'in' value changes form 1 to 0 or 0 to 1) and 'out' = 0, otherwise.  

(A) (15) Show the state transition diagram.  

(B) (10) Show the state encoding and state transition table,  

(C) (10) Show the Boolean equations for the next state and the output  

(D) (5) Draw the logic gates  

----------

3\. (30) Let us consider the logic circuit shown below with the following flip-flop(FF) time, combinational logic delay, and clock skew information:  

setup time t_setup = 100ps,  

hold time t_hold = 0,  

clock-to-q delay t_c2q = 0,  

The numbers shown between FFs indicate the maximum delay of the combinational logic between the FFs. Finally, 'tskew1' and 'tskew2' denote the clock skew between two FFs which is defined to be t_1 - t_2 where t_1 and t_2 indicate the clock signal arrival time to the driving FF(i.e., left one) and to the driven FF(i.e., right one), respectively.  

![2022-2 3]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2022-2-3.png){: .align-center}

(A) (20) When tskew1 = -100ps and tskew2 = 50ps, what is the minimum clock period that is possible with no timing violation on the circuit?  

(B) (10) By adjusting the value of tskew1 and tskew2, we want to find the minimum clock period that is possible to operate the circuit with no timing violation. What is the minimum clock period as well as the corresponding value of tskew1 and tskew2?  

----------

## 2023-1

1\. (60) Below is the block of a 1-bit full adder(FA). Answer the following questions.  

![2023-1 1]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-1-1.png){: .align-center}  

A. (20) With the FA block and a minimum number of other logic gates, draw the block diagram of 4-bit ripple carry adder (RCA) producing $F[3:0] = P[3:0] + Q[3:0]$ with overflow detection signal **overflow**.  

B. (20) If we want the 4-bit RCA getting from problem 1.A to perform 4-bit addtion $F[3:0] = P[3:0] + Q[3:0]$ when input signal add=1 and subtraction $F[3:0] = P[3:0] - Q[3:0]$ when input signal add = 0, what (minimal) changes are needed to the 4-bit RCA in problem 1.A?  

C. (20) If the internal structure of the FA is the one shown below, what is the critical path delay in the RCA in problem 1.A? Specify the critical path. (Assume the delay of 2-input XOR is 3, 2-input AND is 1.5 and 2-input OR is 2, and assume that all inputs are arrived at the same time(i.e., @0).)  

![2023-1 1c]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-1-1c.png){: .align-center}  

2\. (40) Below is the block of a 2:1 multiplexor(MUX). Answer the following questions.  

![2023-1 2]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-1-2.png){: .align-center}  

A. (20) Draw a logic structure of 2:1 MUX using 2-input NAND gates and inverters.  

B. (20) Draw a 4:1 MUX shown below using 2:1 MUX blocks.  

![2023-1 2B]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-1-2b.png){: .align-center}  

----------

## 2023-2

1\. (15) Write the Boolean logic equation of ***f*** in the following 4-input multiplexor(MUX).  

![2023-2 1]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-2-1.png){: .align-center}  

2\. (20) Write the Boolean logic equations of ***F_0***, ***F_1***, ***F_2***, and ***F_3*** in the following 4-output de-multiplexor (DeMUX) or decoder.  

![2023-2 2]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-2-2.png){: .align-center}  

3\. (35) The following diagram shows a part of 4x4 register file with a single write port and a single read port. Complete the logic diagram of the register file by using some copies of 4-inpu MUX and DeMUX in problem 1 and 2. (wa[1:0] and ra[1:0] indicate the 2-bit write address and 2-bit read address, respectively, and wd[3:0] and rd[3:0] indicate the 4-bit word to be written to the register file and the 4-bit word to be read out, respectively.)  

![2023-2 3]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-2-3.png){: .align-center}  

4\. (20) Draw of the waveform of ***gclk*** below, which is the signal to the flip-flop in the logic diagram in problem 3.  

![2023-2 4]({{site.baseurl}}/assets/images/2024-03-04-logic-design-2023-2-4.png){: .align-center}  

5\. (10) What is the role of the latch and AND gate in the logic diagram in problem 3?  
