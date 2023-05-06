---
title: "Elementary Graph Algorithms"
excerpt: "Advanced Programming Methodologies 07"
date: 2023-05-06 17:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Graph

- Graph는 G = (V, E)로 표현한다. V는 vertex의 집합이고, E는 edge의 집합이다.
- E에 (v, w)가 있다면 Vertex w는 v에 adjacent하다고 한다.
- Path는 Edge로 연결된 Vertex의 Sequence이다.
- length: Path에 포함된 Edge의 수
- simple path: Path에 포함된 Vertex가 모두 다른 Path
- cycle: Path의 시작 Vertex와 끝 Vertex가 같은 Path(단, length > 0)
- acyclic graph: cycle이 없는 directed graph
- 하나의 vertex에서 다른 모든 vertex로 가는 path가 존재하는 undirected graph를 connected graph라고 한다.
- directed graph가 connected graph이면 strongly connected라고 한다.
- 모든 vertex pair에 대해 edge가 존재하면 complete graph라고 한다.

## Representations of Graphs

- Adjacency list: 각 vertex마다 adjacent vertex를 list로 표현
- Adjacency matrix: V x V matrix

## Breadth-first Search

- s에서 reachable한 vertex까지의 거리를 계산한다.
- breath-first tree가 만들어진다.
- 각 Vertex를 white, gray, black으로 구분하여 진행한다.
- white: 첫 시작 시 모든 vertex는 white이다.
- gray: 해당 vertex가 발견되었을 때
- black: 모든 adjacent vertex가 발견되었을 때
- breath-first tree의 root는 breadth-first search의 시작 vertex인 s 노드이다. 그 후 이미 발견된 vertex u의 adjacent인 white vertex v를 발견할 때, edge (u, v)가 tree edge에 추가된다. 이 때 u는 v의 predecessor 혹은 parent가 된다.

```text
BFS(G, s)
  for each vertex u ∈ G.V - {s}
    u.color = WHITE
    u.d = ∞
    u.π = NIL
  s.color = GRAY
  s.d = 0
  s.π = NIL
  Q = ∅
  ENQUEUE(Q, s)
  while Q ≠ ∅
    u = DEQUEUE(Q)
    for each v ∈ G.Adj[u]
      if v.color == WHITE
        v.color = GRAY
        v.d = u.d + 1
        v.π = u
        ENQUEUE(Q, v)
    u.color = BLACK
```

## Loop Invariant

- Loop Invariant: while loop에서 Q가 gray vertex로 구성되어 있다. (BFS의 Correctness를 보여주는 것은 아니며 Loop Invariant 연습용)
- Initialization: 첫 Iteration 시 Q는 s만을 포함하고 있으므로 gray vertex로 구성되어 있다.
- Maintenance
  - while문 첫줄에서 gray vertex u를 dequeue한다.
  - for loop에서 u의 adjacent vertex들을 처리하는데, GRAY로 만든 vertex들만 Enqueue한다.
  - 그리고 마지막 줄에서 dequeue된 u는 BLACK으로 변경한다.

## Analysis of BFS

- 각 vertex는 최대 한 번만 enqueue된다.
- enqueue와 dequeue는 $O(1)$이므로, 큐에 관련된 시간 복잡도는 $O(|V|)$이다.
- 각 vertex에 대해 adjacency list를 한 번씩 scan한다. 따라서 시간 복잡도는 $O(|E|)$이다.
- 총 시간 복잡도는 $O(|V| + |E|)$이다.

## Shortest path

- Shortest-path distance $\delta(s, v)$는 s에서 v까지의 최단 경로의 edge 수라고 정의하자.
- 만약 path가 존재하지 않는다면 $\delta(s, v) = \infty$이다.

### Lemma 22.1

- directed 혹은 undirected graph $G = (V, E)$와 임의의 $s \in V$에 대해, $(u, v) \in E$이면 $\delta(s, v) \leq \delta(s, u) + 1$이다.

- 증명
  - 만약 u가 s로부터 reachable하면 v도 reachable하다.
  - s에서 v로 가는 shortest path는 s에서 u로 가는 shortest path에 (u,v)를 추가한 것보다 길 수는 없다.
  - u가 s로부터 unreachable하면 $\delta(s, u) = \infty$이다.

### Lemma 22.2

- directed 혹은 undirected graph $G = (V, E)$의 임의의 $s \in V$에서부터 BFS를 수행한다고 가정하자. 그러면 BFS가 종료될 때, 모든 vertex $v \in V$에 대해 $v.d \geq \delta(s, v)$을 만족한다.
- 증명
  - ENQUEUE 연산의 횟수에 대해 Induction을 사용한다.
  - Inductive hypothesis: $v.d \geq \delta(s, v)$ for all $v \in V$
  - Basis: $s.d = 0 = \delta(s, s)$이고, $v.d = \infty \geq \delta(s, v)$ for all $v \in V - \{s\}$
  - Inductive step: vertex u로부터 white vertex v가 찾아진다.
    - $v.d = u.d + 1$
    - $\qquad \geq \delta(s, u) + 1$ from the induction hypothesis
    - $\qquad \geq \delta(s, v)$ by Lemma 22.1($\delta(s, v) \leq \delta(s, u) + 1$)

### Lemma 22.3

- $G = (V, E)$에 대해 BFS를 수행한다고 가정하자. 큐 Q가 포함하고 있는 vertices $<v_1, v_2, ..., v_r>$에 대해($v_1$이 head이고 $v_r$이 tail), $v_r.d \leq v_1.d + 1$과 $v_i.d \leq v_{i+1}.d$ for $i = 1, 2, ..., r - 1$을 만족한다.
- 증명
  - Inductive hypothesis: $v_r.d \leq v_1.d + 1$이고 $v_i.d \leq v_{i+1}.d$ for $i = 1, 2, ..., r - 1$
  - Basis: 큐에는 s만 포함하고 있으므로 만족한다.
  - Inductive step: Enqueue와 Dequeue 모두에 대해 Lemma를 증명해야 한다.
  - Inductive step(Dequeue)
    - $v_1$이 Dequeue되고, $v_2$가 새로운 head가 된다.
    - hypothesis로부터 $v_1.d \leq v_2.d$이고, $v_r.d \leq v_1.d + 1 \leq v_2.d + 1$이다.
    - 여전히 조건을 만족한다.
  - Inductive step(Enqueue)
    - Enqueue되는 Vertex v가 $v_{r+1}$이 된다.
    - 이 때 u는 이미 큐에서 제거된 상태이다.
    - hypothesis에서 $v_1.d \geq u.d$이므로, $v_{r+1}.d = v.d = u.d + 1 \leq v_1.d + 1$이다.
    - hypothesis로부터 $v_r.d \leq u.d + 1$이고, $v_r.d \leq u.d + 1 = v.d = v_{r+1}.d$이다.
    - 여전히 조건이 만족한다.

### Corollary 22.4

- BFS 수행 중에 $v_i$와 $v_j$가 Enqueue되었고, $v_i$가 $v_j$보다 먼저 Enqueue되었다고 가정하자. 그러면 $v_j$가 enqueue될 때 $v_i.d \leq v_j.d$이다.
- 이 Corollary는 vertex들이 enqueue될 때 d 값이 monotonic하게 증가한다는 것을 보여준다.
- 증명: Lemma 22.3과 BFS 동안에 각 vertex가 최대 한 번만 enqueue되어 d 값이 결정된다는 것으로부터 바로 증명된다.

### Theorem 22.5 (BFS의 Correctness)

- $G=(V, E)$의 source vertex $s \in V$에서 BFS를 수행한다고 가정하자. 그러면 BFS는 source s로부터 도달가능한 모든 v를 발견한다. BFS가 종료될 때, 모든 vertex $v \in V$에 대해 $v.d = \delta(s, v)$이다. 또한, rechable한 모든 $v \neq s$인 vertex v에 대해 s에서 v로 가는 shortest path 중 하나는 s에서 v.π로가는 shortest path에 (v.π, v) edge를 더한 것과 같다.
- 증명
  - 어떤 vertex들이 shortest path distance가 아닌 d를 가진다고 가정하자. vertex v가 **틀린 d value**를 갖고있는 vertex들 중 **minimum δ(s,v)**를 가진 vertex라고 하자. Lemma 22.2로부터 **v.d > δ(s,v)**이다. vertex v는 반드시 s에서 reachable하다(아니면 δ(s,v) = ∞ ≥ v.d가 된다). u가 v의 shortest path의 predecessor라고 하자. δ(s,v) = δ(s,u) + 1 이다.
  - δ(s,u) < δ(s,v) 이고, 우리가 v를 고른 방식으로 인해 u.d = δ(s,u)를 만족한다. v.d > δ(s,v) = δ(s,u) + 1 = u.d + 1
  - 이제 Q에서 u를 deque하는 시점을 고려해보자.
    - v가 white: v.d = u.d + 1
    - v가 black: 이미 제거됨. Corollary 22.4로부터 v.d ≤ u.d
    - v가 gray: vertex w로 인해 v가 gray로 칠해졌다고 하자. w는 u보다 일찍 큐에서 제거되었고 v.d = w.d + 1이다. Corollary 22.4로부터 w.d ≤ u.d이므로 v.d ≤ u.d + 1이다.
  - 모든 Case에 대해서 v.d > u.d + 1 에 모순된다. 그러므로 모든 v ∈ V에 대해 v.d = δ(s,v) 이다.
  - s에서 도달할 수 있는 모든 v가 발견된다. 아니면 ∞ = v.d > δ(s,v)이 된다.
  - Theorem 증명을 마치기 위해, v.π = u이면 v.d = u.d + 1 이다.
  - 그러므로 s로부터 v로 가는 shortest path는 s에서 v.π로 가는 shortest path에 (v.π, v) edge를 더하여 얻을 수 있다.

## Breadth-First Trees(BFTs)

- 그래프 G=(V,E)와 source s에서 predecessor subgraph of G인 $G_\pi = (V_\pi, E_\pi)$ 를 정의할 수 있다.
  - $V_\pi = \lbrace v \in V\ \|\ v.\pi \neq NIL \rbrace ∪ \lbrace s \rbrace$
  - $E_\pi = \lbrace (v.\pi, v)\ \|\ v \in V_\pi - \lbrace s \rbrace \rbrace$
- Predecessor subgraph $G_\pi$는 BFT이고, $V_\pi$는 s로부터 reachable한 모든 vertex의 집합이다. $G_\pi$는 s로부터 reachable한 모든 vertex들의 unique shortest path를 포함한다.
- Breath-first tree는 connected된 tree이기 때문에 |E| = |V| - 1이다. (Theorem B.2)

### Theorem B.2

- G=(V,E)가 undirected graph일 때 다음은 모두 equivalent하다.
  - G는 tree이다.
  - G의 모든 vertex pair가 unique simple path로 연결되어 있다.
  - G가 연결되어 있고, 어떤 edge를 제거하였을 때 disconnected된다.
  - G가 connected이고 |E| = |V| - 1이다.
  - G가 acyclic이고 |E| = |V| - 1이다.
  - G가 acyclic이고 아무 edge하나를 추가하였을 때 cycle이 생긴다.

### Lemma 22.6

- directed 혹은 undirected 그래프인 G=(V, E)에 대해 BFS를 수행하고, π를 구성하였을 때 predecessor subgraph $G_\pi = (V_\pi, E_\pi)$는 BFT이다.
- 증명
  - BFS에서 v.π=u 로 설정할 때는 (u,v)∈E 이고 δ(s,v)<∞ 일 때이다. 따라서 $V_\pi$는 s에서 도달가능한 V의 vertex들로 구성되어 있다.
  - $G_\pi$는 tree를 구성하므로 이는 s에서 각 vertex로 가는 unique path를 포함하고 있다.
  - Theorem 22.5를 inductive하게 적용하면 모든 path들이 shortest path임을 결론내릴 수 있다.