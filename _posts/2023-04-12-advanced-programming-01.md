---
title: "Heap Sort"
excerpt: "Advanced Programming Methodologies 01"
date: 2023-04-12 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Introduction

- How to Succeed in this course:
  - 연필과 컴퓨터로 많은 문제를 풀어보자.
  - 1주일에 하루는 이 수업을 위해 공부하자.
  - 수업 시간에 많이 질문하자. If you think you get lost, try to catch up with the help of TAs.

- Textbook: Introduction to Algorithms(4th Edition)

## 점근적 표기

### O-표기법

- 정의: $O(g(n)) = \\{ f(n)\;\vert\;\exists c > 0, n_0 > 0\;s.t.\;\forall n \ge n_0, f(n) \le cg(n) \\}$
- 풀어서 말하면, $O(g(n)) = \\{ f(n)\;\vert\;$ 모든 $n \ge n_0$에 대하여 $f(n) \le cg(n)$인 양의 상수 $c$와 $n_0$가 존재한다 $\\}$
- 직관적으로 표현하면, $O(g(n))=\\{ f(n)\;\vert\;$ 충분히 큰 모든 $n$에 대하여 $f(n) \le cg(n)$인 양의 상수 $c$가 존재한다 $\\}$
- 예: $5n^2 + 3 = O(n^2)$임을 보여라.
  - $c=6$, $n_0=2$일 때 Big O 정의에 따라 $O(n^2)$이다.
  - $O(n^2) = \\{ f(n)\;\vert\;\exists c > 0, n_0 > 0\;s.t.\;\forall n \ge n_0, f(n) \le cn^2 \\}$

## Heap Sort

### Heaps

- Array로 구현된 Heap은 Complete Binary Tree로 볼 수 있다.
- 각 Array Element는 트리 노드에 해당한다.
- `A[1]`이 루트 노드이며, 인덱스를 통해 자식 노드, 부모 노드에 접근할 수 있다.
  - $LEFT(i)=2i$
  - $RIGHT(i)=2i+1$
  - $PARENT(i)=\lfloor{i/2}\rfloor$

### Min Heap

- Min Heap은 모든 노드 i에 대해 `A[PARENT(i)] ≤ A[i]`를 만족한다(루트 노드는 제외).
- Node의 높이(Height)는 특정 Node에서 가장 긴 Leaf 노드까지의 Edge 수를 뜻한다.
- Heap의 높이는 Root 노드의 높이이다.
- Complete Binary Tree이므로 height는 `O(lg n)`이다.
  - k level의 노드 수 $l_k = 2^{k-1}$
  - Height가 h인 트리를 생각해보면,
    - $n=1+2^1+2^2+...+2^{h-1}=2^h-1$
    - 정리하면, $log_{2} (n+1) = h$

### Max Heap

- `A[i]`가 max-heap property를 violate한다면?
- `MAX-HEAPIFY`를 통해 `A[i]`의 위치를 조정한다.

```text
MAX-HEAPIFY(A, i)
  L = Left(i)
  R = Right(i)
  if L ≤ A.heap-size and A[L] > A[i]
    largest = L
  else
    largest = i
  if R ≤ A.heap-size and A[R] > A[largest]
    largest = R
  if largest ≠ i
    A[i] ↔ A[largest]
    MAX-HEAPIFY(A, largest)
```

### Analysis of `MAX-HEAPIFY`

- 크기가 n인 Tree Root에서 Child Subtree 중 가능한 큰 Subtree는 최대 약 2n/3개의 노드를 가지고 있음.

```text
             1
          /     \
         2       3
       /   \    / \
      4     5  6   7
     / \   / \
    8   9 10 11
```

- 크기가 n인 트리에 대해서 수행 시간을 계산해보면
  - `A[i]`, `A[LEFT(i)]`, `A[RIGHT(i)]` 비교 및 Swap 연산 = $\theta(1)$
  - Recursive로 최대 $T(\frac{2}{3}n)$
  - $T(n) \le T(\frac{2}{3}n) + \theta(1)$
  - $T(n) = O(\lg{n})$

### `BUILD-MAX-HEAP(A)`

- `A[1..n]`을 max-heap으로 바꾸기 위해 Bottom-up으로 `MAX_HEAPIFY`를 할 수 있음

```text
BUILD-MAX-HEAP(A)
  for i = A.length/2 downto 1
    MAX-HEAPIFY(A, i)
```

### Loop Invariant

- Loop Invariant를 통해 알고리즘이 올바른지 확인할 수 있다.
- Loop에 대해서 다음 3가지를 확인해야 한다.
  - **Initialization**: First iteration 전에 True여야함
  - **Maintenance**: 이전 Iteration에서 True 였다면, 다음 Iteration에서도 True이다.
  - **Termination**: Loop가 종료될 때, Invariant Property를 통해 알고리즘이 올바르다는 것을 보인다.

### Correctness Proof of `BUILD-MAX-HEAP(A)`

- **Loop Invariant:**
  - 각 iteration 시작점에서 `i+1`, `i+2`, ..., `n` 각 노드는 max-heap의 root이다.
- **Initialization:**
  - First Iteration 전에 $i = \lfloor \frac{n}{2} \rfloor$이고, $\lfloor \frac{n}{2} \rfloor + 1, \lfloor \frac{n}{2} \rfloor + 2, ..., n$은 Leaf node이므로 max-heap의 루트라고 볼 수 있다.
- **Maintenance:**
  - Loop Invariant 속성에 의해 노드 i의 양쪽 Child는 모두 max-heap의 루트들이다. 그러므로 `MAX-HEAPIFY(A, i)`는 노드 i에 대해 max-heap subtree로 만들어준다. 또한 `i+1`, `i+2`, ..., `n`이 subtree root 속성도 유지된다. i를 1 줄여주면 다음 Iteration의 Loop Invariant가 성립하게 된다.
- **Termination:**
  - `i = 0`일 때 Loop가 종료되며, Loop Invariant에 의해 1, 2, ..., n 각 노드는 max-heap의 루트가 된다. 즉, 노드 1이 max-heap의 루트가 된다.

### Time complexity of `BUILD-MAX-HEAP`

- Simple한 접근: O(n)번 `MAX-HEAPIFY`를 호출한다. `MAX-HEAPIFY`는 O(lg n) 시간이 걸린다. → O(n lg n)

### A Tighter Upper Bound

- 높이가 h인 노드에 대해 `MAX-HEAPIFY` 연산은 O(h) 시간이 걸린다.
- n 크기의 heap은 $\lfloor \lg{n} \rfloor$의 높이를 가진다.
- 높이가 h인 노드는 최대 $\lceil \frac{n}{2^{h+1}} \rceil$개 이다.
- 모든 높이에 대해 (높이 h인 노드 수 × 노드 당 연산)을 더한다.

$$
\begin{align}
\sum_{h=0}^{\lfloor \lg{n} \rfloor} \lceil \frac{n}{2^{h+1}} \rceil O(h) &\le \sum_{h=0}^{\lfloor \lg{n} \rfloor} (\frac{n}{2^{h+1}}) O(h)\\
&=O(n \sum_{h=0}^{\lfloor \lg{n} \rfloor}\frac{h}{2^{h}}) + \sum_{h=0}^{\lfloor \lg{n} \rfloor}O(h) \\
&\le O(n \sum_{h=0}^{\infty}\frac{h}{2^{h}})+O((\lg{n})^2)\\
&= O(n) + O((\lg{n})^2) = O(n)\\
\end{align}
$$

- $\sum_{h=0}^{\infty}\frac{h}{2^{h}}$ 계산하는 법

$$
\begin{align}
S &= \sum_{h=0}^{\infty}\frac{h}{2^{h}} = \frac{0}{2^0} + \frac{1}{2^1} + \frac{2}{2^2} + \frac{3}{2^3} + \cdots\\
\frac{1}{2}S &= \frac{0}{2^1} + \frac{1}{2^2} + \frac{2}{2^3} + \frac{3}{2^4} + \cdots\\
S - \frac{1}{2}S &= \frac{1}{2^1} + \frac{1}{2^2} + \frac{1}{2^3} + \cdots = \frac{ \frac{1}{2} }{ 1 - \frac{1}{2}} = 1\\
\\
\therefore S &= 2
\end{align}
$$

## Heapsort

- `BUILD-MAX-HEAP`을 통해 max-heap을 만든다.
- 루트인 `A[1]`에는 가장 큰 Element가 저장되어 있다.
- `A[1]`과 `A[n]`을 swap한 뒤 heap size를 1 줄이고 `MAX-HEAPIFY(A, 1)`을 호출하여 `A[1..n-1]`을 max-heap으로 만든다.
- 이를 heap을 줄이며 반복한다.
- O(n lg n) 시간이 걸린다.

```text
HEAPSORT(A)
  BUILD-MAX-HEAP(A)
  for i = A.length downto 2
    A[1] ↔ A[i]
    A.heap-size = A.heap-size - 1
    MAX-HEAPIFY(A, 1)
```

## Priority Queues

- `HEAP-MAXIMUM(A)`: `A[1]`를 리턴
- `HEAP-EXTRACT-MAX(A)`:
  1. `A[1]` Result로 저장
  2. `A[N]` 원소를 1번 인덱스에 넣으면서 Heapsize 줄인다
  3. `MAX-HEAPIFY(A, 1)` 수행
- `HEAP-INCREASE-KEY(A, i, key)`: i번째 원소를 key로 증가시킨다(증가만 가능)
  1. `A[i] = key`를 대입 후
  2. `i > 1`이고 `A[PARENT(i)] < A[i]`를 만족하는 동안 `A[i] ↔ A[PARENT(i)]; i = PARENT(i)` 수행
- `MAX-HEAP-INSERT(A, key)`: key값을 가진 새 원소를 추가한다.
  1. heap의 끝에 원소를 추가한다.
  2. `HEAP-INCREASE-KEY(A, A.heap-size, key)`를 통해 원소의 키값을 세팅하고 heapify한다.
