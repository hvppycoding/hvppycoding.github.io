---
title: "Depth First Search"
excerpt: "Advanced Programming Methodologies 08"
date: 2023-05-06 17:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Depth-First Search Pseudocode

![2023-05-07-depth-first-search.png]({{site.baseurl}}/assets/images/2023-05-07-depth-first-search.png)

- vertex u에 대해
  - u.d: discovered time
  - u.f: finished time
  - u.color
    - WHITE: u.d 이전
    - GRAY: u.d ~ u.f
    - BLACK: u.f 이후
  - Timestamp는 1부터 2|V|까지의 정수이다. 각 |V| vertex들에 대해 하나의 discovery event와 하나의 finishing event가 존재한다.

```text
DFS(G)
  for each vertex u ∈ G.V
    u.color = WHITE
    u.π = NIL
  time = 0
  for each vertex u ∈ G.V
    if u.color == WHITE
      DFS-VISIT(G, u)

DFS-VISIT(G, u)
  time = time + 1
  u.d = time
  u.color = GRAY
  for each v ∈ G.adj[u]
    if v.color == WHITE
      v.π = u
      DFS-VISIT(G, v)
  u.color = BLACK
  time = time + 1
  u.f = time
```

- DFS Time Complexity: Θ(|V|)
- DFS-VISIT Time Complexity: $\sum_{v \in V} | Adj[v]| =  Θ(|E|)$
- DFS Running Time: Θ(|V| + |E|)

## Depth-First Search Properties

- $G_\pi$가 forest of tree를 이룬다.
- u=v.π ⇔(iff) `DFS-VISIT(v)`가 u의 adjacency list를 탐색 중 불렸을 때
- Discoverty time과 finishing time은 parenthesis structure 갖고 있음

### Theorem 22.7 (Parenthesis Theorem)

- (directed 혹은 undirected) graph G=(V, E)에서 depth-first search 시 두 임의의 vertex u와 v에 대해 다음 세가지 중 딱 한 가지 조건만 만족한다.
  - [u.d, u.f]와 [v.d, v.f]가 완전히 disjoint한 경우
  - [u.d, u.f]가 [v.d, v.f]에 완전히 포함되는 경우
  - [v.d, v.f]가 [u.d, u.f]에 완전히 포함되는 경우

### Proof of Theorem 22.7 (Parenthesis Theorem)

- u.d < v.d 인 경우부터 살펴보자
  - v.d < u.f:
    - v가 u가 GRAY일 때 발견되었음. 이는 v가 u의 descendant임을 의미한다.
    - 그리고 v가 u보다 최근에 발견되었으므로 v의 모든 outgoing edge가 탐색되고, v가 종료된다.(u로 return하기 전에)
    - 그러므로 [v.d, v.f]는 interval [u.d, u.f]에 완전히 포함된다.
  - u.f < v.d:
    - u.d < u.f 이고 v.d < v.f
    - [u.d, u.f]와 [v.d, v.f]는 disjoint하다.
- v.d < u.d인 경우와 비슷하게 증명할 수 있다.

### Corollary 22.8

- Depth-first forest에서 Vertex v가 u의 proper descendant ⇔(iff) u.d < v.d < v.f < u.f
- Theorem 22.7로부터 바로 증명 가능하다.

### Theorem 22.9 (White-path Theorem)

- (directed 혹은 undirected) Graph G=(V,E)의 depth-first forest에서 vertex v가 vertex u의 descendant ⇔(iff) u.d 발견 시점에서 u에서 White vertex로 구성된 path를 통해 v로 도달할 수 있다.

### Proof of Theorem 22.9(⇒)

- v=u이면 u에서 v로의 path는 vertex u만 포함하고 있다. 이는 u.d 시점에서 u는 White이다.
- 이제 v가 u의 proper descendant인 경우를 가정하자.
  - Corollary 22.8로부터 u.d < v.d이고, v는 u.d 시점에서 White이다.
  - v는 u의 어떤 descendant라도 될 수 있으므로, u.d 시점에서 u에서 v로 가는 unique simple path의 모든 vertex는 White이다.

### Proof of Theorem 22.9(⇐)

- u.d 시점에서 u에서 v로 가는 white vectex들로 구성된 path가 존재하지만, v가 u의 descendant가 아니라고 가정하자.
- 일반성을 잃지 않고, v 외의 path 위의 모든 vertex가 u의 descendant가 된다고 가정하자. (아니면, v가 u의 descendant가 아닌 path에서 가장 가까운 vertex라고 가정하자)
- w가 v의 path의 predecessor라고 하자. w는 u의 descendant이다.(w와 u는 동일한 vertex일 수도 있다)
- Corollary 22.8에 의해 w.f ≤ u.f이다.
- v는 u가 discover된 후에 discover되어야하지만 w가 finish되기 전에는 discover되어야 한다. 즉, u.d < v.d < w.f ≤ u.f
- Theorem 22.7로부터 [v.d, v.f]는 [u.d, u.f]에 완전히 포함되어야 한다.
- Corollary 22.8로부터 v는 u의 descendant가 된다.

## Classification of Edges

- Depth-first forest $G_\pi$에서 Edge (u, v)의 타입들
  - Tree edge: Vertex v가 Edge (u, v)를 탐색할 때 처음 발견됨
  - Back edge: Vertex u가 Ancestor v를 연결
  - Forward edge: Vertex u가 Descendant v를 연결
  - Cross edge: 그 외 모든 경우

### Theorem 22.10

- undirected graph G에서 depth-first search에서 모든 edge는 tree edge나 back edge이다.
- Proof
  - (u,v)가 G의 임의의 Edge라고 하자. 일반성을 잃지 않고 u.d < v.d라고 가정할 수 있다.
  - v는 u가 finish 되기 전에 discover & finish 되어야 한다(u가 gray인 동안). 왜냐하면 v가 u의 adjacency list에 포함되어 있기 때문에.
  - Case 1. edge (u,v)가 u에서 v방향으로로 처음 explore 되었을 때, v는 그 전까지는 undiscover(white) 상태였다. 아니라면 v에서 u 방향으로 먼저 이 edge를 explore했을 것이다. 그러므로 (u, v)는 tree edge가 된다.
  - Case 2. edge (u,v)가 v에서 u 방향으로 먼저 explore가 되었다면, (u,v)는 back edge이다. 왜냐하면 u에서 v로 향하는 tree edge path가 존재할 것이기 때문이다.

## Topological Sort

![2023-05-07-topological-sort.png]({{site.baseurl}}/assets/images/2023-05-07-topological-sort.png)

- Directed Edge (u,v)는 u가 v보다 먼저 나와야 한다는 의미이다.

```text
TOPOLOGICAL-SORT(G)
  call DFS(G) to compute finishing times v.f for each vertex v
  as each vertex is finished, insert it onto the front of a linked list
  return the linked list of vertices
```

### Lemma 22.11

- directed graph G는 acyclic 하다 ⇔(iff) depth-first search에서 back edge가 존재하지 않는다.

### Proof of Lemma 22.11(⇒)

- u에서 v로의 back edge가 존재한다고 가정하자.
- v는 u의 ancestor이다.
- 그러므로 G에는 v에서 u로 가는 path p가 존재한다.
- back edge (u, v)가 있다면 p는 cycle이 된다.

### Proof of Lemma 22.11(⇐)

- (대우 증명)
- G가 cycle c를 포함하고 있다고 가정하자.
- G에 대한 depth-first search가 back edge가 있는 것을 보일 것이다.
- v를 cycle c에서 가장 처음 발견된 vertex라고 하자. (u,v)는 c의 preceding edge이다.
- v.d 시점에서 cycle c는 v에서 u로 가는 white path를 형성한다.
- white-path teorem에 의해 vertex u는 v의 depth-first forest에서의 descendant가 된다.
- 그러므로 (u,v)는 back edge가 된다.

### Theorem 22.12

- `TOPOLOGICAL-SORT(G)`가 DAG(direction acyclic graph) G의 topologycal sort를 반환한다.

- 증명
  - 주어진 DAG G=(V,E)에서 각각의 vertex에 대해 finishing time을 계산하기 위해 DFS를 수행한다고 하자.
  - 모든 u, v ∈ V 페어에 대해 G가 u에서 v로 가는 edge를 포함하면 v.f < u.f임을 보이는 것으로 충분하다.
  - (u,v) Edge가 탐색되었을 때, v는 gray가 될 수 없다.(그렇다면 cycle이 있는 것이기 때문에)
  - v가 white이면 v.f < u.f이다. 왜냐하면 v가 u의 descendant이기 때문이다.
  - v가 black이면 v.f < u.f이다. 왜냐하면 v가 u보다 먼저 finish되었기 때문이다.
  - 그러므로 DAG의 모든 (u,v) edge에 대해 v.f < u.f 임을 알 수 있다.

