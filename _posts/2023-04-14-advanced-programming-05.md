---
title: "Dynamic Programming - Rod Cutting"
excerpt: "Advanced Programming Methodologies 05"
date: 2023-04-14 05:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Divide-and-Conquer Method

- 문제를 겹치지 않는 sub-problem들로 쪼갠다.
- 각 sub-problem들을 recursive하게 푼다.
- 원래의 문제를 풀기 위해 각 solution들 조합한다.

## Dynamic Programming

- sub-problem들이 overlap될 때 적용할 수 있다.
- 각 sub-problem들을 한 번만 풀고, 재계산하는 것을 피할 수 있다.
- 주로 optimization problem에 적용된다.
- optimal value를 만드는 여러가지 솔루션이 있을 수도 있으나, 그 중 하나를 찾는 것이 목표이다(an optimal solution).
- optimal solution의 structure를 characterize한다.
- bottom-up으로 optimal solution의 값을 계산한다.

## Rod Cutting Problem

- $p_i$: 길이 i인 막대의 가격

Length *i*            | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10|
----------------------|---|---|---|---|---|---|---|---|---|---|
Price *P<sub>i</sub>* | 1 | 5 | 8 | 9 | 10| 17| 17| 20| 24| 30|

- 길이 n인 막대가 주어졌을때, maximum revenue $r_n$을 구하자.
- 여러가지 조합이 있을 수 있다. 막대를 k 조각으로 나누는 것이 Optimal solution일 때,
  - 각 막대 길이를 $i_1, i_2, ..., i_k$라고 하면,
  - $n = i_1 + i_2 + \cdots + i_k$
  - 그에 따른 이익은 $r_n = p_{i_1} + p_{i_2} + \cdots + p_{i_k}$가 된다.

- $r_n$의 최대 이익값은 더 작은 막대의 최대 이익을 이용해서 계산될 수 있다.
- $r_n = \max\\{ p_n, r_1 + r_{n-1}, r_2 + r_{n-2}, \cdots, r_{n-1} + r_1 \\}$

## Optimal Substructure

- 크기 n인 문제를 풀기 위해, 더 동일 종류한 더 작은 문제를 푼다.
- 한 번 자르면 두 조각을 서로 독립적인 문제로 다룰 수 있다.
- 두 조각에 대해 각각 최대 이익을 구하면 전체의 최적 솔루션을 구할 수 있다.
- 이러한 특징을 **"Optimal Substructure"**라고 한다.

## Simpler way

- 최종 Solution의 가장 왼쪽에는 항상 어떤 길이를 갖는 하나의 조각이 존재할 것이며, 첫 조각의 길이가 i이면 오른쪽에는 나머지 n-i이 존재할 것이다.
- 가장 왼쪽의 조각과 나머지로 나누는 방식으로 생각하면, 다음과 같이 간소화할 수 있다.
- $r_n = \max{p_i + r_{n-i} : 1 \le i \le n}$

## Recursive Top-Down Implementation

```text
CUT-ROD(p, n)
  if n == 0
    return 0
  q = -∞
  for i = 1 to n
    q = max(q, p[i] + CUT-ROD(p, n-i))
  return q
```

## Recurrence for Time Complexity

$$
T(n) + 1 + \sum_{j=0}^{n-1} T(j)
$$

- 위 Recurrence 식을 풀기 위해 n-1을 대입하여 뺄 수 있다.

$$
T(n) = 1 + \sum_{j=0}^{n-1} T(j) \\
T(n-1) = 1 + \sum_{j=0}^{n-2} T(j) \\
T(n) - T(n-1) = \sum_{j=0}^{n-1} T(j) - \sum_{j=0}^{n-2} T(j) = T(n-1) \\
T(n) = 2T(n - 1) \\
T(n) = T(0) \cdot 2^n = 2^n \\
$$

- 왜 이렇게 오래걸릴까? 모든 가능한 경우의 수를 고려하기 때문이다.

## What Happens for n=4

- n=4일 때 recursion tree는 아래와 같다.
- root에서 leaf로 가는 경로들은 $2^{n-1}$가지 경우의 수를 나타낸 것이다.

![230414_rod_cutting_recursion_tree.png]({{site.baseurl}}/assets/images/230414_rod_cutting_recursion_tree.png)

## Dynamic Programming for Optimal Rod Cutting

- Naive recursive solution은 똑같은 subproblem들을 반복적으로 푼다.
- 우리는 각 subproblem들을 오직 **한 번만** 풀고 정답을 저장해둘 것이다.

## Top-down Implementation with Memoization

- Recursive로 자연스럽게 푼 후, 결과를 저장하는 부분만 수정한다(대개 array나 hash table 사용).
- 먼저 이 subproblem을 푼 적이 있는지 체크한 뒤,
  - 푼 적이 있으면, 저장된 답을 리턴하고,
  - 없으면, 계산한 후 정답을 저장한다(**Memoization**).

## Bottom-up Implementation

- Subproblem을 크기 순으로 정렬한 후 작은 문제 순으로 푼다.
- 특정 Subproblem을 풀 때, 우리는 그보다 작은 모든 문제에 대한 답을 저장해있다.
- Top-down과 Bottom-up은 대부분 동일한 Asymtotic 러닝 타임을 가진다.
- 하지만 경우에 따라 Top-down 방식이 모든 Subproblem을 풀지 않아 유리할 수도 있다.
- Bottom-up은 Function call 등의 Overhead가 적어 빠를 수 있다.

## Top-down Version

```text
MEMOIZED-CUT-ROD(p, n)
  let r[0:n] be a new array
  for i = 0 to n
    r[i] = -∞
  return MEMOIZED-CUT-ROD-AUX(p, n, r)

MEMOIZED-CUT-ROD-AUX(p, n, r)
  if r[n] ≥ 0
    // already have a solution
    return r[n]
  if n == 0
    q = 0
  else
    q = -∞
    for i = 1 to n
      // i is the position of the first cut
      q = max(q, p[i] + MEMOIZED-CUT-ROD-AUX(p, n-i, r))
  r[n] = q
  return q
```

## Bottom-up Version

```text
BOTTOM-UP-CUT-ROD(p, n)
  let r[0:n] be a new array
  r[0] = 0
  // for increasing rod length j
  for j = 1 to n
    q = -∞
    // i is the position of the first cut
    for i = 1 to j
      q = max(q, p[i] + r[j - i])
    r[j] = q
  return r[n]
```

## Reconstructing a Solution

- Solution 값 뿐만 아니라, 실제 조각들의 사이즈도 알고 싶다면?
- Optimal Value 뿐만 아니라 그 때의 선택도 기록한다.

```text
EXTENDED-BOTTOM-UP-CUT-ROD(p, n)
  let r[0:n] and s[1:n] be a new array
  r[0] = 0
  // for increasing rod length j
  for j = 1 to n
    q = -∞
    // i is the position of the first cut
    for i = 1 to j
      if q < p[i] + r[j - i]
        q = p[i] + r[j - i]
        s[j] = i
    r[j] = q
  return r and s

PRINT-CUT-ROD-SOLUTION(p, n)
  (r, s) = EXTENDED-BOTTOM-UP-CUT-ROD(p, n)
  while n > 0
    print s[n]
    n = n - s[n]
```
