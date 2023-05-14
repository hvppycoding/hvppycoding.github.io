---
title: "Single-source Shortest Paths"
excerpt: "Advanced Programming Methodologies 12"
date: 2023-05-15 04:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Single-source Shortest Paths Problem

- weighted directed graph $G=(V, E)$(with weight function $w: E \rightarrow R$)가 주어졌을 때 source vertex s에서부터 vertex v까지의 minimum-weight path를 찾는 문제이다.
  - weight $w(p)$ of path $p=<v_0, v_1, ..., v_k>$는 edge들의 weight sum이다: $w(p) = \sum_{i=1}^{k} w(v_{i-1}, v_i)$
  - shortest-path weight $\delta(s, v)$

$$
\delta(s, v) = \begin{cases}
    \min_p\lbrace w(p): s ⤳ v \rbrace & \text{if there is a path p from u to v} \\
    \infty & \text{otherwise}
\end{cases}
$$

## Optimal Substructure Property

- 두 vertex 사이의 shortest path는 다른 shortest path들을 포함하고 있음.
- Lemma 24.1 (Subpaths of shortest paths are shortest paths)
  - $G=(V, E)$가 weighted directed graph이고 $w$가 edge weight function이라고 하자.
  - $p=<v_0, v_1, ..., v_k>$가 $v_0$에서 $v_k$로 가는 shortest path라고 하자.
  - 그렇다면 $p$의 subpath $p'=<v_i, v_{i+1}, ..., v_j>$는 $v_i$와 $v_j$ 사이의 shortest path이다.

## Shortest Path Properties

- negative weight edge를 포함할 수도 있다.
- 만약 graph $G=(V, E)$에 source s에서 도달 가능한 negative weight cycle은 포함되지 않는다. 이 경우 $v \in V$에 대해 shortest-path weight $\delta(s, v)$는 well-defined된다.
- 만약 s에서 도달 가능한 negative weight cycle이 존재한다면, shortest path는 well-defined되지 않는다.

## Representing Shortest Paths

- breath-first search와 같은 방법으로 shortest path를 표현할 수 있다.
- shortest-paths tree:
  - Predecessor subgraph $G_{\pi} = (V_{\pi}, E_{\pi})$
    - $V_{\pi} = \lbrace v \in V: v.\pi \neq NIL \rbrace \cup \lbrace s \rbrace$
    - $E_{\pi} = \lbrace (v.\pi, v): v \in V_{\pi} - \lbrace s \rbrace \rbrace$

## Relaxation

- 모든 $v \in V$에 대해 $v.d$는 s로부터 v까지의 shortest-path의 upper bound 값을 갖도록 할 것이다. 이를 **shortest-path estimate**라고 부른다.

```text
INITIALIZE-SINGLE-SOURCE(G, s)
1.  for each vertex v ∈ V
2.    v.d = ∞
3.    v.π = NIL
4.  s.d = 0
```

- edge $(u, v)$를 **relaxing**하는 과정은 v로 가는 shortest path를 u를 통해 개선할 수 있는지 확인하는 과정과 $v.d$와 $v.\pi$를 업데이트하는 과정으로 구성된다.

```text
RELAX(u, v, w)
1.  if v.d > u.d + w(u, v)
2.    v.d = u.d + w(u, v)
3.    v.π = u
```

- 본 챕터에 소개되는 알고리즘들은 `INITIALIZE-SINGLE-SOURCE`를 한 후 반복적으로 edge를 relax한다.

## Properties of Shortest Paths and Relaxation

- Triangle inequality (Lemma 24.10)
- Upper-bound property (Lemma 24.11)
- No-path property (Corollary 24.12)
- Convergence property (Lemma 24.14)
- Path-relaxation property (Lemma 24.15)
- Predecessor-subgraph property (Lemma 24.17)

## Triangle Inequality (Lemma 24.10)

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s). $\delta(s, v) \le \delta(s, u) + w(u, v)$이다.
- Proof:
  - source s에서 v로 가는 shortest path p가 있다고 가정하자. 그렇다면 p는 s에서 v로 가는 어떤 다른 path들보다 크지 않은 weight를 갖는다. 특히 p는 s에서 u로 가는 shortest path에 edge (u, v)를 추가한 것보다 크지 않다.

## Upper-bound Property (Lemma 24.11)

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s).
- graph G는 `INITIALIZE-SINGLE-SOURCE`를 통해 initialize되었다.
- $v.d \ge \delta(s, v)$ for all $v \in V$이다. 이 invariant는 relaxation이 어떤 순서로 일어나든지 유지된다. 또한 한 번 $v.d$가 lower bound인 $\delta(s, v)$가 되면 그 값은 변하지 않는다.
- Proof:
  - relaxtion 횟수에 대해 induction
    - Base case:
      - $v.d = \infty \ge \delta(s, v)$ for all $v \in V$
      - $s.d = 0 = \delta(s, s)$
    - Inductive step:
      - Induction hypothesis: edge (u, v)를 relaxation 하기 전에 $v.d \ge \delta(s, v)$ for all $v \in V$를 만족한다.
      - 변경될 수 있는 d값은 $v.d$ 밖에 없다. 만약 변한다면 $v.d = u.d + w(u, v) \ge \delta(s, u) + w(u, v) \ge \delta(s, v)$이다.
    - $v.d$가 lower bound인 $\delta(s, v)$가 되면 그 값은 변하지 않는 것을 확인하자.
      - $v.d \ge \delta(s, v)$이므로 $v.d$는 더 감소할 수 없다.
      - relexation step에서 d값을 증가시키지 않으므로 $v.d$는 더 증가할 수 없다.

## No-path Property (Corollary 24.12)

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s). source s에서 v로 가는 path가 존재하지 않는다고 가정하자.
- `INITIALIZE-SINGLE-SOURCE`를 통해 initialize된 후, $v.d = \delta(s, v) = \infty$이다. 이 식은 relaxation이 어떤 순서로 일어나든지 유지된다.
- Proof:
  - By the upper-bound property, $\infty = \delta(s, v) \le v.d$ for all $v \in V$이며, $v.d = \infty = \delta(s, v)$이다.

## Lemma 24.13

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자.
- `RELAX(u, v, w)`를 실행하여 edge (u, v)를 relax한 직후 $v.d \le u.d + w(u, v)$이다.
- Proof:
  - $v.d > u.d + w(u, v)$였을 경우
    - $v.d = u.d + w(u, v)$로 업데이트된다.
  - $v.d \le u.d + w(u, v)$였을 경우
    - $v.d$는 변하지 않는다.

## Convergence Property (Lemma 24.14)

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s).
- 어떤 vertex u, v에 대해 s⤳u→v가 shortest path라고 하자.
- graph가 `INITIALIZE-SINGLE-SOURCE(G,s)`를 통해 initialize되었고, `RELAX(u, v, w)`가 포함된 relaxation sequence가 수행되었다고 하자.
- 수행 전에 $u.d = \delta(s, u)$이었다면 수행 후에 $v.d = \delta(s, v)$이 된다.
- Proof:
  - upper-bound property에 의해 $u.d = \delta(s, u)$는 유지된다.
  - 특히, edge (u, v)를 relaxing한 후 $v.d \le u.d + w(u, v) = \delta(s, u) + w(u, v) = \delta(s, v)$이다.
  - upper-bound property에 의해 $v.d \ge \delta(s, v)$이므로 $v.d = \delta(s, v)$이다.
  - 이 equality는 이후 계속 유지된다.

## Path-relaxation Property (Lemma 24.15)

- 만약 $p = <v_0, v_1, ..., v_k>$가 $s=v_0$에서 $v_k$까지의 shortest path이고 p의 edge들이 $(v_0, v_1), (v_1, v_2), ..., (v_{k-1}, v_k)$의 순서로 relax되었다면, $v_k.d = \delta(s, v_k)$이다.
- 이 특성은 다른 relaxation이 중간에 일어나도 유지된다.
- Proof:
  - path $p=<v_0, v_1, ..., v_k>$의 i-번째 edge가 relax된 후 induction을 통해 보이자. $v_i.d = \delta(s, v_i)$
  - bases: $i=0$일 때, $v_0.d = s.d = 0 = \delta(s, s)$이다. **upper-bound property**에 의해 $s.d$는 이후로 변하지 않는다.
  - induction step: $v_{i-1}.d=\delta(s, v_{i-1})$이라고 가정한다. edge $(v_{i-1}, v_i)$를 relax하면 convergence property에 의해 relaxation 후에 $v_i.d = \delta(s, v_i)$이다. 이 equality는 이후 계속 유지된다.

## Predecessor-subgraph Property (Lemma 24.17)

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s). 그리고 $G$에 s에서 reachable한 negative weight cycle이 없다고 하자.
- `INITIALIZE-SINGLE-SOURCE(G, s)`를 통해 initialize된 후, 모든 $v \in V$ vertex에 대해 $v.d = \delta(s, v)$로 만드는 relaxtion sequence가 수행되었다고 하자.
- predecessor subgraph G는 s가 root인 shortest-path tree이다.
- Proof is omitted.

## Bellman-Ford Algorithm

- Edge가 negative weight를 가질 수 있다.

```text
1.  INITIALIZE-SINGLE-SOURCE(G, s)
2.  for i = 1 to |G.V| - 1
3.    for each edge (u, v) ∈ G.E
4.      RELAX(u, v, w)
5.  for each edge (u, v) ∈ G.E
6.    if v.d > u.d + w(u, v)
7.      return FALSE
8.  return TRUE
```

- line 4: 모든 Edge에 대해 $\vert V \vert - 1$번 relaxation을 수행한다.
- line 6: negative weight cycle이 존재하는지 확인한다.
- Time Complexity: $O(\vert V \vert \vert E \vert)$

## Lemma 24.2

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s). 그리고 $G$에 s에서 reachable한 negative weight cycle이 없다고 하자.
- `BELLMAN-FORD`의 2-4 line for loop를 $\vert V \vert - 1$회 수행한 후, 모든 s로부터 reachable한 모든 vertex에 대해 $v.d = \delta(s, v)$가 된다.
- Proof:
  - Path-relaxation property를 사용하여 증명한다.
  - 어떤 vertex v가 path $p=<v_0, v_1, ..., v_k>$를 통해 s로부터 reachable하다고 하자($v_0=s$, $v_k=v$).
  - shortest paths는 simple path이기 때문에 p는 최대 $\vert V \vert - 1$개의 edge를 가질 수 있다. 따라서 $k \le \vert V \vert - 1$이다. $\vert V \vert - 1$번의 각 이터레이션은 모든 edge $\vert E \vert$에 대해 relax를 수행한다.
  - edge $(v_{i-1}, v_i)$는 i-번째 이터레이션에서 relax되는 edge들 중 하나이다.
  - path-relaxation property에 의해 $v.d=v_k.d=\delta(s, v_k)=\delta(s, v)$이다.

## Corollary 24.3

- $G=(V, E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s).
- 각 $v \in V$에 대해 s에서 v로 가는 path가 존재한다. ⇔(iff) `BELLMAN-FORD`가 $v.d < \infty$로 terminate한다.
- Proof: Exercise 24.1-2

## Correcntess of Bellman-Ford Algorithm

- $G=(V,E)$가 weighted directed graph이고 edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자(source vertex: s). 이에 대해 `BELLMAN-FORD`를 수행한다.
- $G$에 s에서 reachable한 negative weight cycle이 없으면 알고리즘이 `TRUE`를 반환하고 모든 $v \in V$에 대해 $v.d = \delta(s, v)$를 얻는다. 또한 prodecessor subgraph G는 s가 root인 shortest-path tree이다.
- $G$가 s에서 reachable한 negative weight cycle을 가지면 알고리즘이 `FALSE`를 반환한다.
- Proof(TRUE case):
  - s에서 reachable한 negative weight cycle이 없다고 가정하자.
  - 먼저 termination 시 $v \in V$에 대해 $v.d = \delta(s, v)$를 얻는다는 것을 보이자.
    - vertex v가 reachable하면, Lemma 24.2에 의해 증명된다.
    - vertex v가 reachable하지 않으면, no-path property에 의해 증명된다.
  - predecessor-subgraph property로부터 $G.\pi$는 shortest-path tree이다.
  - 이제 `BELLMAN-FORD`가 `TRUE`를 리턴함을 보이자.
  - termination에서 모든 edge $(u, v) \in E$에 대해
    - $v.d = \delta(s, v)$
    - $\qquad \le \delta(s, u) + w(u, v)$ (by triangle inequality)
    - $\qquad = u.d + w(u, v)$
  - 따라서 line 6에서 `FALSE`를 리턴하는 경우는 존재하지 않는다.
  - `BELLMAN-FORD`가 `TRUE`를 리턴함을 보였다.
- Proof(FALSE case):
  - $G$가 s에서 reachable한 negative weight cycle을 가지고 있다고 가정하자.
  - negative cycle을 $c=<v_0, v_1, ..., v_k>$라고 하자
    - $v_0=v_k$
    - $\sum_{i=1}^k w(v_{i-1}, v_i) < 0$
  - `BELLMAN-FORD`가 `TRUE`를 리턴한다고 가정하자.
    - $v_i.d \le v_{i-1}.d + w(v_{i-1}, v_i)$ for $i=1, 2, ..., k$
  - 위 부등식을 cycle c에 대해 더하자
    - $\sum_{i=1}^k v_i.d \le \sum_{i=1}^k ( v_{i-1}.d + w(v_{i-1}, v_i) )$
    - $\qquad = \sum_{i=1}^{k} v_{i-1}.d + \sum_{i=1}^{k} w(v_{i-1}, v_i)$
  - $v_0=v_k$이기 때문에 c의 각 vertex는 summation들에 한 개씩 있다. $\sum_{i=1}^k v_i.d = \sum_{i=1}^k v_{i-1}.d$이다.
  - Corollary 24.3에 의해 $v_i.d$는 finite하다. (for $i=1, 2, ..., k$)
  - $0 \le \sum_{i=1}^k w(v_{i-1}, v_i)$이므로 negative cycle의 합과 모순된다.

## Single-source Shortest Paths in Directed Acyclic Graphs

- Problem: DAG 그래프에서 shortest path 찾는 문제
  - Bellman-Ford는 $O(\vert V \vert \vert E \vert)$ 시간이 걸린다.
  - Idea: topological sort를 사용하자.

```text
DAG-SHORTEST-PATHS(G, w, s)
1.  topologically sort the vertices of G
2.  INITIALIZE-SINGLE-SOURCE(G, s)
3.  for each vertex u, taken in topologically sorted order
4.    for each vertex v ∈ G.adj[u]
5.      RELAX(u, v, w)
```

- Time Complexity: $O(\vert V \vert + \vert E \vert)$

## Theorem 24.5

- cycle이 없는 weighted directed graph $G=(V, E)$가 주어졌고, edge weight function $w: E \rightarrow R$이 정의되어 있다고 하자. 그리고 source vertex s가 주어졌다고 하자. 그러면 `DAG-SHORTEST-PATHS`는 s로부터 모든 vertex $v \in V$까지의 shortest path를 찾는다. 그리고 predecessor subgraph $G_\pi$는 shortes-paths tree이다.
- Proof:
  - 먼저 종료 시 모든 $v \in V$에 대해 $v.d = \delta(s, v)$임을 보이자.
  - 만약 v가 s로부터 reachable하지 않으면 no-path property에 의해 $v.d = \delta(s, v) = \infty$이다.
  - v가 s로부터 reachable하고, shortest path $p=<v_0, v_1, ..., v_k>$가 있다고 하자. ($v_0=s$, $v_k=v$).
  - 우리가 vertex들을 topologically sorted order로 처리하기 때문에 p의 edge들을 $(v_0, v_1), (v_1, v_2), ..., (v_{k-1}, v_k)$의 순서로 relax한다.
  - path-relaxation property에 의해 $v.d = \delta(s, v)$이다.
  - 마지막으로 predecessor-subgraph property로부터 $G_\pi$는 shortest-paths tree이다.