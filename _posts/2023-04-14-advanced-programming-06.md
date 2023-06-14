---
title: "Greedy Algorithms"
excerpt: "Advanced Programming Methodologies 06"
date: 2023-04-14 07:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

greedy algorithm 설계, 분석을 다룰 것이다.  

## Greedy Algorithms

- greedy algorithm은 현재 상황에서 가장 좋아보이는 선택을 한다.
- locally optimal choice를 하며 이 선택이 globally optimal solution이 되기를 바란다.
- 이 장에서는 greedy algorithm이 optimal solution에 도달하는 문제들을 살폅볼 것이다.

## Activity Selection problem

- 서로 겹쳐서는 안 되는 activity들이 주어졌을 때, compatible한 activity들의 최대 집합을 찾는 문제이다.
- n개의 activity set이 주어진다. $S = \lbrace a_1, a_2, \cdots, a_n \rbrace$
- 각 activity $a_i$는 시작 시간 $s_i$와 종료 시간 $f_i$를 가진다. $0 \le s_i < f_i < \infty$
- $a_i$가 선택되었다면 $a_i$는 $[s_i, f_i)$ 동안의 시간을 차지한다.
- $[s_i, f_i)$와 $[s_j, f_j)$가 겹치지 않는다면 $a_i$와 $a_j$는 compatible하다.
- activity들이 finish time을 기준으로 오름차순으로 정렬되어 있다고 가정한다. $f_1 \le f_2 \le \cdots \le f_n$

## Starting Dynamic Programming

- 어떤 subproblem을 optimal solution에 사용할지 결정하기 위해 여러 선택을 고려해야 한다.

## Optimal Substructure

- $S_{ij}$를 $a_i$ finish 후에 시작하고 $a_j$의 start 전에 끝나는 activity들의 집합이라고 하자.
- $S_{ij}$의 mutually compatible activity들의 최대 집합을 찾고 싶다고 가정하자.
- 그리고 그러한 maximum set $A_{ij}$가 어떤 activity $a_k$를 포함한다고 가정하자.
- $a_k$를 optimal solution에 포함시키면서 두 개의 subproblem이 남는다.
  - $S_{ik}$의 mutually compatible activity들의 최대 집합 찾기
  - $S_{kj}$의 mutually compatible activity들의 최대 집합 찾기
- $A_{ik} = A_{ij} \cap S_{ik}$, $A_{kj} = A_{ij} \cap S_{kj}$라고 하자.
  - $A_{ik}$는 $A_{ij}$ 중에서 $a_k$ start 전에 끝나는 activity들을 담고 있다.
  - $A_{kj}$는 $A_{ij}$ 중에서 $a_k$ finish 후에 시작하는 activity들을 담고 있다.
- $A_{ij} = A_{ik} \cup \lbrace a_k \rbrace \cup A_{kj}$이다.
  - $A_{ij}$는 $\vert A_{ij} \vert = \vert A_{ik} \vert + 1 + \vert A_{kj} \vert$ 개의 activity들로 이루어져 있다.
- optimal solution $A_{ij}$는 두 개의 subproblem $S_{ik}$와 $S_{kj}$의 optimal solution들을 포함한다.
  - $A^\prime_{kj}$가 $S_{kj}$의 mutually compatible activity이며, $\vert A^\prime_{kj} \vert > \vert A_{kj} \vert$라고 하자. $A^\prime_{kj}$ 솔루션을 $S_{ij}$의 subproblem을 푸는데 사용할 수 있다.
  - 이는 $\vert A_{ik} \vert + 1 + \vert A^\prime_{kj} \vert > \vert A_{ik} \vert + 1 + \vert A_{kj} \vert$개의 set을 구성한다. 이는 $A_{ij}$가 optimal solution이라는 가정에 모순된다.
  - 이는 $S_{ik}$에 대해서도 동일하게 적용된다.

## Dynamic Programming

- $S_{ij} = \lbrace a_k \in S \ \vert\  f_i \le s_k < f_k \le s_j \rbrace$
- fictitious activity $a_0 : f_0 = 0$과 $a_{n+1}: s_{n+1} = \infty$를 추가하고, $S = S_{0, n+1}$이라고 하자.
- $i \ge j$이면 $S_{ij}=\emptyset$이다.
- $A_{ij}$를 $S_{ij}$의 optimal solution이고 $a_k \in A_{ij}$라고 하자.
  - $A_{ij} = A_{ik} \cup \lbrace a_k \rbrace \cup A_{kj}$이다.
- $A_{0, n+1}$는 전체 문제의 optimal solution이다.

### Recursive Definition

- $c[i, j]$를 $S_{ij}$의 optimal solution의 크기라고 하자.
- $c[i, j] = c[i, k] + c[k, j] + 1$

$$
\begin{aligned}
c[i, j] = \begin{cases}
    0 &\text{if } S_{ij} = \emptyset\\
    \max_{a_k \in S_{ij}} \lbrace c[i, j] + c[k, j] + 1 \rbrace &\text{if } S_{ij} \ne \emptyset\\
       \end{cases}
\end{aligned}
$$

## Making the Greedy Choice

- $S_k = {a_i \in S \ \vert\ s_i \ge f_k}$라고 하자. ($a_k$가 finish된 이후에 시작하는 activity들의 set)
- 만약 activity $a_1$을 greedy하게 선택한다면 $S_1$가 풀어야할 남은 subproblem이 된다.
- Optimal substructure는 다음을 의미한다.
  - 만약 $a_1$이 optimal solution에 포함된다면
  - optimal solution은 activity $a_1$과 subproblem $S_1$의 optimal solution으로 이루어진다.
- Proof
  - $A_1$이 $S_1$의 optimal solution이라고 하자. 그리고 optimal solution $O$가 존재하며 $\lvert \lbrace a_1 \rbrace \cup A_1 \rvert < \lvert O \rvert$라고 하자.
  - $O$에서 $(O - \lbrace a_1 \rbrace)$를 $A_1$으로 대체하면 $\lvert O \rvert \le \lvert \lbrace a_1 \rbrace \cup A_1 \rvert$가 성립한다.
  - $\lvert \lbrace a_1 \rbrace \cup A_1 \rvert < \lvert \lbrace a_1 \rbrace \cup A_1 \rvert$가 되어 모순이다.

## Greedy Solution

- Theorem 16.1
  - 비어있지 않은 subproblem $S_k$에 대해 $a_m$가 $S_k$의 가장 빠른 finish time을 가진 activity라고 하자.
  - 그러면 $a_m$은 $S_k$의 어떤 mutually compatible maximum-size subset(즉, optimal solution)에 포함된다.
- Proof
  - $A_k$가 $S_k$의 mutually compatible maximum-size subset이라고 하자. $a_j$가 $A_k$의 가장 빠른 finish time을 가진 activity라고 하자.
  - 만약 $a_j = a_m$이라면 $a_m$이 solution에 포함되어 있는 상태이므로 증명이 끝난다.
  - $a_j \ne a_m$이라면 $A^\prime_k = A_k - \lbrace a_j \rbrace \cup \lbrace a_m \rbrace$와 같은 $A^\prime_k$를 생각해보자.
  - $A_k$는 disjoint activity들로 구성되어 있기 때문에 $A^\prime_k$의 activity들도 disjoint하다. 왜냐하면 $a_j$는 $A_k$의 가장 빠른 finish time을 가진 activity인데, $f_m \le f_j$이기 때문이다.
  - $\lvert A^\prime_k \rvert = \lvert A_k \rvert$이므로 $A^\prime_k$도 $S_k$의 mutually compatible maximum-size subset이라고 결론내릴 수 있다.

