---
title: "Qualifying Examination"
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

