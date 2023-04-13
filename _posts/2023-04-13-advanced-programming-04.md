---
title: "Median and Order Statistics"
excerpt: "Advanced Programming Methodologies 04"
date: 2023-04-13 22:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Outline

- 고유한 n개 숫자들 중 i번째 숫자 찾아내기

## i-th Smallest Number Selection Problem

- 편의를 위해 데이터가 distinct number set이라고 가정하자.
- Selection problem:
  - Input: set of n(distinct) numbers and a number i with $i \le i \le n$
  - Output: The element $x \in A$ that is larger than exactly i-1 other elements of A
- 단순히 O(n log n) 정렬 후 찾아낼 수도 있다.

## A Naive Method

- Size i의 array를 할당 후 insertion sort와 유사하게 진행

## Using Max-Heap

- Size i의 max-heap 할당 후 각 원소에 대해 heap max element보다 크다면 무시, 작다면 root에 새로운 원소를 넣은 뒤 `HEAPIFY(root)` 수행

## A Divide and Conque Method

- Quicksort와 유사한 방식으로 `RANDOMIZE-SELECT` 알고리즘을 만들 수 있다.
  - Input을 Recursive하게 Partition한다.
  - 한 쪽 Partition으로만 Recursive 수행한다.
  - Quicksort의 러닝타임은 $\theta(n \lg n)$이었으나, 이 알고리즘의 러닝 타임은 $\theta(n)$이다.

## Quick Selection Algorithm

- S에서 Pivot v를 선택한다.
- $S - \\{ v \\}$를 파티션하여 $S_1$과 $S_2$로 나눈다.
- 만약 $i = 1 + \vert S_1 \vert$이면 v가 답이다.
- $i < \vert S_1 \vert$라면, i-th smallest element는 $S_1$에 존재할 것.
- 아니면, $S_2$에 존재할 것이고 $S_2$에서 $i - \vert S_1 \vert$ 번째 smallest element를 찾으면 된다.

```text
QUICK-SELECT(A, p, r, i)
  if p == r
    return A[p]
  q = PARTITION(A, p, r)
  k = q - p + 1
  if i == k
    return A[q]
  else if i < k
    return QUICK-SELECT(A, p, q - 1, i)
  else
    return QUICK-SELECT(A, q + 1, r, i - k)
```

- `PARTITION`을 Random element에 대해서 수행하면, `RANDOMIZED-SELECT`를 구현할 수 있다.

## Analysis of `RANDOMIZED-SELECT`

- Worst-case 러닝 타임은 $\theta(n^2)$이다.
- 러닝 타임 기대값을 계산하기 위해 길이가 n인 `A[p..r]`에 대한 러닝 타임을 T(n)이라고 하고, E[T(n)]의 Upper bound를 계산하자.
- `RANDOMIZED-PARTITION`은 피봇으로 임의의 원소를 동일한 확률로 선택한다.
- $1 \le k \le n$인 k에 대해서 subarray `A[p..q]`가 정확히 k개 elements를 가질 확률은 1/n이 된다.
- $X_k = I \\{ subarray\ A[p..q]\ has\ exactly\ k\ elements \\}$
- $E[X_k] = \frac{1}{n}$
- `RANDOMIZED-SELECT`를 호출할 때 `A[q]`를 피봇으로 선택하면 정답을 즉시 얻거나, `A[p..q-1]`로 Recurse 혹은 `A[q+1..r]`로 Recurse하게 된다.
- Upper bound를 구하기 위해 항상 큰 파티션쪽에 정답이 있다고 가정하자.
- Indicator random variable $X_k$는 값이 k일 때 1이 되고, 아니면 0이 된다.
- $X_k = 1$일 때, subarray는 k-1과 n-k 사이즈를 가진다.
- $T(n) \le \sum_{k=1}^{n} X_k(T(\max(k - 1, n - k)) + O(n))$
- $ = \sum_{k=1}^{n} X_k(T(\max(k - 1, n - k))) + O(n)$

- Expectation를 취하면,

$$
\begin{align}
E[T(n)] &\le E[\sum_{k=1}^n X_k(T(\max(k-1, n-k)) + O(n))] \\
        &= \sum_{k=1}^n E[X_k(T(\max(k-1, n-k)))] + O(n) \\
        &= \sum_{k=1}^n E[X_k]E[T(\max(k-1, n-k))] + O(n) \\
        &= \sum_{k=1}^n \frac{1}{n} \times E[T(\max(k-1, n-k))] + O(n)
\end{align}
$$

$$
\max(k - 1, n - k) = \begin{cases}k-1&if\ k \ge \lceil \frac{n}{2} \rceil \\ n-k&if\ k \le \lceil \frac{n}{2} \rceil \end{cases}
$$  

- $\sum_{k=1}^n \frac{1}{n} \times E[T(\max(k-1, n-k))]$을 살펴보았을 때, n이 짝수이면 $T(\lceil \frac{n}{2} \rceil)$부터 $T(n-1)$이 sum에서 2번씩 나타나고, n이 홀수면 추가적으로 $T(\lfloor \frac{n}{2} \rfloor)$가 한 번 더 나타난다.

$\implies E[T(n)] \le \frac{2}{n} \sum_{k=\lfloor \frac{n}{2} \rfloor}^{n-1}E[T(k) + O(n)] $  

- $E[T(n)] \le cn$을 가정하고 inductive hypothesis를 하자.

$$
\begin{align}
E[T(n)] &\le \frac{2}{n} \sum_{k=\lfloor \frac{n}{2} \rfloor}^{n-1}ck + an \\
        &= \frac{2c}{n} \left( \sum_{k=1}^{n-1}k - \sum_{k=1}^{\lfloor \frac{n}{2} \rfloor - 1}k\right) + an \\
        &= ... \\
        &\le cn - \left( \frac{cn}{4} - \frac{c}{2} - an \right)
\end{align}
$$

## Selection in Worst-case Linear Time

- Worst case에도 $O(n)$인 알고리즘을 만들고 싶다.
- Good split partitioning

```text
SELECT(A, p, r, i)
  while (r - p + 1) mod % ≠ 0
    // put the minimum into A[p]
    for j = p + 1 to r
      if A[p] > A[j]
        exchange A[p] with A[j]
    // If we want the minimum of A[p:r], we're done.
    if i == 1
      return A[p]
    // Otherwise, we want the (i - 1)st element of A[p + 1:r].
    p = p + 1
    i = i - 1
  // number of 5-element groups
  g = (r - p + 1)
  for j = p to p + g - 1
    sort <A[j], A[j + g], A[j + 2g], A[j + 3g], A[j + 4g]> in place
  // All group medians now lie in the middle fifth of A[p:r].
  // Find the pivot x recursively as the median of the group medians.
  x = SELECT(A, p + 2g, p + 3g - 1, ⌈g/2⌉)
  // partition around the pivot
  q = PARTITION-AROUND(A, p, r, x)
  
  k = q - p + 1
  if i == k
    return A[q]
  else if i < k
    return SELECT(A, p, q - 1, i)
  else
    return SELECT(A, q + 1, r, i - k)
```

- 대략적인 알고리즘을 설명하면 다음과 같다.
  1. 원소의 수가 5로 나누어 떨어지지 않으면, 가장 작은 원소를 제거해가면서 5의 배수로 만든다.
  2. 원소들을 5개씩 묶어 그룹을 만든다.
  3. 각 그룹을 sort한다.
  4. 각 그룹의 중앙값 중에서 가운데 값을 찾는다.
  5. 위에서 찾은 가운데 값을 사용해서 Partition을 수행한다.

![230413_linear_quick_selection.png]({{site.baseurl}}/assets/images/230413_linear_quick_selection.png)

- 위 그림에서 빨간색은 그룹들의 중앙값을 나타내며, $x$는 그들의 중앙값이다.
- 노란색 영역은 x보다 크거나 같고, 파란색 영역은 x보다 작거나 같다.
- 그룹의 수를 g라고 하면, 두 영역은 각각 최소 $3g/2$개의 원소를 보유하고 있다.
- PARTITION을 수행하면 두 영역 중 하나의 영역의 원소는 반드시 제외되므로 PARTITION 후 남는 원소는 많아도 $5g -\frac{3g}{2} = \frac{7g}{2} \le \frac{7n}{10}$ 이하이다.

- 이를 바탕으로 Reccurrence 식을 세워 Worst-case running time은 다음과 같고,

$$
T(n) \le T(\frac{n}{5}) + T(\frac{7n}{10}) + \theta(n)
$$

- $T(n) \le cn$을 대입하여 증명할 수 있다.

$$
\begin{align}
T(n) &\le T(\frac{n}{5}) + T(\frac{7n}{10}) + \theta(n) \\
     &\le \frac{cn}{5} + \frac{7n}{10} + \theta(n) \\
     &= \frac{9cn}{10} + \theta(n) \\
     &= cn - \frac{cn}{10} + \theta(n) \\
     &\le cn
\end{align}
$$
