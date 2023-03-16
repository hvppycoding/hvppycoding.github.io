---
title: "Algorithm - Prerequisites"
excerpt: "CLRS - Polynomials and the FFT"
date: 2023-03-06 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Insertion Sort

Incremental한 방법이다. i개에 대해서 원하는 형태일 때 데이터 1개가 더 늘었을 때 i+1개에 대해 원하는 형태로 만드는 방식을 뜻한다. 예를 들어 정렬의 경우 i개가 이미 정렬되어 있는 경우를 뜻한다.  

```text
INSERTION-SORT(A)
for j = A.length-1 downto 1
    key = A[j]
    // Insert A[j] into the sorted sequence A[j+1..n]
    i = j + 1
    while i <= n and A[i] <= key
        A[i-1] = A[i]
        i = i + 1
    A[i-1] = key
```

## Loop Invariants and the Correctness Proof

- 이 알고리즘이 우리가 원하는 일을 하는지 증명해야 한다. → 수학적 귀납법 사용
- Loop invariant를 위해서는 3가지를 보아야 한다.
  - Initialization: input size가 1개 혹은 0개일 때(basis case) 맞게 동작하는 것을 보인다.
  - Maintenance: k개 이하의 input size에 대해 원하는 결과가 나왔을 때, k+1개의 input size에 대해서 적용하였을 때 원하는 결과가 나옴을 보인다.
  - Termination: 루프가 종료되었을 때 원하는 결과가 나오는 것을 확인

## Insertion Sort 예시
- Loop invariant
  - 각 iteration에서 subarray `A[i ... j-1]`은 `A[1 ... j-1]`의 원래 원소로 구성되어있되, 오름차순으로 정렬되어 있다.
- Intialization:
  - `j=2`일 때 `A[1 ... j-1]`은 기존의 `A[1]`과 동일한 `A[1]` 하나이다. 한개의 원소로 구성된 subarray는 정렬되어 있다고 볼 수 있다.
- Maintenance:
  - `A[j-1]`, `A[j-2]`, `A[j-3]` 순으로 `A[j]`가 들어갈 알맞은 위치가 찾아질때까지 원소를 옆으로 한 칸씩 옮긴다. 
  - 그러면 subarray `A[1 ... j]`는 기존에 `A[1 ... j]`에 있던 원소들이 정렬된 상태로 되어있다.
  - 다음 이터레이션을 위해 `j`를 증가시키면 loop invariant를 유지한다.
- Termination:
  - `j=n+1`일 때 loop가 끝난다.
  - 위의 Loop invariant 설명에서 `j`를 `n+1`로 대체하면, "`A[i ... n]`은 `A[1 ... n]`의 원래 원소로 구성되어있되, 오름차순으로 정렬되어 있다."라는 말이 되며, 이는 전체 array가 정렬되어있고 알고리즘이 올바르다는 것을 뜻한다.

## Divide and Conquer

- 각 Recursion 단계에서 3단계를 Recursive하게 적용하여 문제를 푼다.
  - **Divide**: 풀려는 문제와 동일한 subproblem 문제들로 나눈다.
  - **Conquer**: subproblem들을 재귀적으로 푼다. 만약 problem size가 작으면(base case) subproblem을 간단한 방식으로 푼다.
  - **Combine**: subproblem 문제들의 솔루션을 original 문제의 솔루션으로 만든다.

## Merge Sort

- Divide: n개의 원소를 가진 sequence를 2개의 subsequence로 나눈다.
- Conquer: 두 subsequence를 recursive하게 정렬한다.
- Combine: 두 개의 정렬된 subsequence를 merge하여 정렬된 답을 낸다.

## Merge Procedure

- Merge sort 알고리즘의 핵심 동작은 다음과 같다.
- subarray `A[p...q]`와 `A[q+1...r]`가 정렬되어 있다고 가정한다.
- subarray들을 하나의 정렬된 subarray로 합치고 이를 현재 subarray `A[p...r]`에 replace한다.
- 합치는 과정은 보조 함수 `MERGE(A, p, q, r)`을 호출하여 수행하고, `A`는 array, `p`, `q`, `r`은 array의 index들이다(`p ≤ q < r`).

## Merge Sort 수행 시간 분석

- Divide: subarray의 가운데를 찾음. `Θ(1)` 시간
- Conquer: Recursive하게 두 개의 n/2 크기의 subproblem을 푼다. `2T(n/2)` 시간
- Conbine: `MERGE` 단계를 수행한다. `Θ(n)` 시간

점화식은 다음과 같다.
- `T(1) = 1`
- `T(n) = 2T(n/2) + n`

식을 전개하여 풀면, 수행시간 $O(n \lg n)$ 얻을 수 있다.
