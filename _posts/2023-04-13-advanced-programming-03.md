---
title: "Sorting in Linear Time"
excerpt: "Advanced Programming Methodologies 03"
date: 2023-04-13 02:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Outline

- 비교 기반 정렬의 경우 $\Omega (n \log n)$이 알고리즘의 하한이다.

## Lower Bounds for Sorting

- 비교를 통한 정렬을 Decision Tree 나타낼 수 있다.
- i:j는 A[i]와 A[j]를 비교하는 것을 나타내며,
- 아래 그림은 A = {6, 8, 5} 배열에 대한 Decision Tree를 나타낸다.

![230413_compare_decision_tree.png]({{site.baseurl}}/assets/images/230413_compare_decision_tree.png)

- 어떤 비교 정렬 알고리즘이라도 Worst case의 경우 최소 $\Omega (n \log n)$만큼의 비교가 필요하다.
- 증명:
  - Decision tree가 h의 높이를 갖고 l개의 leaf가 있고, 각 Leaf는 정렬의 sort 결과와 매칭된다고 하자.
  - 가능한 정렬 조합이 n!가지 존재한다.
  - 또한, 높이가 h인 binary tree는 $2^h$개보다 많은 leaf를 가질 수 없다.
  - $n! \le l \le 2^h$를 얻을 수 있다.
  - 양쪽에 로그를 취하면, 

$$
\begin{align}
h &\ge \lg(n!) \\
  &= \lg n + \lg (n - 1) + \lg 1 \\
  &\ge \lg n + \lg (n - 1) + \lg \frac{n}{2} \\
  &\ge \frac{n}{2} \lg(\frac{n}{2}) \\
  &= \Omega(n \lg n) \\
\end{align}
$$

## Counting Sort

- n개의 정수 element가 있고, element는 1부터 k의 값을 가진다고 하자.
- $k = O(n)$일 경우, Sort는 $\theta(n)$ 시간이 걸린다.
- Algorithm
  - `A[1..n]`: initial input
  - `B[1..n]`: stored output
  - `C[1..k]`: 처음엔 `C[i]`에 A 배열 안의 i의 갯수를 저장한다. 그 후 배열합을 통해 i 이하인 갯수를 저장한다.
  - `C[A[j]]`에는 아웃풋에서의 `A[j]`의 위치가 저장되어 있다.

```text
COUNTING-SORT(A, n, k)
  let B[1:n] and C[0:k] be new arrays
  for i = 0 to k
    C[i] = 0
  for j = 1 to n
    C[A[j]] = C[A[j]] + 1
  // C[i] now contains the number of elements equal to i.
  for i = 2 to k
    C[i] = C[i] + C[i - 1]
  // C[i] now contains the number of elements ≤ i.
  // Copy A to B, starting from the end of A.
  for j = n downto 1
    B[C[A[j]]] = A[j]
    C[A[j]] = C[A[j]] - 1    // to handle duplicate values
  return B
```

## Property of Counting Sort

- Element간 비교가 일어나지 않는다.
- Statbility가 유지된다.(같은 숫자가 여러 번 나올 경우, 기존에 앞에 있던 숫자가 결과에서도 앞에 있음)

## Radix Sort

- Counting sort는 작은 정수에 대해서만 작동한다.
- Radix Sort는 한 번에 하나의 Digit만 정렬한다.
  - 각 Input은 d decimal digits
  - Counting sort와 같은 stable sorting 알고리즘을 사용한다
  - lowest order digit부터 highest digit까지 반복적으로 정렬한다
  - Sorting algorithm이 **stable**하므로, low order sort 뒤 high order sort를 수행하면 뒷자리에 대한 정렬이 유지된다.

```text
RADIX-SORT(A, n, d)
  for i = 1 to d
    use a stable sort to sort array A[1:n] on digit i
```

## Bucket Sort

- n개의 input이 uniform distribution [0, 1) 에서 추출되었다고 가정하자.
- Average-case running time은 O(n)

1. [0, 1) 구간을 동일한 n개의 구간으로 나눈다(buckets).
2. n개의 input을 버킷으로 분배한다.
3. 각 버킷에 있는 숫자들을 정렬한다.

```text
BUCKET-SORT(A)
  let B[0..n-1] be a new array
  n = A.length
  for i=0 to n-1
    make B[i] an empty list
  for i=1 to n
    insert A[i] into list B[⌊nA[i⌋]
  for i=0 to n-1
    sort list B[i] with insertion sort
  concatenate the list B[0], B[1], B[n-1] together in order
```

## Analysis of Bucket Sort

- $n_i$를 버킷 `B[i]`에 들어가는 element 수를 나타내는 random variable이라고 하자.
- insertion sort는 quadratic time이므로, bucket sort의 러닝 타임은
  - $T(n) = \theta(n) + \sum_{i=0}^{n-1}O(n_i^2)$
- 이제 평균 러닝 타임을 구하기 위해 Expectation을 구한다.

$$
\begin{align}
E[T(n)] &= E \left[ \theta(n) + \sum_{i=0}^{n-1}O(n_i^2) \right] \\
        &= \theta(n) + \sum_{i=0}^{n-1}E[O(n_i^2)] \\
        &= \theta(n) + \sum_{i=0}^{n-1}O(E[n_i^2]) \\
        & When\ E[n_i^2] = 2 - \frac{1}{n} \implies \\
        &= \theta(n) + n \cdot O(2 - \frac{1}{n}) = \theta(n)
\end{align}
$$

- $E[n_i^2]$를 구해보자. $X_{ij}=I \\{ A[j]\ falls\ in\ bucket\ i \\}$이라고 하면, $n_i = \sum_{j=1}^n X_{ij}$이다.

$$
\begin{align}
E[n_i^2] &= E[(\sum_{j=1}^n X_{ij})^2] \\
         &= E[\sum_{j=1}^n \sum_{k=1}^n X_{ij} X_{ik}] \\
         &= E[\sum_{j=1}^n X_{ij}^2 + \sum_{j=1}^n \sum_{1 \le k \le n, k \neq j} X_{ij} X_{ik}] \\
         &= \sum_{j=1}^n E[X_{ij}^2] + \sum_{j=1}^n \sum_{1 \le k \le n, k \neq j} E[X_{ij} X_{ik}] \\
         & 확률\ 변수\ X_{ij}는\ \frac{1}{n}의\ 확률로\ 1이다. \\
         & E[X_{ij}^2] = 1^2 \cdot \frac{1}{n} + 0^2 \cdot (1 - \frac{1}{n}) = \frac{1}{n} \\
         & k \neq j일\ 경우,\ X_{ij}와\ X_{ik}는\ 독립이다. \\
         & E[X_{ij} X_{ik}] = E[X_{ij}]E[X_{jk}] = \frac{1}{n} \cdot \frac{1}{n} = \frac{1}{n^2} \\
         & 이를\ 대입하면, \\
         &= \sum_{j=1}^n \frac{1}{n} + \sum_{j=1}^n \sum_{1 \le k \le n, k \neq j} \frac{1}{n^2} \\
         &= n \cdot \frac{1}{n} + n(n-1)\cdot \frac{1}{n^2} \\
         &= 2 - \frac{1}{n}
\end{align}
$$

- CLRS 책에서는, "분산 = 제곱의 평균 - 평균의 제곱" 공식을 활용한다.
  - $p = \frac{1}{n}, q = 1 - p$
  - $Var[n_i] = npq = 1 - \frac{1}{n}$
  - $E[n_i] = np = 1$
  - $Var[n_i] = E[n_i^2] - E^2[n_i]$

- 이를 활용하면,

$$
\begin{align}
E[n_i^2] &= Var[n_i] + E^2[n_i] \\
         &= (1 - 1/n) + 1^2 \\
         &= 2 - 1/n
\end{align}
$$
