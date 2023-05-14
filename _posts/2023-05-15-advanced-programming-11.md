---
title: "Minimum Spanning Trees"
excerpt: "Advanced Programming Methodologies 11"
date: 2023-05-15 03:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Minimum Spanning Tree

- 전자회로 설계에서 여러 핀들을 와이어로 연결할 때 전기적으로 동일해진다.
- n개의 핀을 연결할 때 n-1개의 와이어를 사용할 수 있다. 사용하는 와이어의 길이가 최소가 되는 것이 좋다.
- connected, undirected graph $G=(V, E)$로 모델링할 수 있다.
- $V$는 핀의 집합이고, $E$는 핀쌍에 대해 가능한 연결을 나타낸다.
- $w(u, v)$는 비용을 나타낸다(연결에 필요한 와이어의 길이).
- 이 때 모든 vertex들을 연결하는 $T \subseteq E$이고 $w(T) = \sum_{(u, v) \in T} w(u, v)$를 최소화하는 acyclic subset $T$를 찾자.
- $T$가 acyclic이고 모든 vertex들을 연결하기 때문에 MST(Minimum Spanning Tree)라고 부른다.

## Algorithms

- 먼저, 한 번에 하나의 Edge를 추가하여 Spanning tree를 키워가는 일반적인 Minimum-spanning-tree method를 소개한다.
- 두 개의 greedy 알고리즘을 살펴보고, greedy 전략이 MST를 찾는다는 것을 증명할 것이다.
- Kruskal's algorithm과 일반적인 바이너리 힙을 사용하는 Prim's algorithm이 $O(E \lg V)$ 시간에 동작하게 할 것이다.
- 피보나치 힙을 사용하여 Prim's algorithm이 $O(E + V \lg V)$ 시간에 동작하게 할 것이다. 이는 $V$가 $E$보다 훨씬 적을 경우 바이너리 힙 구현보다 성능이 향상된다.

## `GENERIC-MST`

- $A$가 MST의 부분집합이라고 하자.
- $A ∪ \lbrace (u, v) \rbrace$ 가 MST의 부분집합이면 $Edge $(u, v)$는 안전(safe)하다고 할 수 있다.

```text
GENERIC-MST(G,w)
1. A = ∅
2. while A does not form a spanning tree
3.   find an edge (u,v) that is safe for A
4.   A = A ∪ {(u,v)}
5. return A
```

## Loop Invariant

- Loop Invariant: 각 이터레이션 전에 A는 MST의 부분집합이다.
- Initialization: A는 빈 집합이므로 trivially true
- Maintenance: loop 2-4는 A에 안전한 edge를 찾아 추가하므로 Invariant가 유지된다.
- MST의 모든 Edge가 $A$에 추가된다. line 5에서 $A$는 MST이다.

## Terms

- $A ∪ \lbrace (u, v) \rbrace$가 MST의 부분집합이면 $Edge (u, v)$를 Safe edge라고 한다.
- undirected graph $G=(V, E)$의 **cut** $(S, V-S)$는 $V$를 나누는 것이다.
- 만약 Edge $(u, v) \in E$의 하나의 Endpoint가 $S$, 다른 하나의 Endpoint가 $V-S$에 있으면 Edge $(u, v)$가 cut $(S, V-S)$를 **cross**한다고 한다.
- edge 집합 A에 속한 edge들이 어떤 cut을 cross하지 않으면, 그 cut은 A를 **respect**한다고 한다.
- cut을 cross하는 edge들 중 가장 작은 weight를 가진 edge를 **light edge**라고 한다.

## Cut(S, V-S)

![2023-05-15-graph-cut.png]({{site.baseurl}}/assets/images/2023-05-15-graph-cut.png){: .align-center}  

- cut $(S, V-S)$가 그림에 나타나있다.
- Edge (d,c)는 cut을 cross하는 unique light edge이다.

## Theorem 23.1

- G=(V,E)가 E에 대해 weight function w가 정의된 connected undirected graph라고 하자.
- A가 G의 어떤 MST에 속하는 edge들의 집합이라고 하자.
- (S, V-S)가 A를 respect하는 G의 어떤 cut이라고 하자. 그리고 (u, v)는 cut (S, V-S)를 light edge라고 하자.
- 그러면 edge (u, v)는 A에 대해 safe하다.

## Theorem 23.1 (Proof)

- $T$가 $A$를 포함하는 MST라고 하자.
- 그리고 T가 light edge (u, v)를 포함하지 않는다고 가정하자.
- $A ∪ \lbrace (u, v) \rbrace$를 포함하는 다른 MST $T'$를 cut-and-paste technique를 이용하여 만들 수 있고, (u, v)가 A에 대해 safe하다는 것을 보일 수 있다.

![2023-05-15-mst-proof.png]({{site.baseurl}}/assets/images/2023-05-15-mst-proof.png){: .align-center}  

- edge $(u, v)$는 simple path $p$에 속한 edge들과 cycle을 형성한다.
- $u$와 $v$는 cut $(S, V-S)$의 서로 다른 side에 있으므로, simple path $p$에는 cut을 cross하는 edge가 적어도 하나 존재한다. 그 edge를 $(x, y)$라고 하자.
- edge $(x, y)$는 $A$에 포함되어 있지 않다. 왜냐하면 cut은 $A$를 respect하기 때문이다.
- $(x, y)$가 $T$에서 $u$로부터 $v$로 가는 unique path에 있기 때문에 $(x, y)$를 제거하면 $T$가 둘로 나눠진다.
- $(u, v)$를 추가하면 다시 결합되며, 새로운 spanning tree $T'=T - \lbrace (x, y) \rbrace ∪ \lbrace (u, v) \rbrace$를 얻을 수 있다.
- $T'=T - \lbrace (x, y) \rbrace ∪ \lbrace (u, v) \rbrace$가 MST임을 보이자.
- $(u, v)$가 cut $(S, V-S)$를 cross하는 light edge 이고, $(x, y)$도 이 cut을 cross한다. $w(u, v) \le w(x, y)$이므로 $w(T') = w(T) - w(x, y) + w(u, v) ≤ w(T)$이다.
- 그러므로 $T'$는 minimum spanning tree이다.
- MST $T'$에 대해 $A \subseteq T'$이고, $A ∪ \lbrace (u, v) \rbrace \subseteq T'$이므로, $(u, v)$는 $A$에 대해 safe하다.

## Understanding of `GENERIC-MST`

- 진행되면서 set A는 항상 acyclic이다. 아니면 A를 포함학고 있는 MST도 cycle이 있게 되며 이는 모순이다.
- 어떤 실행 시점에서 graph $G_A = (V, A)$는 forest이며, $G_A$의 연결된 component들은 tree이다.
- 몇몇 tree들은 하나의 vertex만 갖고 있을 수도 있다. 예를 들어 초기 시작에는 A가 empty이며, 한 개의 vertex를 가진 tree V개로 구성된 forest이다.
- 그리고 어떤 $A$의 safe edge $(u, v)$는 $G_A$의 distinct component를 연결한다. 왜냐하면 $A ∪ \lbrace (u, v) \rbrace$는 acylic하기 때문이다.
- `GENERIC-MST`의 line 2-4 **`while`** 루프는 $\vert V \vert - 1$번 수행된다. 왜냐하면 각 이터레이션에서 MST의 $\vert V \vert - 1$개 edge 중 하나를 찾아내기 때문이다.
- 초기에 $A = \emptyset$일 때 $G_A$에는 $\vert V \vert$개의 트리가 있다. 각 이터레이션에서 트리의 수를 1씩 감소시킨다.
- forest가 하나의 트리만을 갖게 되면 메소드는 종료된다.

## Corollary 23.2

- G=(V,E)가 E에 대해 weight function w가 정의된 connected undirected graph라고 하자.
- A가 G의 어떤 MST에 속하는 edge들의 집합이라고 하자.
- $C=(V_C, E_C)$가 forest $G_A=(V, A)$의 어떤 connected component (tree)라고 하자.
- cut $(V_C, V - V_C)$
- $(u, v)$가 $C$와 $G_A$의 다른 컴포넌트를 연결하는 light edge라고 하자. 그러면 $(u, v)$는 $A$에 대해 safe하다.

## Corollary 23.2 (Proof)

- cut $(V_C, V - V_C)$가 $A$를 respect하고 $(u, v)$는 이 cut의 light edge이다.
- Theorem 23.1에 의해 $(u, v)$는 $A$에 대해 safe하다.

## Kruskal's and Prim's Algorithm

- 두 알고리즘 모두 `GENERIC-MST`의 line 3에서 safe edge를 정할 때 규칙을 사용한다.
- Kruskal's algorithm:
  - 집합 $A$는 forest이다.
  - $A$에 추가되는 safe edge는 항상 두 개의 component를 연결하는 가장 least-weight edge 선택한다.
- Prim's algorithm:
  - 집합 $A$는 single tree이다.
  - $A$에 추가되는 safe edge는 항상 tree에 속하지 않은 vertex를 tree와 연결하는 가장 least-weight edge 선택한다.

## Kruskal's Algorithm

- 각 step에서 least possible weight를 가진 edge를 forest에 추가하므로 greedy algorithm이다.
- forest에서 두 개의 tree를 연결하는 모든 edges들 중 가장 작은 weight를 가진 edge를 safe edge로 찾아 추가한다.
- 두 개의 tree $C_1$과 $C_2$가 $(u, v)$로 연결된다고 하자.
- $(u, v)$는 $C_1$과 다른 tree를 연결하는 light edge이다. Corollary 23.2로부터 $(u, v)$는 $C_1$의 safe edge이다.

## Implementation of Kruskal's Algorithm

- 여러 disjoint set을 유지하기 위해 disjoint-set data structure를 이용한다.
- 각 set는 현재 forest에서 하나의 tree에 속한 vertex들을 포함한다.
- operation $FIND-SET(u)$는 $u$를 포함하고 있는 set의 대표 원소(representative element)를 리턴한다.
- 두 개의 vertex $u$와 $v$가 같은 tree에 속하는지 확인하기 위해 `FIND-SET(u)`와 `FIND-SET(v)`가 같은지를 확인하면 된다.
- 트리들을 결합하기 위해 `UNION`을 호출한다.

```text
MST-KRUSKAL(G,w)
1. A = ∅
2. for each vertex v ∈ G.V
3.   MAKE-SET(v)
4. sort the edges of G.E into nondecreasing order by weight w
5. for each edge (u, v) ∈ G.E, taken in sorted order
6.   if FIND-SET(u) ≠ FIND-SET(v)
7.     A = A ∪ {(u, v)}
8.     UNION(u, v)
9. return A
```

## Analysis of Kruskal's Algorithm

- line 3: $O(V)$ `MAKE-SET()` calls
- line 4: $O(E \lg E)$
- line 6: $O(E)$ `FIND-SET()` calls
- line 8: $O(V)$ `UNION()` calls

- union-by-rank와 path-compression heuristics를 적용한 disjoint-set-forest 사용
- disjoint-set operation은 $O((\vert V \vert + \vert E \vert) \alpha ( \vert V \vert))$ 시간이 걸린다. 여기서 $\alpha$는 매우 느리가 증가하는 function이다.
  - line 2-3 for loop: $\vert V \vert$ `MAKE-SET()` calls
  - line 5-8 for loop: $\vert E \vert$ `FIND-SET()` and `UNION()` calls
- $G$가 connected이므로 $\vert E \vert \ge \vert V \vert - 1$이다. disjoint-set operation은 $O(\vert E \vert \alpha (\vert V \vert))$ 시간이 걸린다.
- $\alpha (\vert V \vert) = O(\lg \vert V \vert) = O(\lg \vert E \vert)$이므로 Kruskal's algorithm은 $O(E \lg E)$ 시간이 걸린다.
- $\vert E \vert < \vert V \vert^2 \implies \lg \vert E \vert = O(\lg V)$이고, 따라서 Kruskal's algorithm은 $O(E \lg V)$ 시간이 걸린다.
- 정리하자면,
  - sort edges: $O(E \lg E)$
  - disjoint-set operation
    - $O(\vert V \vert + \vert E \vert)$ operations $\implies $O((\vert V \vert + \vert E \vert) \alpha(\vert V \vert))$ time
    - $\vert E \vert \ge \vert V \vert - 1 \implies O(\vert E \vert \lg V)$ time
    - $\alpha(n)$은 tree의 height에 upper bounded되므로 $\alpha (\vert V \vert) = O(\lg \vert V \vert) = O(\lg \vert E \vert)$
- Kruskal's algorithm은 $O(E \lg E)$ 시간이 걸린다.
- $\vert E \vert \gt \vert V \vert^2 \implies \lg \vert E \vert = O(\lg \vert V \vert)$이므로 Kruskal's algorithm은 $O(\vert E \vert \lg \vert V \vert)$ 시간이 걸린다.

## Prim's Algorithm

- 각 step에서 가능한 minimum weight를 가진 edge를 tree에 추가하므로 greedy algorithm이다.
- set $A$에 있는 edge들은 항상 single tree를 이룬다.
- 각 step에서 tree $A$에 $A$와 isolated vertex를 연결하는 light edge를 추가한다.
- Corollary 23.3에 의해 이 규칙은 A에 대해 safe edge만을 추가한다.

## Implementation of Prim's Algorithm

- Input은 connected graph $G$와 minimum spanning tree의 root $r$이다.
- 알고리즘 실행 동안에 tree에 속하지 않은 모든 vertex들은 min-priority queue Q에 저장된다.
- 각 vertex $v$는 key $v.key$를 갖는다. $v.key$는 $v$와 tree에 속한 vertex들을 연결하는 edge들 중 가장 작은 weight를 갖는 edge의 weight이다.
- convention에 따라 $v.key = \infty$로 초기화한다.
- $v.\pi$는 $v$의 parent를 나타낸다.
- 이 알고리즘은 set $A = \lbrace (v, v.\pi): v \in V - \lbrace r \rbrace - Q \rbrace$를 유지한다.
- min-priority queue Q가 비면 종료된다.

```text
MST-PRIM(G, w, r)
1.  for each u ∈ G.V
2.    u.key = ∞
3.    u.π = NIL
4.  r.key = 0
5.  Q = G.V
6.  while Q ≠ ∅
7.    u = EXTRACT-MIN(Q)
8.    for each v ∈ G.Adj[u]
9.      if v ∈ Q and w(u, v) < v.key
10.       v.π = u
11.       v.key = w(u, v)
```

## Running Time of Prim's Algorithm

- Prim 알고리즘의 running time은 우리가 min-priority queue Q를 어떻게 구현하느냐에 따라 달라진다.
- Q를 binary min-heap으로 구현할 경우
  - `EXTRACT-MIN`이 $O(\lg V)$ 시간이 걸린다.
  - `DECREASE-KEY`가 $O(\lg V)$ 시간이 걸린다.
  - Total time: $O(E \lg V)$
- Q를 simple array로 구현할 경우
  - `EXTRACT-MIN`이 $O(V)$ 시간이 걸린다.
  - `DECREASE-KEY`가 $O(1)$ 시간이 걸린다.
  - Total time: $O(E + V^2)$
- Q를 Fibonacci heap으로 구현할 경우
  - `EXTRACT-MIN`이 $O(\lg V)$ amortized 시간이 걸린다.
  - `DECREASE-KEY`가 $O(1)$ amortized 시간이 걸린다.
  - Total time: $O(E + V \lg V)$
