---
title: "Quick Sorting"
excerpt: "Advanced Programming Methodologies 02"
date: 2023-04-12 23:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Quicksort

- **Divide:** Array A[1..r]을 피봇(Pivot) A[p]를 기준으로 두 개의 Subarray A[1..p-1]과 A[p+1..r]로 나눈다.
  - Every element in A[1..p-1] ≤ A[p]
  - A[p] ≤ Every element in A[p+1..r]

![230412_quicksort_divide.png]({{site.baseurl}}/assets/images/230412_quicksort_divide.png)

- **Conquer:** A[1..p-1]과 A[p+1..r]를 Recursive quicksort로 정렬한다.
- **Combine:** 추가 Combine 작업은 불필요

## Quicksort Implementation Type1

- Partition 방법에 따라 두 가지 구현 존재

![230412_quicksort_partition1.png]({{site.baseurl}}/assets/images/230412_quicksort_partition1.png)

```cpp
template <class T>
void QuickSort(T *a, const int left, const int right) {
  // Sort a[left:right] into nondecreasing order.
  // a[left] is arbitrarily chosen as the pivot. Variables i and j
  // are used to partition the subarray so that at any time a[m] ≤ pivot, m < i
  // and a[m] ≥ pivot, m > j. It is assumed that a[left] ≤ a[right + 1]
  if (left < right) {
    int i = left,
    j = right + 1,
    pivot = a[left];
    do {
      do i++; while (a[i] < pivot); // 경계 확인을 넣어야할 것 같음
      do j--; while (a[j] > pivot);
      if(i < j) swap(a[i], a[j]);
    } while (i<j);
    swap(a[left], a[j]);

    QuickSort(a, left, j-1);
    QuickSort(a, j+1, right);
  }
}
```

## Quicksort Implementation Type2

- 두 번째 구현
- i는 x보다 작거나 같은 부분의 마지막 원소를 가리킴
- j는 p부터 r-1까지 증가시키면서 원소 비교 수행

![230412_quicksort_partition2.png]({{site.baseurl}}/assets/images/230412_quicksort_partition2.png)

```text
PARTITION(A, p, r)
  x = A[r]
  i = p - 1
  for j = p to r-1
    if A[j] ≤ x
      i = i + 1
      A[i] ↔ A[j]
  A[i+1] ↔ A[r]
  return i + 1
```

```text
QUICKSORT(A, p, r)
  if p < r
    q = PARTITION(A, p, r)
    QUICKSORT(A, p, q-1)
    QUICKSORT(A, q+1, r)
```

## Correctness of `PARTITION` Procedure

- **Loop Invariant:** 각 Iteration의 시작에서 다음을 만족한다.
  1. A[k] ≤ x    if p ≤ k ≤ i
  2. A[k] > x    if i+1 ≤ k ≤ j-1
  3. A[k] = x    if k = r
- **Initialization:** 첫 Iteration 시작 전 i=p-1, j=p에서 subarray empty이므로 trivial true이다. x=A[r]를 수행했으므로 세 번째 항목도 만족한다.
- **Maintenance:** 
  - A[j] > x일 때, j만 증가되므로 2번 항목이 만족하며, 나머지 항목은 변함없다.
  - A[j] ≤ x일 때, i가 증가하고, A[i]와 A[j]가 스왑되고, 그 후 j가 증가한다. 스왑으로 인해 A[i] ≤ x와 A[j-1] > x가 만족하게 된다.
- **Termination:**
  - j=r일 때 Termination된다. 이 때 모든 각각의 원소는 Invariant의 세 개의 Set 중에 하나에 속하게 된다.
  - `PARTITION`의 마지막 두 라인에서 x보다 큰 원소들 중 가장 왼쪽의 원소와 x를 스왑한다.

![230412_quicksort_loop_invariant.png]({{site.baseurl}}/assets/images/230412_quicksort_loop_invariant.png)

## Analysis of Quicksort

$T(n) = T(i) + T(n-i-1) + n$

- Worst-case partitioning: 파티션이 항상 n-1 원소와 0 원소로 나누어질 때 Worst이다. $O(n^2)$
- Best-case paritioning: 파티션이 항상 절반에 가깝게 나누어질 때 Best이다. $O(n \log{n})$

- Worst-case analysis
  - $T(n) = \max_{0 \le q \le n-1}(T(q) + T(n-q-1)) + \theta(n)$
- $T(n) \le cn^2$ 가정하자
  - $$\begin{align}T(n) &= \max_{0 \le q \le n-1}(cq^2 + c(n-q-1)^2) + \theta(n)\\
           &= c \cdot max_{0 \le q \le n-1}(q^2 + (n-q-1)^2) + \theta(n)\\ \end{align}$$
- $q^2 + (n-q-1)^2$는 $q = 0$이거나 $q=n-1$일 때 최대가 된다.
  - $\max_{0 \le q \le n-1}(q^2 + (n-q-1)^2) \le (n-1)^2$
- Thus, $T(n) \le cn^2 - c(2n - 1) + \theta(n) \le cn^2$

## Balanced partitioning

- `PARTITION`이 항상 9 대 1 비율로 일어난다고 가정하자.
  - 상당히 Unbalance하다.
  - Recurrence는 $T(n) = T(9n/10) + T(n/10) + \theta(n)$이다.
  - 아래 그림은 Recursion Tree이다.
    - Tree의 Level별 cost 합을 살쳐보면 depth $\log_{10}{n} = \theta(\lg n)$까지는 $cn$이다.
    - Recursion은 $\log_{9/10}{n} = \theta(\lg n)$이 최대 길이이다.
    - 최대 깊이 x 레벨 별 cost를 곱하여 $O(n \lg n)$의 time complexity가 된다.

![230412_quicksort_recursion_tree.png]({{site.baseurl}}/assets/images/230412_quicksort_recursion_tree.png)

## Intuition for the average case

![230412_quicksort_average_intuition.png]({{site.baseurl}}/assets/images/230412_quicksort_average_intuition.png)

- Good split과 Bad split이 반복된다고 하자. bad split cost인 $\theta(n-1)$이 $\theta(n)$에 합쳐질 수 있다(그림). 이 경우 수행 시간은 $O(n \lg n)$이 된다.

## Expected running time

- n 원소 array 대해 `QUICKSORT`를 수행할 때, 전체 실행 과정 중 `PARTITION` 내에서 X번 비교가 일어난다고 하자. 그렇다면 `QUICKSORT`의 러닝 타임은 O(n + X)이다.
- 한 번 `PARTITION`을 Call할 때 마다 Pivot이 골라지며, 이는 다시는 `QUICKSORT`나 `PARTITION`에 포함되지 않는다.
- 그러므로 최대 n번만 `PARTITION` 콜이 가능하다.

- A의 원소를 크기 순으로 $z_1, z_2, ..., z_n$이라고 하자. $z_i$는 i번째 작은 원소가 된다.
- A = {3, 5, 1, 9}이면 $z_1=1, z_2=3, z_3=5, z_4=9$이다.
- $Z_{i, j} = \\{ z_i, z_{i+1}, ..., z_j \\} $ : Elements between $z_i$ and $z_j$
- $X_{i, j} = I \\{ z_i\ is\ compared\ to\ z_j \\} $ : $z_i$와 $z_j$가 비교되었다면 $I=1$, 아니면 $I=0$이 된다.
- 총 비교 횟수는 $X = \sum_{i=1}^{n-1} \sum_{j=i+1}^{n} X_{i, j}$ 이다.
- 기대값을 구해야한다.

$$
\begin{align}
E[X] &= E[\sum_{i=1}^{n-1} \sum_{j=i+1}^{n} X_{i, j}]\\
     &= \sum_{i=1}^{n-1} \sum_{j=i+1}^{n} E[ X_{i, j} ]\\
     &= \sum_{i=1}^{n-1} \sum_{j=i+1}^{n} Prob \{ z_i\ is\ compared\ to\ z_j \} \\
\end{align}
$$

- 언제 $z_i$와 $z_j$가 비교되는가?
  - 하나의 Element pair는 최대 1번 비교된다.
  - 비교는 Pivot과만 일어나며, 한 번 Pivot이 된 후에는 다시 비교되지 않는다.
  - $z_i < x < z_j$인 x가 pivot이 되었다면, $z_i$와 $z_j$는 비교되지 않는다.
  - $z_i$가 pivot으로 골라졌다면, $z_i$는 $Z_{i, j}$의 모든 원소와 비교된다.
  - $z_j$가 pivot으로 골라졌다면, $z_j$는 $Z_{i, j}$의 모든 원소와 비교된다.

- $Prob \\{ z_i\ compared\ to\ z_j \\} = \frac{2}{j - i + 1}$ 이다.
  - 증명: `RANDOMIZED-PARTITION`으로 pivot을 랜덤하게 고른다고 가정하자.
  - $Prob \\{ z_i\ compared\ to\ z_j\ from\ Z_{i,j}\\} = \frac{2}{j - i + 1}$
  - $Prob \\{ z_i\ compared\ to\ z_j\ from\ Z_{i-1,j}\\}$
    - $= \frac{1}{j - i + 2} \cdot Prob \\{ z_i\ compared\ to\ z_j\ from\ Z_{i,j} \\} + \frac{2}{j-i+2}\cdot 1 + \frac{j-i-1}{j-i+2}\cdot 0$
    - $= \frac{1}{j - i + 2} \cdot \frac{2}{j-i+1} + \frac{2}{j-i+2}\cdot 1 + \frac{j-i-1}{j-i+2}\cdot 0$
    - 식을 정리해보면, $\frac{2}{j - i + 1}$
  - $Z_{i, j+1}$에 대해서도 마찬가지이다.
  - $Z_{1, n}$까지도 확장 가능하다.

- 수행 시간 계산을 위해 알아야할 특성

![230412_harmonic_series.png]({{site.baseurl}}/assets/images/230412_harmonic_series.png)

$$
\begin{align}
S &= s_1 + s_2 + s_3 + \cdots + s_n \\
  &= \frac{1}{2} + \frac{1}{3} + \cdots + \frac{1}{n} \\
  &< \int_1^n \frac{1}{x} dx = [\ln x]_1^n = \ln n
\end{align}
$$

- Average 계산

$$
\begin{align}
E[X] &= \sum_{i=1}^{n-1} \sum_{j=i+1}^{n} Prob \{ z_i\ is\ compared\ to\ z_j \} \\
     &= \sum_{i=1}^{n-1} \sum_{j=i+1}^{n} \frac{2}{j-i+1} \\
     &= \sum_{i=1}^{n-1} \sum_{k=1}^{n-i} \frac{2}{k+1} \\
     &< \sum_{i=1}^{n-1} \sum_{k=1}^{n} \frac{2}{k+1} \\
     &= \sum_{i=1}^{n-1} O(\lg n)\\
     &= O(n \lg n)\\
\end{align}
$$

## Expected running time 2

- 다른 증명법

- $2 \le k < n$인 모든 k에 대해 $T(k) \le ck\log k$가 성립한다고 가정하고, $T(n) \le n \log n$이 됨을 증명하자.
  - 우선 $T(2) = c2 \log 2$가 되는 c를 잡을 수 있다.

$$
\begin{align}
T(n) &= \frac{1}{n}\sum_{i=1}^{n}[T(i-1) + T(n-i)] + \theta(n) \\
     &= \frac{2}{n}\sum_{k=0}^{n-1}T(k) + \theta(n) \\
     &= \frac{2}{n}\sum_{k=2}^{n-1} T(k) + \theta(n) \qquad // 상수항\ \theta(n)에\ 흡수 \\
     &\le \frac{2c}{n}\sum_{k=2}^{n-1} k \log k + \theta(n) \\
     &\le \frac{2c}{n}\int_{2}^{n} x \log x dx + \theta(n) \\
     &= \frac{2c}{n} \left[ \frac{x^2 \log x}{2} - \frac{x^2}{4} \right]_{2}^{n} + \theta(n) \\
     &= cn\log n - \frac{cn}{2} + \theta(n) \qquad // 상수항\ 생략 \\
     &\le cn\log n
\end{align}
$$