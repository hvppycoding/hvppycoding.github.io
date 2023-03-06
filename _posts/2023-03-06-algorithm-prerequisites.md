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

