---
title: "구간 트리(Segment Tree) 파이썬 코드"
excerpt: "Python - Segment Tree"
date: 2022-04-24 23:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

# 구간 트리<sup>Segment Tree</sup>

구간 트리<sup>segment tree</sup>는 저장된 자료들을 적절히 전처리해 그들에 대한 질의들을 빠르게 대답ㅎ라 수 있도록 한다. 가장 간단한 예로 구간의 최소치를 구하는 문제가 있다. 어떤 배열 A의 부분 구간의 최소치를 구하는 연산을 여러 번 하는 상황을 생각해보자. 예를 들어 `A=[1, 2, 1, 2, 3, 1, 2, 3, 4]`라면, `[2, 4]` 구간의 최소치는 1이고, `[6, 8]` 구간의 최소치는 2이다. 이 연산을 구현하는 간단한 알고리즘은 구간이 주어질 때마다 배열을 순회해서 최소치를 찾는 것으로, 질문 하나 당 $O(n)$의 시간이 걸린다. 

구간 트리의 기본적인 아이디어는 주어진 배열의 구간들을 표현하는 이진트리를 만드는 것이다. 이때 구간 트리의 루트는 항상 배열의 전체 구간 `[0, n-1]`을 표현하며, 한 트리의 왼쪽 자식과 오른쪽 자식은 각각 해당 구간의 왼쪽 반과 오른쪽 반을 표현한다.

아래는 구간 트리를 구현한 파이썬 코드이다.

```python
class SegTree:
    '''
    << 구간 트리(Segment tree) >> 
    
    - 아래 그림은 길이 15인 배열을 표현하는 구간 트리 예시
    +----------------------------------------------------------------+
    |                             (0,14)                             | <- 1번 원소에 저장
    +-------------------------------+--------------------------------+
    |             (0,7)             |               (8,14)           | <- 2번 원소, 3번 원소에 저장
    +---------------+---------------+-----------------+--------------+
    |     (0,3)     |     (4,7)     |     (8,11)      |    (12,14)   | <- 4, 5, 6, 7 번 원소에 저장
    +-------+-------+-------+-------+-------+---------+--------------+
    | (0,1) | (2,3) | (4,5) | (6,7) | (8,9) | (10,11) | (12,13) | 14 |
    +---+---+---+---+---+---+---+---+---+---+----+----+----+----+----+
    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
    +---+---+---+---+---+---+---+---+---+---+----+----+----+----+    
    
    (부모 원소가 i일 때 왼쪽 자식은 2*i 원소, 오른쪽 자식은 2*i+1 원소)
    '''
    
    def __init__(self, A, default, func):
        self.N = len(A)
        self.A = A
        self.SEG = [default for _ in range(4 * self.N)]
        self.func = func
        self.default = default
        self.init(A, 0, self.N-1, 1)
        
    def init(self, A, left, right, node):
        'node 노드가 A[left...right]을 표현할 때, node를 루트로 하는 서브트리를 초기화하고, 이 구간의 값 반환'
        if left == right:
            self.SEG[node] = A[left]
            return self.SEG[node]
        mid = (left + right) // 2
        leftRes = self.init(A, left, mid, node * 2)
        rightRes = self.init(A, mid + 1, right, node * 2 + 1)
        self.SEG[node] = self.func(leftRes, rightRes)
        return self.SEG[node]
    
    def query(self, left, right):
        '외부에서 query하기 위한 인터페이스'
        return self._query(left, right, 1, 0, self.N - 1)
    
    def _query(self, left, right, node, nodeLeft, nodeRight):
        'node가 표현하는 범위 A[nodeLeft...nodeRight]가 주어질 때, 이 범위와 A[left...right]의 교집합의 값 반환'
        
        # 두 구간이 겹치지 않을 때 -> 무시될 값 반환(예: 최소값 트리일 때는 매우 큰 값 반환)
        if left > nodeRight or right < nodeLeft:
            return self.default
        # node가 표현하는 범위가 array[left...right]에 완전히 포함되는 경우
        if left <= nodeLeft and nodeRight <= right:
            return self.SEG[node]
        # 양쪽 구간을 나눠서 푼 뒤 결과를 합친다.
        mid = (nodeLeft + nodeRight) // 2
        leftRes = self._query(left, right, node * 2, nodeLeft, mid)
        rightRes = self._query(left, right, node * 2 + 1, mid + 1, nodeRight)
        return self.func(leftRes, rightRes)
    
    def update(self, index, newValue):
        '외부에서 update하기 위한 인터페이스'
        return self._update(index, newValue, 1, 0, self.N - 1)
    
    def _update(self, index, newValue, node, nodeLeft, nodeRight):
        'A[index]=newValue로 바뀌었을 때, node를 루트로 하는 구간 트리를 갱신하고 노드가 표현하는 구간의 값 반환'
        # index가 현재 노드의 표현 구간과 상관 없을 때
        if index < nodeLeft or index > nodeRight:
            return self.SEG[node]
        # 트리의 리프까지 내려온 경우
        if index == nodeLeft and index == nodeRight:
            self.SEG[node] = newValue
            return self.SEG[node]
        # 구간을 나눠서 업데이트
        mid = (nodeLeft + nodeRight) // 2
        leftRes = self._update(index, newValue, node * 2, nodeLeft, mid)
        rightRes = self._update(index, newValue, node * 2 + 1, mid + 1, nodeRight)
        self.SEG[node] = self.func(leftRes, rightRes)
        return self.SEG[node]
```

아래는 `SegTree` 클래스를 이용하여 구간 최소 쿼리<sup>range minimum query, RMQ</sup> 예시이다.

```python
import math

A = [1, 2, 1, 2, 3, 1, 2, 3, 4]
tree = SegTree(A, math.inf, func=min)

print(tree.query(2, 4)) # 1
print(tree.query(6, 8)) # 2
```